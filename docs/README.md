# Backend Docs

This folder contains backend-specific documentation for the FastAPI service.

## Available Docs

- `API_ENDPOINTS.md`
- `ARCHITECTURE.md`
- `CHAT_SESSION_EXPIRY.md`
- `DATABASE.md`
- `DEPLOYMENT_GUIDE.md`
- `HOW_TO_RUN.md`
- `LOGGING_APPROACH.md`
- `PACKAGE_INFO.md`
- `PROJECT_OVERVIEW.md`

## Local Admin Login

The database seed includes a local test admin user for development.

- Email / Username: `testadmin@example.com`
- Password: `TestAdmin123!`

These credentials come from the seed section in `backend-python/Database.sql` and should only be used for local or test environments.

## Backend Summary

The backend provides:

- JWT-based authentication
- admin user management endpoints
- event CRUD APIs
- conversational chat session APIs for event creation and updates
- PostgreSQL-backed persistence

## Entry Points

- App entry: `backend-python/app/main.py`
- Run script: `backend-python/run.py`
- Requirements: `backend-python/requirements.txt`

## Notes

- Chat sessions are stored in PostgreSQL using the `chat_sessions` table.
- Session design and expiry behavior are documented in `CHAT_SESSION_EXPIRY.md`.
- Alembic migration files live under `backend-python/alembic/`.
