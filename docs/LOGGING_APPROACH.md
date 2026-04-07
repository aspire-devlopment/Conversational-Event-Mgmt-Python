# Python Backend Logging Approach

## Overview

The Python backend uses structured logging to support debugging, auditing, performance monitoring, and production issue tracing.

The purpose of this approach is to make backend behavior observable without relying only on ad hoc print statements or manual debugging.

It is designed to answer practical questions like:

- which request failed
- which user action triggered it
- how long a request took
- what safe request payload reached the server
- whether the same failure is repeating across requests

## Main Features

- request and response logging in JSON
- request duration tracking
- trace ID correlation
- sensitive field redaction
- date-wise log storage in a separate folder
- application error persistence to the `error_logs` table

## Why This Logging Design Exists

- JSON logs are easier to search, filter, and ship into log tooling than plain text
- request IDs let engineers connect request logs and error logs for the same flow
- redaction reduces the chance of leaking passwords, tokens, or secrets
- date-based file storage keeps local logs organized
- database-backed error records keep high-value failures queryable even if file logs are inconvenient to inspect

## Key Files

- [logging.py](/e:/ai-conversational-python-backend-repo/backend-python/app/core/logging.py)
- [request_context.py](/e:/ai-conversational-python-backend-repo/backend-python/app/presentation/middleware/request_context.py)
- [auth_context.py](/e:/ai-conversational-python-backend-repo/backend-python/app/presentation/middleware/auth_context.py)
- [main.py](/e:/ai-conversational-python-backend-repo/backend-python/app/main.py)
- [log_repository.py](/e:/ai-conversational-python-backend-repo/backend-python/app/infrastructure/repositories/log_repository.py)

## How Request Logging Works

1. `RequestContextMiddleware` creates or accepts an `X-Request-ID`.
2. The request ID is stored in request state and in a context variable.
3. The middleware captures a safe version of the request body.
4. The request is processed normally.
5. The middleware captures the response body, status code, and duration.
6. One structured access log line is emitted for the request.
7. The response includes the same `X-Request-ID` header.

This makes it easier to trace one request end to end across client reports, API responses, file logs, and database error records.

## How Error Logging Works

When exceptions reach the FastAPI exception handlers in [main.py](/e:/ai-conversational-python-backend-repo/backend-python/app/main.py):

- the backend logs the error in structured JSON
- it includes request metadata such as method, path, status code, trace ID, and safe request body
- it attempts to persist the error into `error_logs`
- if database logging itself fails, the original API response still continues

The app has dedicated handlers for:

- application-level errors
- request validation errors
- FastAPI HTTP exceptions
- unhandled server exceptions

## Redaction And Safety

Sensitive keys are redacted before logs are written.

Examples include:

- `password`
- `password_hash`
- `token`
- `authorization`
- `api_key`
- `access_token`
- `refresh_token`
- `secret`
- `key`

Long strings are also truncated to keep log entries practical and reduce noisy payloads.

## File Storage

Logs are written by date:

```text
backend-python/logs/YYYY-MM-DD/application.log
backend-python/logs/YYYY-MM-DD/error.log
```

## Database Error Logging

High-value application errors are also persisted into:

- `error_logs`

This supports auditing and historical troubleshooting without storing routine request traffic in the database.

## What Gets Logged

- request method
- request path
- status code
- request duration
- client IP when available
- user agent
- safe request body
- safe response body
- trace or request ID
- structured error details for failure cases

## Limits And Tradeoffs

- response bodies are captured for request logging, which is useful for debugging but can add overhead
- routine access logs go to files and console, not to the database
- database logging is focused on errors, not every request
- this is strong application logging, but it is not a full observability stack with metrics and distributed tracing yet

## Practical Benefit

This logging approach helps developers and operators investigate auth issues, broken event writes, failed chat requests, validation errors, and unexpected production exceptions with much less guesswork.
