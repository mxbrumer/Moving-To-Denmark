# 07 — Email intake & posting fetcher

**Depends on:** 06 · **Branch:** `feat/07-intake`

## Goal
Poll IMAP for job-alert emails (D3), store them idempotently, get full posting text (D4), and drive the per-job pipeline.

## Prerequisites (ask user)
- IMAP host, port, username, **app password** (put in `infra/.env` themselves; agent never reads `.env`, per settings deny rule).
- Which folder/label to poll and which senders are job alerts (e.g. Jobindex, LinkedIn, The Hub). Recommend the user sets a mail rule moving alerts into folder `JobAlerts`.

## Steps
1. **Settings:** `IMAP_HOST`, `IMAP_PORT=993`, `IMAP_USER`, `IMAP_PASSWORD`, `IMAP_FOLDER`, `IMAP_SENDER_ALLOWLIST`, `POLL_INTERVAL_MIN=30`.
2. **Poller** `worker/imap_poller.py` using `imap-tools`: fetch unseen messages in folder from allowlisted senders; save raw `.eml` to `DATA_ROOT/inbox/<date>/<message-id-hash>.eml`; insert `email_message` (unique Message-ID → skip repeats); mark seen only after DB commit.
3. **Scheduler:** separate `worker` service in compose (same image as backend, command `python -m ajs.worker`) using APScheduler; also exposes nothing. Manual trigger via `POST /api/v1/workflow/run` (plan 06) enqueues the same job. Use a Postgres advisory lock so only one pipeline run happens at a time.
4. **Email → text:** prefer `text/plain`; else HTML → text with `selectolax`/`html2text`; keep links. Pass cleaned text to LangFlow Intake flow (plan 08) which returns a job list.
5. **Posting fetcher** `services/fetch_posting.py`: for each job:
   - if email snippet ≥ ~150 words and has requirements → use it;
   - else `httpx` GET (timeout 15 s, real UA, follow redirects, honor robots.txt) + `trafilatura.extract`;
   - domain denylist (default `linkedin.com`) → skip fetch;
   - failure/short result → status `needs_manual_text` (UI lets user paste; plan 12 adds field).
   Unwrap tracking redirect links (e.g. Jobindex click URLs) before dedup so `source_url` is canonical.
6. **Pipeline orchestrator** `worker/pipeline.py` (the loop from architecture §5):
   - Phase A (qwen): for each new email → Intake flow → for each job: `find` → stop if exists → create row → fetch text → Analysis+Match flow (plan 08).
   - Phase B (Mimir): for each `matched` → Generation flow (plan 09).
   - Each job isolated in try/except → `error` status with message; retries only on transient errors (max 2).
   - Until 08/09 exist, flows are called through an interface that tests can stub.
7. Sample fixtures: 3 anonymized `.eml` files (DA, EN, links-only) in `backend/tests/fixtures/` — synthetic content only.

## Acceptance criteria
- [ ] Tests: idempotent re-poll (same Message-ID ignored), HTML→text, fetch fallback chain, denylist, redirect unwrapping, pipeline with stubbed flows covers dedup-stop and low-match-stop.
- [ ] Against the real mailbox (user present): one poll ingests alerts, no duplicates on second poll.

## Out of scope
Parsing/LLM logic (08).
