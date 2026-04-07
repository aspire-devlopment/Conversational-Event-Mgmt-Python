# Detailed Architecture

## Architecture Summary

This project uses a two-application architecture:

- `frontend/`: React web application for public pages, authentication, admin event management, and AI chat interaction
- `backend-python/`: FastAPI service for authentication, admin operations, event APIs, conversational event orchestration, and PostgreSQL persistence

The overall design is API-first. The frontend is a presentation client, while the backend owns authentication, business rules, AI orchestration, validation, and persistence.

## High-Level Architecture

```text
Browser
  |
  v
React Frontend
  |
  | HTTPS / JSON APIs
  v
FastAPI Backend
  |
  +--> Auth + Middleware
  +--> Application Services
  +--> Domain Validation / Draft Logic
  +--> Repository Layer
  |
  +--> PostgreSQL
  |
  +--> External LLM Provider
       - Groq
       - OpenRouter
       - Gemini
```

## Frontend Architecture

The frontend is a React application using:

- React 19
- React Router
- Tailwind CSS
- a centralized API client
- an auth context for local session state

### Frontend Responsibilities

- render public pages and auth pages
- protect admin routes
- store JWT token and user payload in local storage
- call backend APIs
- maintain chat UI state
- restore active chat sessions between page refreshes

### Important Frontend Areas

- `src/context/AuthContext.jsx`
  Handles login state and local token storage.
- `src/services/api.js`
  Centralizes backend API calls for auth, admin, events, and chat.
- `src/components/admin/AdminChatPage.jsx`
  Main AI assistant UI for conversational event creation and update.
- `src/App.js`
  Defines public and protected route structure.

## Backend Architecture

The backend follows a layered architecture.

```text
Presentation Layer
  -> Application Layer
     -> Domain Layer
     -> Infrastructure Layer
        -> PostgreSQL
  -> External LLM Provider
```

### 1. Presentation Layer

Location:

- `backend-python/app/presentation`

Responsibilities:

- define HTTP routes
- validate request payloads with Pydantic
- enforce auth dependencies
- apply middleware
- shape JSON responses for frontend compatibility

Key modules:

- `routes/auth.py`
- `routes/admin.py`
- `routes/events.py`
- `routes/chat.py`
- `dependencies/auth.py`
- `middleware/request_context.py`
- `middleware/auth_context.py`
- `middleware/security.py`

### 2. Application Layer

Location:

- `backend-python/app/application/services`

Responsibilities:

- orchestrate use cases
- coordinate repositories and domain utilities
- manage chat session lifecycle
- handle provider-based LLM invocation

Key services:

- `auth_service.py`
- `event_service.py`
- `chat_service.py`
- `llm_service.py`

### 3. Domain Layer

Location:

- `backend-python/app/domain`

Responsibilities:

- keep pure business logic independent from HTTP and SQL concerns
- normalize event drafts
- detect language
- parse dates
- validate event completeness and consistency
- define constants for roles, statuses, and supported timezones

Key modules:

- `chat_utils.py`
- `constants.py`

### 4. Infrastructure Layer

Location:

- `backend-python/app/infrastructure/repositories`

Responsibilities:

- execute SQL queries
- manage event-role synchronization
- persist chat sessions
- fetch users and roles
- write error logs

Key repositories:

- `user_repository.py`
- `role_repository.py`
- `event_repository.py`
- `chat_session_repository.py`
- `log_repository.py`

### 5. Core Layer

Location:

- `backend-python/app/core`

Responsibilities:

- environment-based configuration
- database connection management
- JWT and password security
- shared exception types
- logging helpers

Key modules:

- `config.py`
- `database.py`
- `security.py`
- `logging.py`
- `exceptions.py`

## Request Lifecycle

### Standard API Request

1. Browser sends request to FastAPI.
2. `RequestContextMiddleware` creates a request ID, captures safe request/response bodies, and logs the transaction.
3. `AuthContextMiddleware` reads the bearer token and loads the current user from PostgreSQL.
4. Route dependency checks whether the request requires authentication or admin role.
5. Pydantic schema validates input.
6. Application service executes business logic.
7. Repository performs SQL operations.
8. Response is returned as JSON.
9. Security headers are appended.

### Error Flow

1. An exception is raised in route or service logic.
2. FastAPI exception handler maps it into a structured error response.
3. Error context is logged.
4. `LogRepository` attempts to persist the error into `error_logs`.

## Authentication Architecture

### Login Flow

1. User submits email and password.
2. `AuthService.login` looks up the user by email.
3. Password is verified with bcrypt.
4. A JWT token is created with user identity and role.
5. Frontend stores the token in local storage.
6. Subsequent requests include `Authorization: Bearer <token>`.

### Authorization Model

Roles in the system:

- `Admin`
- `Manager`
- `Sales Rep`
- `Viewer`

Authorization behavior:

- admin routes require `Admin`
- event visibility allows:
  - admins to see all events
  - event creators to see their own events
  - users whose role matches event role assignments to see the event

## Conversational AI Architecture

The AI assistant is the most important architectural feature in this system.

In the included deployment configuration, this conversational layer is wired to Groq with the `llama-3.3-70b-versatile` model by default. The same service abstraction can also switch to OpenRouter or Google Gemini through environment variables without changing application code.

### Chat Session State

Each chat session stores:

