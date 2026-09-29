# 01 — Claude setup & repo scaffold

**Depends on:** none · **Branch:** `main` (repo has no commits yet; this plan makes the first one)

## Goal
Create the Claude Code configuration and the empty monorepo skeleton so every later session starts with the same context, permissions, and conventions.

## Read first
[../architecture.md](../architecture.md) §2, §3, §7.

## Steps
1. **Folder skeleton** (per architecture §7): `backend/`, `frontend/`, `langflow/flows/`, `langflow/components/`, `infra/`, `scripts/`, `.github/`. Add a `.gitkeep` only where a folder would otherwise be empty.
2. **Root files**
   - `.gitignore`: Python (`.venv`, `__pycache__`, `.pytest_cache`), Node (`node_modules`, `dist`), `.env`, `*.local.*`, `DATA_ROOT` artifacts, `.claude/settings.local.json`, OS/editor files.
   - `.gitattributes`: `* text=auto eol=lf`; `*.ps1 eol=crlf`. LF matters because files are mounted into Linux containers.
   - `.editorconfig`: 2 spaces for TS/JSON/YAML/MD, 4 for Python, final newline.
   - `README.md`: one-paragraph purpose, prerequisites (Docker Desktop, Ollama, Node 24, Python 3.12, uv), links to `docs/`.
   - `VERSION` is **not** created here (release-please owns versioning in plan 02).
3. **`CLAUDE.md`** (root, ≤ 80 lines). Must contain:
   - Project one-liner and link to `docs/architecture.md` and `docs/plans/README.md`.
   - Hard rules: local-only LLMs through Ollama; FastAPI is the only DB/blob owner; no personal data in git; `DATA_ROOT` outside OneDrive; Conventional Commits.
   - How to run: `scripts/dev.ps1 up|down|logs` (created in plan 03), backend `uv run pytest`, frontend `npm test`.
   - Model roles: qwen2.5:14b (tools/extraction), DFM-Mimir (writing), bge-m3 (embeddings), VRAM note (both LLMs cannot be resident at once).
   - Windows notes: PowerShell 5.1 (no `&&`), containers reach Ollama via `host.docker.internal`.
4. **`.claude/settings.json`** (committed, team-shared):
   - `permissions.allow`: read-only and routine commands: `git status/diff/log/add/commit/branch/checkout`, `docker compose *`, `uv run *`, `uv sync`, `npm run *`, `npm ci`, `ollama list/ps/show`, `gh pr *`, `gh run *`.
   - `permissions.deny`: reading `.env`, `**/profile/**`, `**/*.eml`; `git push --force*`.
   - Use the `update-config` skill or the official settings schema; verify keys against Claude Code docs, don't guess.
5. **`.claude/skills/`** — project skills (each a folder with `SKILL.md` + frontmatter `name`, `description`):
   - `execute-plan`: reads `docs/plans/README.md`, picks the requested or next `todo` plan whose deps are `done`, marks it in progress, executes, verifies acceptance criteria, marks done, appends follow-ups.
   - `export-flows`: runs `scripts/export-flows.ps1` (plan 08) and commits changed flow JSON.
   - `release-check`: runs lint + tests for all packages and summarises what release-please will bump.
6. **`.claude/agents/`** — optional subagent definitions, only these two:
   - `reviewer.md`: read-only reviewer checking a diff against CLAUDE.md hard rules.
   - `flow-builder.md`: knows LangFlow 1.12 docs URLs and repo flow conventions (plan 08).
7. **Commit tooling:** `.github/pull_request_template.md` (summary, plan #, test evidence, checklist incl. "no personal data"). Commit-message linting itself is added in plan 02.
8. **`.env.example`** in `infra/` is created in plan 03; here only mention it in README.
9. Initial commit: `chore: initial project scaffold and Claude Code setup` (include `prompts/` and `docs/`).

## Acceptance criteria
- [ ] `git log` shows the initial commit; `git status` clean.
- [ ] `claude` in repo loads CLAUDE.md (check `/memory` or just confirm file location).
- [ ] `/execute-plan` appears as an available skill in a new session.
- [ ] `.claude/settings.json` is valid JSON and matches the documented schema.
- [ ] No file contains personal data.

## Out of scope
CI workflows (02), Docker (03), any application code.
