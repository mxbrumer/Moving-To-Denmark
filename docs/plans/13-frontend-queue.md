# 13 — Frontend queue tab

**Depends on:** 11 (uses endpoints from 06) · **Branch:** `feat/13-queue`

## Goal
Work through approved applications one at a time and record the outcome.

## UI `/queue`
1. Card for `GET /api/v1/queue/next` showing: company, title, match score, position "3 of 12".
2. **Blob storage location**: Windows host path (from API, plan 06 step 6) with **Copy path** button; list of final files (latest approved PDFs + recruiter email text with **Copy email** button, including subject and recruiter address).
3. **Application website** link (`source_url`, opens in a new tab).
4. Actions → `POST /queue/{id}/action`:
   - **Mark submitted** → `submitted`, sets `submitted_at`.
   - **Reject** → `declined` (user chose not to apply), optional reason.
   - **Skip** → moves to end of queue (no status change).
   Confirmation for submitted/reject; undo toast for 5 s (calls a reverse transition endpoint, or delays the call).
5. Empty queue state with link to Review.

## Acceptance criteria
- [ ] With seed data: skip reorders, submit increments the dashboard "submitted" tile, reject removes from queue.
- [ ] Component tests for all three actions + empty state; backend queue tests from plan 06 still pass.

## Out of scope
Tracking employer responses (possible follow-up: `outcome` field for interview/offer/rejection).
