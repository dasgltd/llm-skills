---
name: infinity-api-guidance
description: "Best practices, architectural patterns, and known quirks/limitations for integrating with the Infinity (StartInfinity) project management API. Trigger this skill whenever the user asks to create, modify, or debug any integration, automation, webhook handler, script, or MCP server communicating with Infinity — including n8n workflows (community node or HTTP), board webhooks, item/value updates, comments, or time tracking. Trigger on mentions of: 'Infinity API', 'StartInfinity', 'startinfinity.com', 'n8n-nodes-infinity', or Infinity boards, folders, items, attributes, and webhooks."
---

# Infinity (StartInfinity) API Universal Guidance

This skill contains the collective knowledge of integrating the Infinity (StartInfinity) project management API into any application, automation, webhook handler, or MCP server. The API is capable but poorly documented in places, and has several non-obvious quirks (versioned headers, webhook cascades, missing routes) that must be handled in software to avoid silent failures.

Official docs: https://devdocs.startinfinity.com

## 1. Authentication and API Versioning (Crucial)

- **Base URL:** `https://app.startinfinity.com/api/v2` — this is the ONLY host. `api.startinfinity.com` does not exist (it does not even resolve in DNS).
- **Required headers:** `Authorization: Bearer <personal_access_token>` plus `X-API-Version: <version>` (e.g. `2026-04-20.morava`). A missing/wrong version header can surface as a misleading `Unauthenticated` error, and some endpoint groups (e.g. time tracking) only exist from a given version onward — always pin the version header explicitly.
- **Tokens are JWTs** (`sub` = member/profile id, scope `public-api`, ~1-year expiry). Schedule rotation before expiry.
- **Use a dedicated integration user** (e.g. `integrations@yourcompany.com`) instead of the owner's personal token. Two benefits: revocation without touching the owner account, and its `member_id` differs from every human's → free anti-loop filtering in webhook automations (bot-authored comments/changes are distinguishable by author).
- **Token rotation order:** create the new token → update EVERY consumer (all credentials in n8n, env files, etc.) → only then revoke the old one. Board webhooks are not token-bound and survive rotation.
- **OAuth applications** exist (Profile → Developer → OAuth apps: name + redirect URL) for multi-user apps and MCP servers; a plain Bearer token is sufficient for single-workspace automation.

## 2. Data Model and Writing Values

- Hierarchy: **Workspace → Board → Folder → Item**. Attributes are defined at board level and attached per folder; items store a `values` array.
- Read an item with its values: `GET .../items/{item}?expand[]=values`.
- **Response-shape gotcha:** the item's `values` array is returned at the **TOP level of the item object**, not nested under a `data` key. Depending on the endpoint/version the item itself may or may not be wrapped in `{"data": …}`, so parse defensively — read `values` from both `response.values` and `response.data.values`. A handler that assumes only one shape silently reads an empty array (this breaks idempotency/state checks, which then act as if the item has no values).
- **There is NO `POST .../items/{item}/values` route (404).** To create/update values: `PUT /workspaces/{ws}/boards/{board}/items/{item}` with body `{"values": [{"attribute_id": "…", "data": …}]}` — it behaves as an upsert.
- **`data` format by attribute type:** label → array of label UUIDs · date → ISO 8601 string · rating → integer · checkbox → boolean · number → number · text/longtext → string · links → array of `{url, name}` objects. The links attribute accepts custom URI schemes (e.g. `app://…` deeplinks), useful for mobile shortcuts.
- **`custom_id` (auto-increment ID) attributes:** `data` is the raw number; the prefix is display-only (changing the prefix later does not migrate anything). Treat as read-only.
- **`reference` (item-to-item relation) attributes are the big exception — they are NOT writable via the item `PUT`.** Sending them in the `values` array returns `422` ("use the Create reference endpoint"). Links between items go through a dedicated endpoint: `POST /workspaces/{ws}/boards/{board}/references` with body `{"attribute_id": "…", "from_item_id": "…", "to_item_id": "…"}` (201 → returns a `reference` object with its own `id`). For a multi-selection reference attribute, issue one POST per target item. **In current API versions the read/list and delete reference routes return "Removed in selected version"** → you can CREATE references by API but cannot re-read or delete them programmatically; confirmation is visual in the UI, and cleanup must be done in the UI. Persist created reference ids at creation time if you need any handle to them.
- **Respect native board automations.** If an automation fills dates/checkboxes on a status change, write ONLY the trigger value (e.g. the status label) and let the automation do the rest. Duplicating automation logic in code causes double writes and extra webhook cascade events.

## 3. Webhooks (Per Board)

Read `references/webhooks.md` BEFORE writing any handler — it contains the real payload shapes per event and the cascade/dedupe patterns. Summary of the non-negotiables:

- Created in the UI (Profile → Developer → Webhooks) or via `POST /workspaces/{ws}/boards/{board}/hooks`. Events: `item.created/updated/deleted`, `value.created/updated`, `comment.created/updated`.
- **Scope is the whole BOARD, never a folder** → if your automation targets one folder, the handler MUST filter by `folder_id` (a guardrail), or it will react to every team's items.
- **Return HTTP 200 fast** (ack-then-process). Repeated delivery failures cause Infinity to silently disable the hook.
- The UI "test request" sends an **empty body** — never infer payload shape from it; capture real events first.
- One user action can fire N events (native automation cascades, including duplicates of the same transition seconds apart). **Trigger on condition (attribute + value), never on event count, and dedupe by item + transition within a time window.**

## 4. Comments

- Full CRUD per item: `GET/POST .../items/{item}/comments` + `GET/PUT/DELETE .../comments/{comment}`. The `text` field accepts and returns **HTML** (`<p>…</p>`) — strip tags when processing.
- **Retention warning:** comments can be purged (limited retention). NEVER store durable or formal information (decisions, specs, definitions of done) in comments — persist those to long-text attributes. Use comments for lightweight communication only.

## 5. Time Tracking

- The item's value for a time-tracking attribute is ALWAYS `[]` — entries live at dedicated endpoints (available from API version `2026-04-20` onward):
  - `POST /workspaces/{ws}/boards/{board}/time-tracking` with `{"item_id": "…", "attribute_id": "…"}` → starts a timer (201, returns the entry with its `id`).
  - `PUT .../time-tracking/{entryId}` with `{"ended_at": "YYYY-MM-DDTHH:MM:SS", "description": "…"}` → stops/edits the entry (UTC datetime without timezone suffix works).
- **`GET` on the collection returns 405: you CANNOT list entries via the API.** Persist entry ids in your own storage at creation time, or you permanently lose the handle to close them.
- No webhook events exist for time entries.

## 6. n8n Integration Patterns

- The community node `n8n-nodes-infinity` covers workspace/board/folder/attribute/item/subitem only — **NOT comments, hooks, or time tracking**. For those, use HTTP Request nodes against the endpoints above.
- The node's `infinityApi` credential has no generic `authenticate` block → **HTTP Request nodes cannot reuse it as a predefined credential** (returns `Unauthenticated`). Create a separate Header Auth credential (`Authorization` / `Bearer <token>`) for raw HTTP calls, and remember the `X-API-Version` header.
- Recent n8n versions validate node parameters at activation (including activation via the public API): always set `operation` explicitly on Infinity nodes, and note list operations are named `list`, not `getAll`.

## 7. Endpoint Quick Reference

For the exact endpoint catalog (items, values, comments, hooks, time tracking) with methods and body examples, read: `references/endpoints.md`.
