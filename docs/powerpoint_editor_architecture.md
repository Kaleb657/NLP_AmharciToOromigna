# AI-Powered PowerPoint Web Editor Architecture

This document captures a production-ready architecture for a web PowerPoint editor with AI-assisted generation.

## Core Stack
- Backend: Python Quart + python-pptx
- Frontend: React 18 + Vite + Axios
- AI providers: OpenAI, Gemini, DeepSeek via one adapter layer

## Backend Design
- `routes/`: HTTP transport and request/response handling only.
- `services/`: business logic (slide operations, AI orchestration, preview rendering).
- `utils/`: validation, file safety, and session utilities.

Recommended boundaries:
- Routes should never call provider SDKs directly.
- AI output must be schema-validated before generator logic.
- Storage paths should be centralized in one config module.

## API Surface
- Upload and analysis endpoints for existing `.pptx` files.
- Slide mutation endpoints (move/delete/duplicate/text update).
- Generation endpoints for preview and final PPTX.
- Template listing and download endpoints.

## AI Generation Contract
Require model output as strict JSON array of slide objects:
- `type`: one of `title|bullet|section|two-column|image|quote`
- `title`: concise string
- `bullets`: array of short strings

Validation rules:
- Slide 1 must be `title`
- Section cadence every 5 slides
- Final slide summarizes takeaways

## Frontend Design
- Page-level split (`Home`, `Editor`, `Generator`) with reusable component modules.
- API calls encapsulated in `services/` + `hooks/`.
- Global session/slide state in context, transient UI state local to components.

## Reliability and Security
- File type + extension checks; maximum upload size.
- Filename/path sanitization to prevent traversal.
- Explicit error mapping (`400`, `404`, `500`) with user-friendly messages.
- Rate limiting and provider timeouts for AI endpoints.

## Scale Path
- Move in-memory session storage to Redis.
- Offload long generation jobs to Celery/worker queue.
- Add caching for templates and deterministic AI previews.

## Suggested Immediate Next Steps
1. Define JSON schema for AI output and enforce in backend.
2. Add idempotent job endpoint for long-running generation.
3. Add integration tests for all slide mutation endpoints.
4. Add structured logging with request/session correlation IDs.