- `conversation_history`
- `event_draft`
- `current_step`
- `language`
- `state`
- `mode`
- `event_id` for update sessions
- timestamps and expiry metadata

This state lives in the `chat_sessions` table as JSONB plus indexed metadata fields.

### Chat Processing Flow

```text
Admin message
  -> Chat route
  -> ChatService.send_message()
  -> Load session
  -> Add user message to history
  -> Process message
     -> direct JSON parse if possible
     -> otherwise call LLM provider
  -> Merge extracted values into draft
  -> Validate draft
  -> Ask next question or request confirmation
  -> On explicit confirmation:
       create/update event in PostgreSQL
       delete chat session
```

### AI Decision Logic

The backend first tries a deterministic shortcut:

- if the user pasted valid JSON with event fields, the backend parses it directly without asking the LLM

If not, the backend constructs:

- a system prompt
- recent message history
- current event draft summary
- constraints such as statuses, timezones, and roles

It then asks the configured LLM to return JSON only with this contract:

- `intent`
- `language`
- `extractedData`
- `changedFields`
- `nextStep`
- `message`
- `confidence`

This structured response is then normalized and merged with the current event draft.

### Event Draft Lifecycle

The event draft evolves across the conversation:

1. start with empty draft
2. fill fields from message or JSON
3. normalize date/time and enums
4. auto-derive end time or vanish time in some cases
5. validate completeness and chronology
6. request corrections if needed
7. ask for explicit confirmation
8. persist into `events` and `event_roles`

## Database Architecture

The application uses PostgreSQL as the primary system of record.

PostgreSQL also backs the conversational workflow itself, not just the final event records. In-progress chat state, conversation history, and draft event data are persisted in the `chat_sessions` table using JSONB fields so the assistant can resume across refreshes and multi-turn interactions.

### Core Tables Referenced In Code

- `roles`
- `users`
- `events`
- `event_roles`
- `chat_sessions`
- `error_logs`

The documentation also references `idempotency_keys` as part of the shared schema, although the current Python backend does not actively use it.

## Database Flow

### User Flow

1. User registers.
2. Backend looks up target role in `roles`.
3. Backend inserts a row into `users`.
4. Login later reads `users` joined with `roles`.

### Event Creation Flow

1. Validated payload reaches `EventRepository.create_with_roles`.
2. Transaction starts.
3. Row is inserted into `events`.
4. Selected roles are synchronized into `event_roles`.
5. Transaction commits.
6. Event is read back with aggregated role names.

### Event Update Flow

1. Transaction starts.
2. Target `events` row is locked with `FOR UPDATE`.
3. Event columns are updated.
4. Existing `event_roles` links are replaced deterministically.
5. Transaction commits.

### Chat Session Flow

1. `chat_sessions` cleanup removes expired rows.
2. New session is inserted with UUID and JSONB session state.
3. Messages append into `session_data.conversation_history`.
4. Session draft and current step are updated after every turn.
5. Session is deleted after successful event creation or update.

### Error Logging Flow

1. Exception occurs.
2. Error payload is assembled.
3. Row is inserted into `error_logs`.
4. If logging persistence fails, the original API response still continues.

## Deployment Architecture

### Frontend Deployment

Configured through `vercel.json`:

- builds from `frontend/`
- outputs static build assets
- rewrites SPA routes to `index.html`

### Backend Deployment

Configured through `render.yaml`:

- deploys `backend-python/` as a Python web service
- installs dependencies from `requirements.txt`
- starts `uvicorn app.main:app`
- exposes `/health` for health checking

### Runtime Configuration

Environment variables drive:

- application mode
- CORS origins
- trusted hosts
- database connection
- JWT secret and expiration
- LLM provider and model selection
- provider API keys and timeouts

## Security Architecture

Implemented controls include:

- JWT bearer authentication
- bcrypt password hashing
- admin route enforcement
- security response headers
- trusted host filtering
- optional HTTPS redirect
- structured request logging with sensitive-data redaction

Notable security gaps or design decisions:

- frontend stores JWT in local storage rather than HTTP-only cookies
- no explicit rate-limiting layer appears in this repository
- no CSRF design is needed for bearer-token API style, but token storage choice has browser risk tradeoffs

## Scalability Considerations

Current design strengths:

- clear separation of concerns
- provider abstraction for LLM calls
- PostgreSQL-backed session persistence
- transactional event writes

Current scale constraints:

- event visibility filtering happens after fetching many rows
- chat relies on synchronous request/response LLM calls
- no queue or worker layer for async processing
- no cache layer is present

## Limitations In The Current Architecture

- direct dependency on structured JSON output from the model
- no streaming responses to improve chat UX
- no background processing for retries or analytics
- no retrieval augmentation or enterprise knowledge grounding
- stale links remain in some older backend docs
- testing and CI enforcement are not yet documented as mature platform capabilities

## Recommended Next Architecture Steps

1. Move role-based event visibility into SQL for better scalability.
2. Add real idempotency support for create/update operations.
3. Introduce observability tooling such as metrics and tracing export.
4. Add asynchronous jobs for notifications, retries, and AI post-processing.
5. Add streaming chat support.
6. Consider cookie-based auth or stronger token handling strategy if browser threat model becomes stricter.
7. Expand automated test coverage for auth, chat, and repository behavior.
