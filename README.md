# Conversational Event Management Python

This repository is organized as a simple monorepo:

- `backend-python/`
  FastAPI backend, Alembic setup, backend docs, Dockerfile, and provider integrations.
- `frontend/`
  React frontend application.

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

- Render can deploy the backend using the root [render.yaml](/e:/AI-Conversational/ai-conversational-python-backend-repo/render.yaml)
- Vercel can deploy the frontend from the `frontend/` directory

## More Docs

- [Backend Docs](/e:/AI-Conversational/ai-conversational-python-backend-repo/backend-python/docs/README.md)
- [Backend Deployment Guide](/e:/AI-Conversational/ai-conversational-python-backend-repo/backend-python/docs/DEPLOYMENT_GUIDE.md)
