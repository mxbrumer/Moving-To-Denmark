# 04 — Local models setup & evaluation

**Depends on:** 03 · **Branch:** `feat/04-models`

## Goal
Pin the three local models, confirm each one's role with a small repeatable benchmark, and fix the VRAM strategy (D1).

## Known facts (from architecture §1 — don't re-test unless versions change)
- Mimir Q4_K_M: 4096 ctx, 4.2 GB VRAM, JSON mode OK, **no tool calls**.
- qwen2.5:14b: 9.5 GB VRAM, tool calls OK.
- Together > 12 GB → one is evicted.

## Steps
1. **`scripts/pull-models.ps1`**: pulls `qwen2.5:14b`, `hf.co/danish-foundation-models/DFM-Mimir-GGUF:Q4_K_M`, `bge-m3`. Idempotent; prints `ollama list`.
2. **Mimir Modelfile** `infra/ollama/Mimir.Modelfile` → `ollama create mimir-writer`: `FROM` the HF tag, `num_ctx 4096`, conservative `temperature` (~0.5), keep embedded chat template and stop tokens (see `ollama show --modelfile`). Also `qwen-agent` Modelfile with `num_ctx 8192` if the benchmark shows the posting + CV fits; watch VRAM (`ollama ps`). Model names used by the app: `qwen-agent`, `mimir-writer`, `bge-m3` — config values, not hard-coded.
3. **Benchmark harness** `backend/evals/model_bench.py` (run with `uv run`): calls Ollama HTTP API directly. Inputs: 5–10 real postings (DA + EN) the user provides into `DATA_ROOT/profile/eval_postings/` (ask user). Tasks:
   - extraction → `job_brief` JSON schema (title, company, language, contact, must_have[], nice_to_have[], keywords[]); measure schema validity %.
   - cover-letter paragraph in posting language; user rates 1–5 in a CSV.
   - tokens/s and wall time per model.
   Output `DATA_ROOT/evals/<date>.md` (not committed); commit only the harness and a summary table in `docs/model-evaluation.md` (no personal content).
4. **VRAM strategy**: document in `docs/model-evaluation.md` the chosen `keep_alive` values and phase batching (all qwen steps for a batch, then all Mimir steps). If qwen2.5:14b + bge-m3 swap too often, test `qwen2.5:7b`/newer tool-capable models as fallback and record results.
5. **Token budgeting helper** spec (implemented in plan 09): count tokens with Mimir's tokenizer (`tokenizer.json` from HF repo via `tokenizers` lib) to enforce the budgets in architecture §5.

## Acceptance criteria
- [ ] `ollama list` shows `qwen-agent`, `mimir-writer`, `bge-m3`.
- [ ] Benchmark runs end-to-end; summary committed in `docs/model-evaluation.md`.
- [ ] Decision recorded: final model per role, `num_ctx`, `keep_alive`. If the benchmark contradicts D1, **stop and ask the user**.

## Out of scope
LangFlow flows (08/09).
