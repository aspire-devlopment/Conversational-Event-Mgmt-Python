# AI Tools And Models Used

## Purpose Of This Document

This document explains the AI-related tools, providers, and implementation choices used in the project, along with why each one matters from a product and business perspective.

## AI Capability In This Project

The AI feature in this platform is focused on one clear business task:

- helping administrators create or update event records through natural-language conversation

The system does not use AI for every part of the platform. AI is concentrated in the chat-assisted event drafting workflow.

## AI Components Used In The Application

### 1. `LlmService`

Location:

- `backend-python/app/application/services/llm_service.py`

Why it is used:

- provides one abstraction layer for all model providers
- hides provider-specific API request formats
- lets the rest of the code use a single structured JSON interface
- reduces vendor lock-in

Business value:

- easier provider switching
- lower future migration cost
- simpler experimentation with cost, speed, and quality tradeoffs

### 2. Structured JSON Prompting

How it is used:

- the backend sends a system prompt instructing the model to return JSON only
- the expected keys are fixed
- extracted values are merged into an event draft

Why it is used:

- easier to validate than free-form text
- reduces parsing ambiguity
- supports predictable backend logic
- fits production use cases better than unstructured chat output

Business value:

- more reliable event creation flow
- less manual cleanup after AI output
- lower operational risk

### 3. Deterministic Pre-Processing Before LLM Call

How it is used:

- the backend attempts direct JSON extraction first
- date normalization, role normalization, timezone normalization, and draft merging are handled with code

Why it is used:

- not every task should spend tokens or rely on the model
- deterministic logic is cheaper and more reliable for clearly structured inputs

Business value:

- lower inference cost
- faster response time
- more stable behavior

## Supported LLM Providers

The backend supports three providers.

### OpenRouter

How it is used:

- OpenAI-compatible chat completion endpoint
- configured through `OPENROUTER_API_KEY`, `OPENROUTER_MODEL`, and `OPENROUTER_API_URL`

Why it is used:

- gives access to many models through one interface
- useful when the business wants flexibility in model selection
- good fit for experimentation across providers

Best business reason:

- strong flexibility and optional multi-model strategy

### Groq

How it is used:

- OpenAI-compatible chat completion endpoint
- configured through `GROQ_API_KEY`, `GROQ_MODEL`, and `GROQ_API_URL`
- set as the default deployment provider in `render.yaml`

Why it is used:

- optimized for very fast inference
- useful for chat workflows where latency strongly affects user experience

Best business reason:

- lower wait time in conversational admin workflows

### Google Gemini

How it is used:

- native Gemini `generateContent` request shape
- configured through `GOOGLE_API_KEY` and `GEMINI_MODEL`

Why it is used:

- provides an alternative provider path
- supports structured JSON generation through response MIME configuration

Best business reason:

- additional provider resilience and model choice diversity

## Model Configuration

The platform is environment-driven. It can change model provider and model name without code changes.

Important environment settings:

- `LLM_PROVIDER`
- `LLM_API_KEY`
- `LLM_MODEL`
- `LLM_TEMPERATURE`
- provider-specific API keys and model names

Current defaults visible in code and deployment config:

- default provider in app settings: `openrouter`
- default generic model in settings: `openrouter/auto`
- default Render deployment provider: `groq`
- default Render model: `llama-3.3-70b-versatile`
- default Gemini model in settings: `gemini-2.5-flash`

## Why These AI Tools Were Chosen

### Provider Abstraction Instead Of Hardcoding One Vendor

Reason:

- lets the business change provider based on price, speed, policy, or availability

Why that matters:

- reduces dependency risk
- supports commercial flexibility
- helps with future procurement and platform evolution

### JSON-Only Response Contract

Reason:

- the backend needs event fields, not creative prose

Why that matters:

- cleaner automation
- fewer parsing errors
- easier validation and safer persistence

### Domain Rules Around The Model

Reason:

- models are useful for extraction and conversational interpretation, but deterministic validation is still needed

Why that matters:

- stronger data quality
- better governance
- less chance of invalid event records entering the database

## AI Workflow In Detail

1. Frontend sends the user's chat message to `/api/chat/message`.
2. Backend loads the saved chat session and current draft.
3. Backend checks whether the message contains structured JSON that can be parsed directly.
4. If not, backend prepares:
   - system prompt
   - recent message history
   - current event summary
   - allowed statuses, timezones, and roles
5. Backend sends the request to the configured provider.
6. Provider returns structured JSON.
7. Backend normalizes and validates the extracted fields.
8. Backend either:
   - asks for the next missing item, or
   - asks for explicit confirmation
9. After confirmation, the backend creates or updates the event.

## What AI Does Not Currently Do

- it does not auto-publish without explicit confirmation
- it does not search external knowledge sources
- it does not generate banners or marketing copy automatically
- it does not rank business impact or recommend strategy
- it does not support streaming token output
- it does not provide advanced multilingual coverage beyond the current supported languages

## Risks And Limitations Of The Current AI Design

### Structured Output Dependency

The backend expects machine-readable JSON. If a provider returns malformed content, the flow can fail.

### Limited Context Window Usage

Only recent conversation history is passed to the model, which is practical but may lose some older context in very long sessions.

### No Retrieval-Augmented Grounding

The assistant works from the conversation and fixed rules only. It is not grounded in company policies, historical event templates, or document retrieval.

### Provider Latency And Availability

The experience still depends on external provider uptime and response speed.

### Domain Scope Is Narrow

The assistant is specialized for event drafting, not broad enterprise copilot use.

## Recommended Future AI Enhancements

1. Add streaming responses for a better chat experience.
2. Add schema validation and fallback retry when model JSON is malformed.
3. Add template retrieval from prior successful events.
4. Add confidence thresholds and human review routing.
5. Add multilingual expansion and better language detection.
6. Add event copy suggestions, banner suggestions, or form autofill recommendations.

## AI Tools Used For Preparing These Repository Documents

For this documentation pass, the repository itself was analyzed through local code inspection:

- FastAPI service files
- React frontend integration files
- repository and middleware layers
- deployment manifests
- existing documentation files

The reason for using direct repository inspection instead of assumptions was to ensure the business and architecture write-up stayed aligned with the actual implementation.
