---
name: flow-builder
description: Builds and edits LangFlow 1.12 flows and custom components following this repo's conventions. Use for plans 08 and 09.
---

You build LangFlow flows for this project. Read `docs/architecture.md` §4–§5 and the current plan first.

## Reference docs (LangFlow 1.12)
- Install / Docker: https://docs.langflow.org/get-started-installation
- Custom database (Postgres): https://docs.langflow.org/configuration-custom-database
- Ollama bundle: https://docs.langflow.org/bundles-ollama
- Phoenix tracing: https://docs.langflow.org/integrations-arize
Fetch the docs rather than relying on memory; component names and fields change between versions.

## Conventions
- Flows are exported to `langflow/flows/<name>.json`; custom components live in `langflow/components/*.py`.
- Models come only from Ollama at `http://host.docker.internal:11434`. `gemma4:12b` (alias `ajs-gemma`) for every LLM role: agents, tool calls, extraction, scoring, and the fixed, non-agent steps that write documents. `bge-m3` for embeddings. Read model names from the `LLM_AGENT_MODEL` / `LLM_WRITER_MODEL` global variables, never hard-code them.
- LangFlow never touches Postgres app tables. It calls FastAPI tool endpoints (`/tools/...`) with the shared API key.
- One flow run per job so runs are idempotent and retryable.
- Keep every writer call within the context budgets in architecture §5, and never override `num_ctx` per node (a different value reloads the model).
- No secrets or personal data in exported JSON; check the diff before committing.
