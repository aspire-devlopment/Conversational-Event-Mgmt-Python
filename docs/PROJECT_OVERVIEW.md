# Project Overview

## Executive Summary

This project is an AI-assisted event management platform built as a monorepo with a React frontend and a FastAPI backend. Its main business value is that it reduces the effort required to create and manage virtual events by combining standard admin workflows with a conversational AI assistant that can collect event details, validate them, and save them into the event database.

The platform supports:

- user registration and login
- role-based access control
- admin user management
- event creation, update, listing, and deletion
- AI-guided event drafting through chat
- multilingual conversational guidance in English, German, and French
- deployment-ready backend and frontend configuration

## What The System Does In Detail

At a practical level, the system gives organizations a web application where authenticated users can access event information and administrators can manage the platform. The differentiating feature is the AI chat workflow for event creation and update.

Instead of forcing an administrator to fill every event field manually, the assistant can guide the user through a conversation such as:

- naming the event
- adding the subheading and description
- choosing the banner image URL
- selecting the timezone and publication status
- defining start, end, and vanish dates
- assigning which user roles can access the event

The backend turns that conversation into a structured event draft, validates the required fields, asks follow-up questions for missing information, and only saves the event after explicit confirmation.

This makes the product useful for teams that want:

- faster event creation
- lower operational effort
- a more intuitive admin experience
- a more guided workflow for non-technical users
- a reusable backend that can support both manual and conversational operations

## Business Purpose

From a business perspective, the platform solves two connected problems:

1. Event operations need structured governance.
   Organizations need controlled user roles, secure authentication, and a central place to manage events.

2. Event setup is repetitive and time-consuming.
   AI-assisted drafting reduces friction by letting admins describe an event in natural language instead of completing a long form from scratch.

This makes the product relevant for:

- internal corporate event teams
- training and enablement groups
- webinar and virtual conference teams
- partner enablement programs
- sales kickoff and campaign operations

## Monorepo Structure

The repository is organized into two main applications plus deployment configuration.

