# Implementation Plans

Each plan is sized for **one agent session**. Read [../architecture.md](../architecture.md) first; it holds verified facts, decisions (D1–D8) and assumptions (A1–A6) that every plan relies on. Do not re-decide them; if one proves wrong, stop and ask the user, then update architecture.md.

## Order & status

| # | Plan | Depends on | Status |
|---|---|---|---|
| 01 | [Claude setup & repo scaffold](01-claude-setup-and-scaffold.md) | – | done |
| 02 | [CI/CD & release versioning](02-ci-cd-and-releases.md) | 01 | in progress |
| 03 | [Docker infrastructure](03-docker-infrastructure.md) | 01 | todo |
| 04 | [Local models setup & evaluation](04-local-models.md) | 03 | todo |
| 05 | [Database schema & migrations](05-database-schema.md) | 03 | todo |
| 06 | [FastAPI core & tool endpoints](06-fastapi-core.md) | 05 | todo |
| 07 | [Email intake & posting fetcher](07-email-intake.md) | 06 | todo |
| 08 | [LangFlow flows: intake → match](08-langflow-analysis-flows.md) | 04, 06 | todo |
| 09 | [LangFlow flows: document generation](09-langflow-generation-flows.md) | 08 | todo |
| 10 | [Observability & governance](10-observability-governance.md) | 08 | todo |
| 11 | [Frontend scaffold & dashboard](11-frontend-dashboard.md) | 06 | todo |
| 12 | [Frontend review tab & chat editing](12-frontend-review.md) | 09, 11 | todo |
| 13 | [Frontend queue tab](13-frontend-queue.md) | 11 | todo |
| 14 | [End-to-end test & v0.1.0 release](14-e2e-and-first-release.md) | all | todo |

Parallelizable after 06: {07, 08, 11}; after 11: {12 (needs 09), 13}.

## Rules for executing agents
1. Set this plan's status to `in progress` at start and `done` at end (in this table).
2. Stay in scope. Out-of-scope discoveries go under **Follow-ups** at the bottom of this file.
3. Work on a branch `feat/NN-short-name`; Conventional Commits (`feat:`, `fix:`, `chore:`, `docs:`, `ci:`); open a PR once plan 02 is done.
4. Every plan ends with its **Acceptance criteria** checked and verified by running the listed commands; report failures honestly.
5. No cloud LLM SDKs or API keys anywhere in the app (governance rule; enforced by plan 10).
6. Never commit personal data (CV, templates, emails, `.env`).

## Follow-ups
_(append here)_
- (02) Plans 05 and 11 mention an `x-release-please-version` marker. It is not needed: `release-please-config.json` updates `project.version` in `backend/pyproject.toml` and `version` in `frontend/package.json` through toml/json `extra-files`.
- (02 → 05) CI runs `uv run mypy` with no arguments, so `backend/pyproject.toml` needs `[tool.mypy] files = [...]`. CI provides `DATABASE_URL=postgresql+psycopg://jobsearch:jobsearch@localhost:5432/jobsearch_test` (Postgres 17 service).
- (02 → 06, 11) The release workflow builds images with the package directory (`backend/`, `frontend/`) as the Docker build context, so Dockerfiles must not reference files outside it.
- (02 → 14) The release PR is opened with `GITHUB_TOKEN`, so no checks run on it. Either merge it as admin (the ruleset allows admin bypass on PRs only) or add a fine-grained PAT secret `RELEASE_PLEASE_TOKEN`. See `docs/releasing.md`.
- (02) On this machine git's OpenSSL backend fails TLS to github.com (likely HTTPS interception by antivirus or a proxy). The repo-local git config sets `http.sslBackend=schannel`, and `uvx` needs `--system-certs`.
