# How To Run

This guide covers both ways to run the project:

- with Docker using the root `docker-compose.yml`
- without Docker using local Python, Node.js, and PostgreSQL

## Run With Docker

### What This Runs

The root Docker Compose setup starts:

- `db`: PostgreSQL 16
- `backend`: FastAPI application on `http://localhost:8000`
- `frontend`: React application on `http://localhost:3000`

### Prerequisites

- Docker Desktop or Docker Engine
- Docker Compose support
- at least one LLM API key if you want the conversational chat flow to call a real model

### Start The Stack

From the repository root:

```bash
docker compose up --build
```

This exposes:

- frontend: `http://localhost:3000`
- backend API: `http://localhost:8000/api`
- backend health: `http://localhost:8000/health`
- PostgreSQL: `localhost:5432`

### Optional LLM Provider Keys

The Compose file reads these optional host environment variables if you set them before startup:

- `GROQ_API_KEY`
- `OPENROUTER_API_KEY`
- `GOOGLE_API_KEY`
- `LLM_API_KEY`

Example in PowerShell:

```powershell
$env:GROQ_API_KEY="your-key-here"
docker compose up --build
```

### Default Local Database Settings

The Compose setup creates this PostgreSQL database by default:

- database: `EVENT_MANAGEMENT_SYSTEM`
- user: `postgres`
- password: `pass123`
- host from your machine: `localhost`
- host from containers: `db`

### Stop The Stack

```bash
docker compose down
```

To also remove the PostgreSQL volume:

```bash
docker compose down -v
```

### Notes

- The backend expects the PostgreSQL schema used by this project to exist.
- This Compose setup provides the database server, but it does not generate every application table automatically.
- If you already have a seeded schema or migration flow for this project, apply that against the Compose PostgreSQL instance.
- The frontend container proxies `/api` requests to the backend container internally.

## Run Without Docker

### Prerequisites

- Python 3.12 or compatible local Python runtime
- Node.js 20 or compatible local Node.js runtime
- PostgreSQL running locally or on a reachable host
- at least one LLM API key if you want the conversational chat flow to call a real model

### 1. Prepare The Database

Create a PostgreSQL database and then apply the schema in [Database.sql](/e:/ai-conversational-python-backend-repo/backend-python/Database.sql).

Example:

```bash
psql -U postgres -f backend-python/Database.sql
```

### 2. Run The Backend

```bash
cd backend-python
pip install -r requirements.txt
copy .env.example .env
```

Update `.env` with your real values, especially:

- `DB_HOST`
- `DB_PORT`
- `DB_USER`
- `DB_PASSWORD`
- `DB_NAME`
- `JWT_SECRET`
- `LLM_PROVIDER`
- provider API key such as `GROQ_API_KEY`, `OPENROUTER_API_KEY`, or `GOOGLE_API_KEY`

Then start the backend:

```bash
python run.py
```

The backend will be available at:

- API: `http://localhost:8000/api`
- health: `http://localhost:8000/health`

### 3. Run The Frontend

In a new terminal:

```bash
cd frontend
npm install
copy .env.example .env.local
```

Set:

```env
REACT_APP_API_URL=http://localhost:8000/api
```

Then start the frontend:

```bash
npm start
```

The frontend will be available at:

- `http://localhost:3000`

## Quick Smoke Test

After either setup path:

1. Open `http://localhost:3000`.
2. Check backend health at `http://localhost:8000/health`.
3. Log in or register.
4. Open the admin event/chat flow.
5. If chat fails, confirm your LLM API key and provider settings are valid.
