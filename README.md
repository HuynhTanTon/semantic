# Semantic Search for Student Forms (HUTECH)

Semantic search (Vietnamese-friendly) over student forms: users type a natural-language query and get the most relevant forms by **meaning** instead of exact keyword matching.

## Highlights

- **Semantic retrieval** using text embeddings (Sentence-Transformers) + **PostgreSQL pgvector**
- **Microservice split**: React UI → Node.js/Express API proxy → FastAPI AI service → Postgres
- **Simple data ingestion** from JSON into the vector database
- **Dockerized database** with pgvector for reproducible local setup

## System architecture

```
┌───────────────┐        ┌──────────────────┐        ┌──────────────────────┐
│  React UI     │  HTTP  │ Node.js/Express  │  HTTP  │ FastAPI AI service   │
│ (Tailwind)    ├───────►│ /api/search      ├───────►│ embedding + top-K     │
└───────────────┘        └──────────────────┘        └──────────┬───────────┘
                                                                 │
                                                                 │ SQL + vector ops
                                                                 ▼
                                                     ┌──────────────────────┐
                                                     │ PostgreSQL + pgvector │
                                                     └──────────────────────┘
```

## Tech stack

- **Frontend**: React, Tailwind CSS, TypeScript
- **Backend**: Node.js, Express
- **AI service**: Python, FastAPI, Sentence-Transformers
- **Database**: PostgreSQL + pgvector
- **DevOps**: Docker, Docker Compose

## Quick start (local)

### Prerequisites

- Docker + Docker Compose
- Node.js (v14+)
- Python (v3.8+)

Run the services in separate terminals (DB / AI / backend / frontend).

### 1) Start Postgres (pgvector)

```bash
docker-compose up -d
```

### 2) Run the AI service (FastAPI)

```bash
cd ai_service
pip install -r requirements.txt
```

Create `ai_service/.env`:

```env
PORT=8000
DB_HOST=localhost
DB_NAME=semantic_search
DB_USER=postgres
DB_PASSWORD=postgres
DB_PORT=5432
```

Start:

```bash
python app.py
```

API docs:
- Swagger: `http://localhost:8000/docs`
- ReDoc: `http://localhost:8000/redoc`

### 3) Run the backend (Express proxy)

```bash
cd backend
npm install
```

Create `backend/.env`:

```env
PORT=4000
AI_SERVICE_URL=http://localhost:8000
```

Start:

```bash
npm start
```

### 4) Run the frontend (React + Vite)

```bash
cd ..
npm install
npm run dev
```

Open: `http://localhost:5173`

## API overview

### Backend (Express)

- `GET /` — health check
- `POST /api/search` — semantic search proxy

Default local ports:
- Frontend (Vite): `5173`
- Backend (Express): `4000` (configurable via `backend/.env`)
- AI service (FastAPI): `8000`
- Postgres (pgvector): `5432`

Example:

```json
POST /api/search
{
  "query": "đơn xin xác nhận sinh viên",
  "limit": 5
}
```

### AI service (FastAPI)

Expose search and ingestion capabilities (see `http://localhost:8000/docs` once running).

## Import data

Import a JSON file that contains an array of form objects:

```bash
cd ai_service
python import_data.py --file path/to/forms.json
```

Example JSON shape:

```json
[
  {
    "title": "Student Registration Form",
    "content": "Form for registering new students for the semester",
    "form_type": "Registration",
    "url": "/forms/registration"
  }
]
```

## Project structure

```
ai_service/        FastAPI AI service + ingestion script
backend/           Express proxy API
src/               React UI (Vite + Tailwind)
docker-compose.yml PostgreSQL + pgvector
```

## Notes for recruiters

- **What’s interesting here**: embedding-based retrieval + vector database (pgvector), clean service boundaries, and a practical ingestion/search flow suitable for Vietnamese queries.
- **Where to look**:
  - AI service: `ai_service/app.py`, `ai_service/import_data.py`
  - Backend proxy: `backend/index.js`
  - Frontend UI: `src/`

## License

MIT
