---
name: Ask Endgame a revenue question
description: Create a thread with a natural-language prompt, then poll until the assistant's answer is ready.
api: openapi/end-game-openapi.json
operations: [createThread, getThread]
---

# Ask Endgame a revenue question

Use this skill to ask Endgame a question across your book of business and retrieve the answer.

## Auth
Send `Authorization: Bearer eak_...` (a user-scoped API key) on every request. Writes
require a user-scoped key — org-wide keys and M2M applications get `403 FORBIDDEN` during beta.

## Steps
1. **createThread** — `POST /api/v1/threads` with body `{ "prompt": "<your question>" }`.
   Optionally set `title`, `accountId`, or `secondaryId`. The response returns the new
   `thread.id` plus `responseMessageId` (the in-flight assistant message).
2. **getThread** — `GET /api/v1/threads/{id}` using `thread.id`. Read `status.state`:
   - `in_progress` → wait and poll again (the assistant is still generating).
   - `idle` → the answer is ready; read the latest `assistant` message's `content`.
   - `error` → the assistant message failed; surface `status` / the message `status`.
3. Poll step 2 with backoff until `status.state` is `idle` or `error`.

## Rules
- Assistant generation is asynchronous — never assume the answer is present in the createThread response.
- On `429 RATE_LIMITED`, slow the entire organization's request rate (beta quota 10,000/org/day).
- Every error uses the envelope `{ "error": { "code", "message", "trace_id" } }`; include `trace_id` in support requests.
