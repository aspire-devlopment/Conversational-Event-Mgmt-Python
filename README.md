# Conversational Event Management Python

This repository is organized as a simple monorepo:

- `backend-python/`
  FastAPI backend, Alembic setup, backend docs, Dockerfile, and provider integrations.
- `frontend/`
  React frontend application.

## Stack Snapshot

- Conversational AI layer: provider-configurable through `LLM_PROVIDER`
- Conversational model in the included Render deployment: Groq `llama-3.3-70b-versatile`
- Other supported model paths: OpenRouter `openrouter/auto` and Google Gemini `gemini-2.5-flash`
- Primary database: PostgreSQL
- Conversational session storage: PostgreSQL `chat_sessions` table with JSONB session state

## Local Development

Backend:

```bash
cd backend-python
pip install -r requirements.txt
copy .env.example .env
python run.py
```

Frontend:

```bash
cd frontend
npm install
copy .env.example .env.local
npm start
```

## Deployment

- Render can deploy the backend using the root [render.yaml](/e:/ai-conversational-python-backend-repo/render.yaml)
- Vercel can deploy the frontend from the `frontend/` directory

## More Docs

- [Backend Docs](/e:/ai-conversational-python-backend-repo/backend-python/docs/README.md)
- [Backend Deployment Guide](/e:/ai-conversational-python-backend-repo/backend-python/docs/DEPLOYMENT_GUIDE.md)
- [How To Run](/e:/ai-conversational-python-backend-repo/docs/HOW_TO_RUN.md)
- [Database ER Diagram](/e:/ai-conversational-python-backend-repo/docs/DATABASE_ER_DIAGRAM.md)
- [Project Overview](/e:/ai-conversational-python-backend-repo/docs/PROJECT_BUSINESS_OVERVIEW.md)
- [Detailed Architecture](/e:/ai-conversational-python-backend-repo/docs/ARCHITECTURE_DETAILED.md)
- [AI Tools And Models](/e:/ai-conversational-python-backend-repo/docs/AI_TOOLS_AND_MODELS.md)
