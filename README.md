# Students Database CRUD API

Python backend for managing student records (Create, Read, Update, Delete) built with **FastAPI** and **SQLite**.
Includes an **optional** local-AI endpoint powered by **Ollama**.

## Prerequisites
- Python 3.10+
- Git
- (Optional bonus) [Ollama](https://ollama.com/download)

## Installation
```bash
git clone <your-repo-url>
cd students-crud

python -m venv venv
# Windows:  venv\Scripts\activate
# Mac/Linux: source venv/bin/activate

pip install -r requirements.txt
```

## Run
```bash
python -m app.seed                 # optional: load 6 dummy students
uvicorn app.main:app --reload
```
- API: http://127.0.0.1:8000
- Interactive docs (test every endpoint in the browser): http://127.0.0.1:8000/docs

The SQLite file `students.db` is created automatically and keeps data between restarts.

## Endpoints
| Method | Endpoint | Purpose |
|---|---|---|
| POST | /students | Create a student |
| GET | /students | List / filter (`name`, `course`, `born_after`, `born_before`, `skip`, `limit`) |
| GET | /students/{id} | View one student |
| PUT | /students/{id} | Full update (send all fields) |
| PATCH | /students/{id} | Partial update (send only changed fields) |
| DELETE | /students/{id} | Delete a student |
| POST | /ai/ask | (Optional) Ask a question using Ollama |

### Student fields
`id` (auto), `name`*, `date_of_birth`*, `email`, `phone` (*at least one of email/phone*), `course`, `address`, `enrollment_date`

### Sample requests (curl)
```bash
# Create
curl -X POST http://127.0.0.1:8000/students -H "Content-Type: application/json" \
  -d '{"name":"Riya Roy","date_of_birth":"2005-04-12","email":"riya@example.com","course":"Python Backend"}'

# List + filter
curl "http://127.0.0.1:8000/students"
curl "http://127.0.0.1:8000/students?course=python&born_after=2005-01-01"

# Read one
curl http://127.0.0.1:8000/students/1

# Partial update
curl -X PATCH http://127.0.0.1:8000/students/1 -H "Content-Type: application/json" -d '{"course":"Data Science"}'

# Delete
curl -X DELETE http://127.0.0.1:8000/students/1
```
(Windows PowerShell: easiest is to use the `/docs` page instead of curl.)

### Error handling
| Situation | Response |
|---|---|
| Invalid data (bad email, future DOB, short name, no contact) | 422 |
| Student id not found | 404 |
| Duplicate email | 409 |

## Run tests
```bash
python -m pytest -q
```

## Optional: Local AI with Ollama
```bash
ollama pull llama3.2:1b     # one-time download
ollama serve                # skip if Ollama already runs in the background
```
Then:
```bash
curl -X POST http://127.0.0.1:8000/ai/ask -H "Content-Type: application/json" \
  -d '{"question":"List students born after 2005"}'
```
Try also: "How many students are there?", "Who is enrolled in Data Science?", "Students born in 2005".

**How it stays safe & accurate:** the model never writes or runs SQL. Backend code detects filters
(birth year, course), fetches matching rows with parameterised queries, and passes only those rows
(plus exact counts) to the model as context. If Ollama is not running, `/ai/ask` returns a clear 503 message.
Config via env vars: `OLLAMA_URL`, `OLLAMA_MODEL`.

## Project structure
```
app/
  main.py      # FastAPI routes
  schemas.py   # Pydantic validation models
  crud.py      # SQLite queries (parameterised)
  database.py  # connection + table creation
  ai.py        # optional Ollama endpoint
  seed.py      # dummy data loader
tests/test_api.py
requirements.txt
```

## Notes
Only dummy data is used. No secrets or real student information are committed (`*.db` and `.env` are git-ignored).
