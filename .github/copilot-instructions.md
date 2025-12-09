<!-- GitHub Copilot / AI agent guidance for contributors and agents -->
# Quick agent instructions — pms_one

This file contains concise, actionable information for AI coding agents to be productive in this repository.

- **Big picture:** Backend is a FastAPI app (`backend/app`) using SQLModel (ORM + Pydantic). Frontend is a Vite + React + TypeScript app (`frontend/`) with an automatically generated OpenAPI client under `frontend/src/client`. Development and CI use Docker Compose (`docker-compose.yml`) so many workflows run inside containers.

- **Where configuration lives:** Top-level `.env` (used by the backend; referenced from `backend/app/core/config.py` via `env_file="../.env"`). Environment-sensitive behavior (SENTRY, DB credentials, FIRST_SUPERUSER, SECRET_KEY) is enforced in `backend/app/core/config.py`.

- **Key patterns and conventions (concrete examples):**
  - Models / schemas: `backend/app/models.py` uses `SQLModel` classes. Database tables use `class X(..., table=True)` and Pydantic-style request/response models are in the same file (e.g., `UserCreate`, `UserPublic`).
  - API organization: routers are assembled in `backend/app/api` and mounted in `backend/app/main.py` via `app.include_router(api_router, prefix=settings.API_V1_STR)`.
  - CRUD helpers live in `backend/app/crud.py` and are referenced by route handlers.
  - Alembic migrations are in `backend/app/alembic` and are configured to import models from `backend/app/models.py` (run migrations inside the backend container).
  - Generated client: `./scripts/generate-client.sh` produces the OpenAPI client for the frontend; resulting code appears under `frontend/src/client`.
  - Email templates: authoring source MJML files in `backend/app/email-templates/src` and exported HTML in `backend/app/email-templates/build` (use an MJML VS Code extension as indicated in `backend/README.md`).

- **Developer workflows (commands with examples):**
  - Backend local dev inside container (recommended when using Docker Compose):
    - Start dev stack: `docker compose watch` (uses `docker-compose.override.yml` for live-sync and `fastapi run --reload`).
    - Exec into backend: `docker compose exec backend bash`
    - Start development server inside container: `fastapi run --reload app/main.py`
  - Backend tests (local): `bash ./backend/scripts/test.sh` or when stack is up: `docker compose exec backend bash scripts/tests-start.sh` (forwards extra pytest args).
  - Frontend development (local machine):
    - Install Node via `fnm` or `nvm` then `cd frontend && npm install && npm run dev` (server at `http://localhost:5173`).
    - Generate client (top-level): `./scripts/generate-client.sh` (committed output goes to `frontend/src/client`).
  - Playwright E2E: start stack with backend available: `docker compose up -d --wait backend` then `npx playwright test`.

- **Important repo-level conventions to preserve when editing:**
  - Commit generated artifacts: client generation and Alembic revision files are intended to be committed (see frontend README and backend README instructions).
  - Settings are read from a single top-level `.env` (backend `Settings.model_config.env_file` points to `../.env`). Agents must not hardcode credentials — use env vars or suggest secure defaults.
  - Do not remove or rework `docker-compose.override.yml` behavior without updating README; developers rely on the `watch`/volume mount pattern for fast iteration.

- **Integration points / external dependencies to be careful with:**
  - Database: Postgres connection built in `backend/app/core/config.py` (`SQLALCHEMY_DATABASE_URI` uses `postgresql+psycopg`). Alembic migrations run against that DB.
  - Email: SMTP host/user/password drive `emails_enabled` and templates (MJML). Tests use a local Mailcatcher in Docker Compose during CI/local tests.
  - Sentry: only enabled when `SENTRY_DSN` is set and `ENVIRONMENT != "local"` (see `backend/app/main.py`).

- **Where to look for examples when writing code or tests:**
  - API route examples: `backend/app/api/routes/*` (existing route patterns and dependency injection usage).
  - Model + Pydantic patterns: `backend/app/models.py` (typical pattern: Request models, DB model with `table=True`, Public models).
  - Tests: `backend/tests/` and `tests/` at repository root include CI and integration test examples.

- **Helpful small checks AI agents should run before proposing changes:**
  - Confirm `.env` keys used by your change are already present or documented in README/deployment.md.
  - If adding DB columns, create an Alembic revision and include it under `backend/app/alembic/versions/` (follow README migration steps).
  - If changing API schema, update client generation instructions and re-run `./scripts/generate-client.sh` to regenerate `frontend/src/client` and include generated changes in the PR.

If anything here is unclear or you'd like additional examples (e.g., a short snippet showing how to add a CRUD endpoint, or exactly where GitHub Actions run tests), tell me which area to expand and I will iterate.
