---
name: List, rename, and delete Endgame threads
description: Page through threads, rename one, and soft-delete another using the public API.
api: openapi/end-game-openapi.json
operations: [listThreads, updateThread, deleteThread]
---

# List, rename, and delete Endgame threads

Use this skill to manage existing Endgame threads.

## Auth
Send `Authorization: Bearer eak_...`. Renaming and deleting require a user-scoped key
and only work on threads the caller created (otherwise `403 FORBIDDEN`).

## Steps
1. **listThreads** — `GET /api/v1/threads?limit=25`. Results are ordered by most recent
   activity. Follow pagination with `cursor` = the previous response's `nextCursor` until
   `nextCursor` is `null`. Optionally filter with `accountId`. If `truncated` is `true`,
   narrow with `accountId` to reach threads the window can't surface.
2. **updateThread** — `PATCH /api/v1/threads/{id}` with body `{ "title": "<new title>" }`.
   Only `title` is writable; unknown fields are rejected with `INVALID_PARAMS`.
3. **deleteThread** — `DELETE /api/v1/threads/{id}`. Soft-deletes the thread; it will no
   longer appear in list or get. A successful response returns `{ "id", "deleted": true }`.

## Rules
- Pagination is cursor-based (opaque `cursor` / `nextCursor`), not offset.
- Org-wide API keys and M2M tokens are read-only during beta — use a user-scoped key to mutate.
- Handle `404 NOT_FOUND` (thread not visible to caller) and the standard `{ error }` envelope.
