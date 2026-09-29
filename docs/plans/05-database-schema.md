# 05 — Database schema & migrations

**Depends on:** 03 · **Branch:** `feat/05-schema`

## Goal
Create the `backend/` uv project skeleton plus the app schema in `jobsearch` DB via Alembic.

## Steps
1. `uv init backend --package` (Python 3.12). Deps: `fastapi`, `uvicorn[standard]`, `sqlalchemy>=2`, `psycopg[binary]`, `alembic`, `pydantic-settings`. Dev: `pytest`, `pytest-asyncio`, `ruff`, `mypy`, `testcontainers[postgres]` (or use the CI service container). Keep the `x-release-please-version` marker on `version` (plan 02).
2. `backend/src/ajs/db/models.py` — SQLAlchemy 2.0 typed models. Tables:

   **`dim_company`**: `company_id` PK, `name`, `name_normalized` (unique), `website`, `country`, `created_at`, `updated_at`.

   **`fact_application`** (one row per posting/application, per brief): `application_id` PK, `company_id` FK, `title`, `title_normalized`, `source` (email sender/board), `source_job_id` (nullable), `source_url`, `posting_text`, `posting_language` (`da`/`en`/other), `recruiter_name/email/phone` (nullable), `job_brief` JSONB, `embedding_score`, `rubric_score`, `match_score` (0–100), `match_rationale` text, `status` (enum from architecture §6), `artifact_folder` (path relative to `DATA_ROOT/applications`), `queue_position`, `email_message_id` FK, `created_at`, `updated_at`, `submitted_at`.
   Indexes: unique partial on `source_url` where not null; unique on (`source`, `source_job_id`) where not null; GIN trigram on `company name_normalized` and `title_normalized` (`CREATE EXTENSION pg_trgm`).

   **Supporting tables** (needed by later plans): `email_message` (Message-ID unique, received_at, from, subject, raw path, processed_at), `application_event` (application_id, from_status, to_status, actor `agent|user|system`, note, at), `application_artifact` (application_id, kind `cv|cover_letter|recruiter_email`, version int, md_path, pdf_path, approved bool, created_at), `chat_message` (application_id, artifact kind, role, content, created_at), `workflow_run` (langflow flow id, run id, application_id, phase, status, started/finished, error, phoenix trace id).
3. `normalize()` rules for company/title (lowercase, strip legal suffixes like A/S, ApS, ltd; collapse whitespace) in `ajs/domain/normalize.py`, with unit tests — dedup (A5) depends on it.
4. Alembic: `alembic init`, env reads `DATABASE_URL` from settings; first migration `0001_initial` includes `pg_trgm` extension and enum type. Status enum changes must be migrations.
5. Views for dashboard (plan 11): `v_dashboard_counts` → `reviewed` (all rows except `duplicate`), `good_match` (`match_score >= threshold` — threshold passed as a param in the query, so use a SQL function or compute in API instead of a view; pick one and document).
6. Seed script `backend/scripts/seed_dev.py` with fake companies/applications in every status (no real data) for frontend dev.

## Acceptance criteria
- [ ] `uv run alembic upgrade head` on a clean DB succeeds; `downgrade base` succeeds.
- [ ] `uv run pytest` passes (normalize tests + a migration smoke test).
- [ ] `uv run ruff check` and `mypy` clean.
- [ ] Seed produces rows visible via `psql`.

## Out of scope
API endpoints (06).
