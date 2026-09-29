---
name: release-check
description: Run lint and tests for every package and summarise what release-please would bump. Use before merging to main or when the user asks "are we release ready".
---

# Release check

1. Backend (if `backend/pyproject.toml` exists): `cd backend; uv run ruff check .; uv run pytest`.
2. Frontend (if `frontend/package.json` exists): `cd frontend; npm run lint; npm test`.
3. Skip packages that do not exist yet and say so. Report failures with the relevant output; do not hide them.
4. Run `git log --oneline <last-tag>..HEAD` (or the full log if there is no tag) and classify commits by Conventional Commit type: `feat` → minor, `fix` → patch, `!` or `BREAKING CHANGE` → major (minor while on 0.x, per release-please config). Flag commits that are not Conventional.
5. Summarise: test status per package, the expected next version, and the changelog entries.