```text
ai-conversational-python-backend-repo/
+-- backend-python/
|   +-- app/
|   |   +-- application/        # Use-case orchestration
|   |   +-- core/               # Config, security, logging, DB helpers
|   |   +-- domain/             # Pure chat and validation logic
|   |   +-- infrastructure/     # PostgreSQL repositories
|   |   `-- presentation/       # FastAPI routes, schemas, middleware
|   +-- alembic/                # Migration scaffolding
|   +-- docs/                   # Backend-specific technical docs
|   +-- requirements.txt
|   `-- run.py
+-- frontend/
|   +-- public/
|   +-- src/
|   |   +-- components/         # Public, auth, and admin UI
|   |   +-- context/            # Auth state
|   |   +-- services/           # API client
|   |   +-- hooks/
|   |   `-- utils/
|   +-- package.json
|   `-- tailwind.config.js
+-- docs/                       # Cross-project business and architecture docs
+-- render.yaml                 # Backend deployment
`-- vercel.json                 # Frontend deployment
```

## Functional Modules

### 1. Authentication and Access

The backend supports:

- user registration for Manager, Sales Rep, and Viewer roles
- JWT-based login
- authenticated profile retrieval
- admin-only user listing
- admin password reset for other users

Business impact:

- secures access to the platform
- separates administrative authority from general access
- supports operational governance in shared environments

### 2. Event Management

The backend supports:

- create event
- edit event
- delete event
- list events
- fetch a single event
- assign visibility roles to each event

Business impact:

- allows central event ownership
- supports event targeting by user role
- makes event visibility manageable without duplicating data

### 3. Conversational AI Event Drafting

The AI chat module supports:

- create-mode chat session for new events
- update-mode chat session for existing events
- session persistence in PostgreSQL
- natural-language extraction of event attributes
- direct JSON paste support for faster admin workflows
- explicit confirmation before save

Business impact:

- reduces time to create an event
- lowers form fatigue for admins
- supports iterative correction through chat
- increases adoption by making the workflow easier to use

### 4. Logging and Operational Visibility

The backend includes:

- request correlation IDs
- structured request and response logging
- persistent error logging in the database
- security headers
- trusted host enforcement

Business impact:

- improves auditability
- helps production support teams troubleshoot issues
- increases readiness for deployment in controlled environments

## Project Workflow

### User Workflow

1. A user registers or logs in.
2. The frontend stores the JWT token and user profile in local storage.
3. Protected routes become available based on authentication state.
4. The user accesses admin modules if authorized.

### Admin Event Workflow

1. Admin opens the event area.
2. Admin can create an event manually through API-backed UI actions.
3. Admin can edit or delete existing events.
4. Role assignments determine which users can view each event.

### AI Chat Workflow

1. Admin opens the AI Event Creation Assistant.
2. Frontend creates or restores a chat session.
3. Backend initializes a draft with mode, language, and next required field.
4. Admin sends natural-language instructions.
5. Backend either:
   - parses direct JSON input immediately, or
   - sends context to the configured LLM provider
6. Backend merges extracted values into the draft.
7. Domain validation checks missing fields and date rules.
8. Backend asks the next question or moves to confirmation.
9. Admin confirms creation or update.
10. Backend writes the event and role mappings to PostgreSQL.
11. Chat session is deleted after successful completion.

## Real Use Cases

### Corporate Training Programs

An HR or enablement team can quickly create role-specific training events and publish them only to Managers or Sales Reps.

### Sales Campaign Launches

A revenue operations team can use chat to create campaign kickoff sessions with dates, banner image, access roles, and publish status in a few prompts.

### Multi-Regional Webinar Operations

The assistant supports timezone handling and multilingual interaction, which helps distributed teams create events more naturally.

### Admin Productivity Improvement

Teams with frequent event setup can use the AI assistant to convert rough instructions or pasted JSON into a complete event draft faster than manual entry.

## LLMs Used In The Project

The backend supports multiple LLM providers behind one service abstraction:

- OpenRouter
- Groq
- Google Gemini

The provider is selected through environment configuration. In the included deployment configuration, the backend is set to use Groq by default on Render.

For the conversational event-drafting workflow specifically:

- the included Render deployment uses Groq with the `llama-3.3-70b-versatile` model
- the codebase also supports OpenRouter with `openrouter/auto`
- the codebase also supports Google Gemini with `gemini-2.5-flash`

This means the conversational assistant is not tied to one vendor, but the current deployment-ready default is Groq `llama-3.3-70b-versatile`.

Business reason for this design:

- avoids vendor lock-in
- gives flexibility on cost, speed, and model quality
- allows future provider replacement without changing business workflows or frontend APIs

More detail is documented in [AI_TOOLS_AND_MODELS.md](/e:/ai-conversational-python-backend-repo/docs/AI_TOOLS_AND_MODELS.md).

## Database Used In The Project

The backend uses PostgreSQL as its primary database.

That database stores:

- application users and roles
- events and event-role mappings
- conversational chat sessions
- persistent error logs

For the conversational workflow, PostgreSQL is not just the final event store. It also keeps the in-progress assistant session in the `chat_sessions` table, including conversation history and the evolving event draft, so admins can refresh and continue their work.

## Current Limitations

### Product Limitations

- The conversational assistant is focused on event drafting, not broad general-purpose support.
- The supported language set is limited to English, German, and French.
- Timezone suggestions are limited to a predefined list, even though aliases are partially normalized.
- Role visibility logic is applied in the application layer after fetching events, which may become inefficient at larger scale.
- Chat sessions expire after 24 hours.
- The frontend sends an `Idempotency-Key` header for event creation, but the current FastAPI backend does not yet persist or enforce idempotency handling.

### Technical Limitations

- No background job queue exists for retries, async notifications, or long-running AI workflows.
- No streaming chat response path is implemented.
- Event visibility filtering is not pushed fully into SQL.
- The LLM output contract depends on JSON compliance from the provider response.
- There is no vector search, knowledge base, or retrieval-augmented generation layer.
- No comprehensive automated test suite is present in this repository snapshot for backend business flows.

## Future Enhancements

### Business-Facing Enhancements

- approval workflow before publishing events
- richer event analytics and reporting
- calendar and notification integrations
- self-service event templates
- multi-tenant organization support
- localization expansion beyond the current three languages

### AI Enhancements

- streaming chat responses
- better prompt orchestration and guardrails
- confidence-based review routing
- retrieval over prior event templates and company policy
- automatic banner and content suggestions
- deeper multilingual support

### Platform Enhancements

- stronger idempotency support
- role-aware filtering at the database layer
- caching for high-read event lists
- background workers for operational jobs
- expanded monitoring and alerting
- more complete automated tests and CI quality gates

## Why This Project Matters For Business Stakeholders

This platform is more than a CRUD event tool. It combines structured operations with guided AI interaction, which can improve speed, consistency, and usability for internal teams. It is especially valuable where event configuration is frequent, role-sensitive, and operationally repetitive.

In short, the system helps teams create and manage events faster while still keeping control over users, permissions, and data quality.
