# 02 — CI/CD & release versioning

**Depends on:** 01 · **Branch:** `ci/02-releases`

## Goal
GitHub repo with CI on every PR and automated SemVer releases from Conventional Commits (D7).

## Prerequisites (ask user if missing)
- Install gh: `winget install --id GitHub.cli`, then user runs `gh auth login` (interactive — ask the user to do it).
- User confirms repo name and visibility (recommend **private**; repo will contain prompts describing their job search).

## Steps
1. Create remote: `gh repo create <name> --private --source . --push`.
2. **Branch protection** on `main` (via `gh api`): require PR, require status checks `ci / *`, linear history, no force push.
3. **CI workflow** `.github/workflows/ci.yml`, triggered on `pull_request` and `push: main`:
   - `changes` job with `dorny/paths-filter` → outputs `backend`, `frontend`, `langflow`, `infra`.
   - `backend`: `astral-sh/setup-uv`, `uv sync --frozen`, `uv run ruff check`, `uv run ruff format --check`, `uv run mypy`, `uv run pytest` (Postgres service container `postgres:17`).
   - `frontend`: `actions/setup-node` (node 24, npm cache), `npm ci`, `npm run lint`, `npm run typecheck`, `npm test -- --run`, `npm run build`.
   - `langflow`: validate every `langflow/flows/*.json` parses and passes the governance check script (added in plan 10; job is a no-op until the script exists).
   - `infra`: `docker compose -f infra/docker-compose.yml config -q`.
   - Jobs whose package does not exist yet must skip cleanly (guard with `hashFiles`).
4. **Commit linting:** `.github/workflows/pr-title.yml` using `amannn/action-semantic-pull-request` (squash-merge makes the PR title the commit). Set repo to squash-merge only.
5. **Releases:** `googleapis/release-please-action@v4` in `.github/workflows/release.yml` on push to `main`.
   - `release-please-config.json`: single package at root, `release-type: simple`, `bump-minor-pre-major: true`, changelog sections for feat/fix/perf/docs.
   - `.release-please-manifest.json`: `{ ".": "0.0.0" }`.
   - `extra-files` so the version is stamped into `backend/pyproject.toml` and `frontend/package.json` (use generic updater markers `x-release-please-version` where needed).
6. **Release artifacts** (job in release.yml, runs only when a release is created): build backend and frontend images and push to GHCR as `ghcr.io/<owner>/ajs-backend:<version>` and `:latest`. LangFlow flows are attached to the GitHub Release as a zip.
7. **Dependabot** `.github/dependabot.yml`: pip (uv), npm, github-actions, docker; weekly.
8. Document the release flow in `docs/releasing.md` (≤ 30 lines): merge PRs → release-please opens release PR → merge → tag `vX.Y.Z` + CHANGELOG + images.

## Acceptance criteria
- [ ] A test PR shows `ci` and PR-title checks; all pass or skip.
- [ ] After merging, release-please opens a release PR proposing `0.1.0` (or `0.0.1`); do **not** merge it yet (first release in plan 14).
- [ ] `main` rejects direct pushes.

## Out of scope
Deployment to any server (local-only system).
