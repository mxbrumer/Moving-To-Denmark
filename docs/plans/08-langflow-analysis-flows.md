# 08 — LangFlow flows: intake → match

**Depends on:** 04, 06 · **Branch:** `feat/08-flows-analysis`

## Goal
Build the Gemma-powered agents in LangFlow 1.12 for brief steps 2–10, versioned as JSON in the repo.

## Read first
LangFlow docs: components, Agent component, custom components, API run endpoint, flow export/import (https://docs.langflow.org/). Verify component names/fields in 1.12 before building. Don't rely on memory.

## Conventions
- Flows: `langflow/flows/<name>.json`, exported with `scripts/export-flows.ps1` (uses LangFlow API to download flows by name; strips API keys/secrets; stable key order so diffs are reviewable). `scripts/import-flows.ps1` does the reverse on a fresh install.
- Custom components: `langflow/components/ajs/*.py` (mounted via `LANGFLOW_COMPONENTS_PATH`). Each backend tool endpoint (plan 06) gets one component that calls it with `httpx` and `BACKEND_API_KEY` from a LangFlow global variable. Mark inputs usable as agent tools (`tool_mode`) where the agent needs them.
- All LLM nodes use the **Ollama** component, base URL from global variable `OLLAMA_BASE_URL`, model from global variable `LLM_AGENT_MODEL` (`ajs-gemma`, plan 04). Set `format=json`/structured output where available; `temperature` ≤ 0.2 for extraction. Set thinking on or off per task as plan 04 recorded, if the 1.12 Ollama component exposes it; otherwise note it under Follow-ups.
- Prompts live in the flow but are also mirrored to `langflow/prompts/*.md` with a version header, so they can be reviewed (plan 10 checks they match).

## Flows
1. **`intake`** (Intake agent) — input: cleaned email text. Output: JSON `{jobs:[{title, company, url, source_job_id?, snippet}]}`. Validate with a JSON schema component/custom validator; one retry with the error message on invalid JSON.
2. **`analyze_and_match`** — input: `{application_id, posting_text}`. A multi-agent chain:
   - **Analyst agent** (Gemma, tool-calling): language (pre-check via custom `detect_language` component using `lingua`), recruiter contact (regex pre-extract component → LLM confirms name/role), `job_brief` JSON (must_have, nice_to_have, keywords, seniority, location, remote, deadline). Calls `PATCH /analysis`. Unsupported language → `POST /status unsupported_language`, end.
   - **Matcher agent**:
     - embedding score: custom component computes bge-m3 embeddings for each CV section (`GET /profile/cv-sections`, cache by content hash in component memory or a backend endpoint) and the brief; score = mean of top-3 cosine sims → 0–100.
     - rubric score: Gemma rates each `must_have`/`nice_to_have` against CV evidence as `met|partial|missing` with cited CV section ids; score = weighted % (must_have ×2).
     - calls `PATCH /match` with both scores and a ≤ 80-word rationale; backend decides matched/low_match (D8).
   - Keep each LLM call under the `num_ctx` chosen in plan 04, and never override `num_ctx` per node (a different value reloads the model). Pass the brief, not the full posting, where possible.
3. **Tracing:** confirm runs show up in Phoenix (env already set in plan 03). Deeper governance in plan 10.
4. **Backend wiring:** put flow ids in settings (or look up by name via LangFlow API at startup); replace the stubs from plan 07 with real calls.
5. **Golden tests:** `langflow/tests/` pytest suite that runs the flows via API against 3 synthetic postings and asserts schema validity + expected language + expected match/low_match. Marked `@pytest.mark.llm` (needs Ollama; excluded from CI, run locally).

## Acceptance criteria
- [ ] `import-flows.ps1` on a fresh LangFlow restores working flows.
- [ ] Synthetic `.eml` fixture → pipeline phase A → rows in correct statuses with `job_brief`, scores and rationale.
- [ ] `pytest -m llm` passes locally; traces visible in Phoenix.
- [ ] No non-Ollama model component in any flow JSON.

## Out of scope
Document generation (09).
