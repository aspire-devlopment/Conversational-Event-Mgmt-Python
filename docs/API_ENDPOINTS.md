# Python Backend API Endpoints

## Base URL

Local default:

```text
http://localhost:8000
```

Frontend API base:

```text
http://localhost:8000/api
```

## Purpose Of The API

This backend API exists to support four main business capabilities:

- authentication and identity for platform users
- admin-only user management
- event CRUD and event visibility control
- conversational event creation and update through chat

The frontend is the main client for these endpoints, but the API is structured clearly enough for other clients or internal integrations later.

## Response Style

Most routes return a consistent envelope:

```json
{
  "status": "success",
  "message": "Optional message",
  "data": {}
}
```

The chat session routes are a partial exception because they preserve the response shapes expected by the current React chat UI.

## Health

- `GET /health`

Purpose:

- verifies that the FastAPI service is up
- useful for Docker, Render, uptime checks, and smoke tests

## Authentication

- `POST /api/auth/login`
- `POST /api/auth/register`
- `POST /api/auth/logout`
- `GET /api/auth/me`

Purpose:

- `POST /api/auth/login`: authenticates a user and returns the auth payload used by the frontend session flow
- `POST /api/auth/register`: creates a new user account
- `POST /api/auth/logout`: validates an authenticated logout action for the current session flow
- `GET /api/auth/me`: returns the current authenticated user profile

Why this group exists:

- centralizes identity, JWT-based access, and profile retrieval
- gives the frontend one predictable auth contract
- supports role-aware access to admin and event features

## Admin

- `GET /api/admin/users?page=1&pageSize=20`
- `POST /api/admin/users/{user_id}/reset-password`

Purpose:

- `GET /api/admin/users`: lists users with bounded pagination for admin dashboards
- `POST /api/admin/users/{user_id}/reset-password`: lets an admin reset another user's password

Why this group exists:

- supports platform governance and user lifecycle operations
- keeps sensitive user-management actions behind the `Admin` role

## Events

- `GET /api/events?page=1&pageSize=20`
- `GET /api/events/{event_id}`
- `POST /api/events`
- `PUT /api/events/{event_id}`
- `DELETE /api/events/{event_id}`

Purpose:

- `GET /api/events`: lists events visible to the authenticated user
- `GET /api/events/{event_id}`: fetches one event if the user has access
- `POST /api/events`: creates a new event and role mappings
- `PUT /api/events/{event_id}`: updates an event and its role mappings
- `DELETE /api/events/{event_id}`: deletes an event

Why this group exists:

- provides the core event-management capability of the platform
- supports both manual admin workflows and the final persistence step of the chat workflow
- enforces visibility rules based on role and ownership

## Chat

- `POST /api/chat/session`
- `GET /api/chat/session/{session_id}`
- `POST /api/chat/message`
- `DELETE /api/chat/session/{session_id}`

Purpose:

- `POST /api/chat/session`: creates a new conversational drafting session for event create or update mode
- `GET /api/chat/session/{session_id}`: restores an existing chat session
- `POST /api/chat/message`: sends a user message into the assistant workflow and returns the next assistant response
- `DELETE /api/chat/session/{session_id}`: removes a chat session

Why this group exists:

- powers the AI-assisted event drafting flow
- persists draft state across page refreshes and multiple messages
- allows the assistant to gather event fields step by step before final save

## Authorization Notes

- Auth uses JWT bearer tokens.
- Admin routes require an authenticated user with the `Admin` role.
- Event routes require authentication.
- Event and admin listing endpoints support pagination.
- Chat routes require authentication and enforce session ownership.

## Operational Notes

- `POST /api/events` is where the frontend sends an `Idempotency-Key` header, although the current Python backend does not yet persist or enforce idempotency handling.
- The chat flow can either parse direct JSON input or call the configured LLM provider.
- The current backend health route is `/health`, while most business routes live under `/api/...`.
