# Infinity Webhooks — Real Payload Shapes and Handling Patterns

Everything below was captured from real webhook deliveries (not inferred from docs). Payload shapes differ per event type in important ways — a handler written for one shape silently drops the others.

## Envelope

Every delivery is a `POST` with JSON body:

```json
{ "event": "<type>", "payload": { … } }
```

- `User-Agent: Infinity/<build>` (e.g. `Infinity/42.18.0`).
- The **UI "test request" sends an empty body** (`[]`). It only validates connectivity — never infer payload shape from it.

## Payload shape per event type

### `value.created` / `value.updated`

```json
{
  "id": "<value-uuid>",
  "object": "value",
  "data": …,
  "attribute_id": "<attribute-uuid>",
  "item_id": "<item-uuid>",
  "deleted": false
}
```

- **NO `folder_id`** → if your automation is scoped to a folder, you must `GET` the item to apply the folder guardrail.
- **NO author field** → you cannot tell who (or what) changed the value. Author-based filters are impossible for value events.
- `data` follows the attribute type: label changes arrive as an array of label UUIDs, dates as ISO strings, etc.
- A status/label change arrives as `value.created` the FIRST time the attribute is ever set on that item, and as `value.updated` afterwards → **subscribe to both** or you miss first-time transitions.

### `item.updated`

```json
{
  "id": "<item-uuid>",
  "object": "item",
  "folder_id": "<folder-uuid>",
  "parent_id": null,
  "created_at": "…",
  "sort_order": "…",
  "deleted": false
}
```

- **NO `values`** → an `item.updated` alone tells you nothing about WHAT changed.
- **Item deletion arrives as `item.updated` with `deleted: true`** (soft delete) — there is no reliance on `item.deleted` for this case.

### `comment.created` / `comment.updated`

```json
{
  "id": "<comment-uuid>",
  "object": "comment",
  "parent_id": null,
  "text": "<p>HTML content</p>",
  "item": { "id": "<item-uuid>", "folder_id": "<folder-uuid>", "…": "…" },
  "created_at": "…",
  "created_by": <member_id>,
  "deleted": false
}
```

- The item reference is **NESTED** (`payload.item.id`, `payload.item.folder_id`) — NOT flattened like `item_id` in value events. Handlers that read `payload.item_id` for every event silently ignore all comments.
- `text` is HTML — strip tags (`<p>…</p>` → plain text) before processing.
- `created_by` IS present → comments are the only event where **author-based anti-loop filtering** works without an extra request (e.g. only react when the author is a human member, never when it is your integration user).

## Automation cascades (the #1 source of bugs)

Native board automations (e.g. "when Priority changes → set Status", "when Status = Doing → fill Start date") fire their own `value.*` events. Consequences observed in production:

1. **One user action = N webhook deliveries.** Setting one label can produce 3+ events within seconds (the user's change + each automation's write).
2. **The same logical transition can be delivered more than once** (e.g. two `value.updated` for the same status change ~60s apart).
3. **Automation-driven changes have no meaningful author** — never rely on author filtering for value events (see above: the field does not even exist).

### Required handling pattern

- **Trigger on CONDITION, never on event count:** check `attribute_id` + `data` against the exact transition you care about (e.g. status attribute now contains the "Prioritized" label UUID).
- **Dedupe by `item_id` + transition within a time window** (10 minutes works well). In n8n, `$getWorkflowStaticData('global')` is a good place for the dedupe map; expire keys older than ~24h to avoid unbounded growth. Do NOT dedupe comments — each comment is a legitimate, distinct event.
- **Idempotency check at the consumer:** if the webhook wakes an agent/worker, it should verify the item's CURRENT state before acting (e.g. if status is already "Doing", another invocation of the same transition is in flight → log and exit instead of double-processing).

## Board scope and folder guardrails

- A hook subscribes to the **entire board**. If your board hosts multiple teams/folders, your endpoint receives all of their events.
- For `comment.*` events the folder is available inline (`payload.item.folder_id`); for `value.*` events you must `GET` the item first. Apply the folder check BEFORE any side effect.

## Operational rules

- **Respond 200 immediately** and process asynchronously (in n8n: Webhook node with "Respond Immediately"). Slow or failing endpoints get the hook silently disabled by Infinity after repeated failures.
- If your endpoint is not live yet, do NOT create the hook — deliveries to a dead URL count as failures. A good staging pattern is a "capture mode": endpoint live and returning 200, all side-effect actions disabled, every delivery logged — so real payloads accumulate for analysis before you enable the logic.
- Choosing events in the UI form: for a status-driven flow, `Item updated` + `Value created` + `Value updated` (+ `Comment created`/`Comment updated` if you react to comments) is the useful set. `Item created` is only needed if you react to bare item creation.
