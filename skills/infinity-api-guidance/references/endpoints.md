# Infinity API v2 — Endpoint Quick Reference

Base URL: `https://app.startinfinity.com/api/v2`
Headers on EVERY request: `Authorization: Bearer <token>` + `X-API-Version: <version>` (e.g. `2026-04-20.morava`) + `Content-Type: application/json`.

Full official reference: https://devdocs.startinfinity.com

## Core resources

| Action | Method + Path |
|--------|---------------|
| List boards | `GET /workspaces/{ws}/boards` |
| List folders | `GET /workspaces/{ws}/boards/{board}/folders` |
| List attributes | `GET /workspaces/{ws}/boards/{board}/attributes` |
| List items (filter by folder) | `GET /workspaces/{ws}/boards/{board}/items?folder_id={folder}` |
| Get item with values | `GET /workspaces/{ws}/boards/{board}/items/{item}?expand[]=values` |
| Create item | `POST /workspaces/{ws}/boards/{board}/items` (body: `folder_id`, `values` array) |
| Update item / upsert values | `PUT /workspaces/{ws}/boards/{board}/items/{item}` (body: `{"values": [{"attribute_id": "…", "data": …}]}`) |

**Gotcha:** there is NO `POST /items/{item}/values` route — writing values is ALWAYS the `PUT` on the item shown above (upsert semantics; only the attributes you send are touched).

## Comments

| Action | Method + Path |
|--------|---------------|
| List | `GET /workspaces/{ws}/boards/{board}/items/{item}/comments` (paginated, sortable by `created_at`) |
| Create | `POST /workspaces/{ws}/boards/{board}/items/{item}/comments` (body: `{"text": "…"}`, HTML accepted) |
| Update / Delete | `PUT` / `DELETE /workspaces/{ws}/boards/{board}/items/{item}/comments/{comment}` |

Remember: comment text is HTML in both directions; comments may be purged over time (don't store durable data there).

## Webhooks (hooks)

| Action | Method + Path |
|--------|---------------|
| Create | `POST /workspaces/{ws}/boards/{board}/hooks` (body: `url`, `events` array) |
| List / Delete | `GET` / `DELETE /workspaces/{ws}/boards/{board}/hooks/{hook}` |

Events: `item.created`, `item.updated`, `item.deleted`, `value.created`, `value.updated`, `comment.created`, `comment.updated`. Hooks can also be managed in the UI (Profile → Developer → Webhooks). Hooks are not bound to the token that created them — they survive token rotation.

## Time tracking (API version 2026-04-20+)

| Action | Method + Path |
|--------|---------------|
| Start timer | `POST /workspaces/{ws}/boards/{board}/time-tracking` (body: `{"item_id": "…", "attribute_id": "…"}` — both required) → `201` with entry `{id, started_at, ended_at: null, description, created_by}` |
| Stop / edit entry | `PUT /workspaces/{ws}/boards/{board}/time-tracking/{entryId}` (body: `{"ended_at": "YYYY-MM-DDTHH:MM:SS", "description": "…"}`) — UTC datetime without timezone suffix |
| Delete entry | `DELETE /workspaces/{ws}/boards/{board}/time-tracking/{entryId}` |

**Gotchas:** `GET` on the collection returns `405` — entries CANNOT be listed via the API, so persist entry ids yourself at creation time. The item's value for a time-tracking attribute is always `[]` regardless of entries. No webhook events fire for time entries.

## Common error signatures

| Symptom | Actual cause |
|---------|--------------|
| `Unauthenticated` with a valid token | Missing/typo'd `X-API-Version` header, or wrong host (`api.startinfinity.com` does not exist — use `app.startinfinity.com`) |
| `404` on `POST …/items/{item}/values` | Route does not exist — use `PUT` on the item |
| `405` on `GET …/time-tracking` | Listing not supported — only `POST`/`PUT`/`DELETE` |
| `422` listing `item_id`/`attribute_id` | Time-tracking start requires exactly those two fields |
