# 10 — Observability & governance

**Depends on:** 08 (09 recommended) · **Branch:** `feat/10-observability`

## Goal
End-to-end traces in Arize Phoenix and enforceable governance rules (local-only LLMs, no PII leaks, prompt versioning).

## Steps
1. **LangFlow → Phoenix:** docs state a local Phoenix at `http://localhost:6006` needs no API key but *non-local endpoints require `PHOENIX_API_KEY`* (https://docs.langflow.org/integrations-arize). Inside compose the endpoint is `http://phoenix:6006`; verify traces arrive. If LangFlow demands a key, enable Phoenix auth (`PHOENIX_ENABLE_AUTH`, secret) and issue a system API key; document in CLAUDE.md.
2. **FastAPI → Phoenix:** OpenTelemetry (`opentelemetry-instrumentation-fastapi`, `-httpx`, `-sqlalchemy`, OTLP exporter to `http://phoenix:6006/v1/traces`). Propagate trace context into LangFlow calls where supported; always store LangFlow run id + trace id in `workflow_run` so UI can link to Phoenix.
3. **Projects:** Phoenix project names `ajs-langflow`, `ajs-backend`.
4. **Governance checks** `scripts/governance_check.py` (run in CI langflow job + pre-commit):
   - every model component in `langflow/flows/*.json` is Ollama-based; base URL is the `OLLAMA_BASE_URL` variable;
   - no strings matching cloud-LLM SDKs/keys (`openai`, `anthropic`, `api.openai.com`, `sk-…`) in `backend/`, `langflow/`, `frontend/src` (allowlist docs);
   - prompts in flows match `langflow/prompts/*.md` (hash compare) — prompt changes must go through PR;
   - no files from `profile/`, `.eml`, or `.env` tracked by git.
5. **Egress guard (runtime):** backend refuses outbound HTTP except the posting fetcher's allowed use and IMAP; LangFlow container gets no extra secrets. Document that Ollama is the only model endpoint.
6. **PII & retention:** traces contain CV and posting text. Phoenix stays local; add a retention setting (e.g. 90 days) and `scripts/dev.ps1 prune-traces`. Note in docs.
7. **Evals in Phoenix:** upload the golden set from plan 08 as a Phoenix dataset; record experiment runs when prompts/models change (manual step documented in `docs/model-evaluation.md`).
8. **`docs/governance.md`** (≤ 1 page): rules, where enforced, how to add a model.

## Acceptance criteria
- [ ] One pipeline run shows linked backend + LangFlow spans in Phoenix.
- [ ] `governance_check.py` fails on a planted OpenAI component in a temp flow (test) and passes on the repo.
- [ ] CI runs the check.

## Out of scope
Frontend.
