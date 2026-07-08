# MCP Server Best Practices

## Quick Reference

### Server Naming
- **Python**: `{service}_mcp` (e.g., `slack_mcp`)
- **Node/TypeScript**: `{service}-mcp-server` (e.g., `slack-mcp-server`)

### Tool Naming
- Use snake_case with service prefix
- Format: `{service}_{action}_{resource}`
- Example: `slack_send_message`, `github_create_issue`

### Response Formats
- Support both JSON and Markdown formats
- JSON for programmatic processing
- Markdown for human readability

### Pagination
- Always respect `limit` parameter
- Return `has_more`, `next_offset`, `total_count`
- Default to 20-50 items

### Transport
- **Streamable HTTP**: For remote servers, multi-client scenarios
- **stdio**: For local integrations, command-line tools
- Avoid SSE (deprecated in favor of streamable HTTP)

### Remote Deployment & Hubs
- Ship an unauthenticated `/health` probe; keep `/mcp` stateless (`POST /mcp`)
- Reach sibling containers via internal DNS (`<project>_<service>:3000/mcp`), publish no host port
- Run many servers on one host by path: `server.example.com/<service>/mcp` (don't repeat "mcp" in the host)
- **Dual-mount** routes at root `/` AND under `BASE_PATH` — root serves internal DNS + healthcheck, `BASE_PATH` serves the public hub path (match it to the proxy's forwarded path; no strip needed)
- Behind anti-bot proxies (Cloudflare), keep the record **DNS-only (grey cloud)** or local clients hit HTTP 1010
- Registry **package** visibility is separate from repo visibility; verify with an anonymous pull

### Access Control (multi-client servers)
- Gate capability in tiers: `read < write < admin` (delete)
- Read is always on; write/delete require a key
- Resolve level **per request** from a header; filter tools/list by level
- Default to read on missing/unknown key; global `READ_ONLY` kill-switch wins

### Wrapping Real-World APIs
- Absorb vendor quirks in the server (payload casing, generated IDs, typed defaults)
- Reverse-engineer undocumented payloads by create-in-UI → GET → mirror
- Translate human names ↔ internal UUIDs both ways; expose a `get_schema` read tool first
- Gate bulk/scaffolding writes behind a blueprint-for-approval step

---

## Server Naming Conventions

Follow these standardized naming patterns:

**Python**: Use format `{service}_mcp` (lowercase with underscores)
- Examples: `slack_mcp`, `github_mcp`, `jira_mcp`

**Node/TypeScript**: Use format `{service}-mcp-server` (lowercase with hyphens)
- Examples: `slack-mcp-server`, `github-mcp-server`, `jira-mcp-server`

The name should be general, descriptive of the service being integrated, easy to infer from the task description, and without version numbers.

---

## Tool Naming and Design

### Tool Naming

1. **Use snake_case**: `search_users`, `create_project`, `get_channel_info`
2. **Include service prefix**: Anticipate that your MCP server may be used alongside other MCP servers
   - Use `slack_send_message` instead of just `send_message`
   - Use `github_create_issue` instead of just `create_issue`
3. **Be action-oriented**: Start with verbs (get, list, search, create, etc.)
4. **Be specific**: Avoid generic names that could conflict with other servers

### Tool Design

- Tool descriptions must narrowly and unambiguously describe functionality
- Descriptions must precisely match actual functionality
- Provide tool annotations (readOnlyHint, destructiveHint, idempotentHint, openWorldHint)
- Keep tool operations focused and atomic

---

## Response Formats

All tools that return data should support multiple formats:

### JSON Format (`response_format="json"`)
- Machine-readable structured data
- Include all available fields and metadata
- Consistent field names and types
- Use for programmatic processing

### Markdown Format (`response_format="markdown"`, typically default)
- Human-readable formatted text
- Use headers, lists, and formatting for clarity
- Convert timestamps to human-readable format
- Show display names with IDs in parentheses
- Omit verbose metadata

---

## Pagination

For tools that list resources:

- **Always respect the `limit` parameter**
- **Implement pagination**: Use `offset` or cursor-based pagination
- **Return pagination metadata**: Include `has_more`, `next_offset`/`next_cursor`, `total_count`
- **Never load all results into memory**: Especially important for large datasets
- **Default to reasonable limits**: 20-50 items is typical

Example pagination response:
```json
{
  "total": 150,
  "count": 20,
  "offset": 0,
  "items": [...],
  "has_more": true,
  "next_offset": 20
}
```

---

## Transport Options

### Streamable HTTP

**Best for**: Remote servers, web services, multi-client scenarios

**Characteristics**:
- Bidirectional communication over HTTP
- Supports multiple simultaneous clients
- Can be deployed as a web service
- Enables server-to-client notifications

**Use when**:
- Serving multiple clients simultaneously
- Deploying as a cloud service
- Integration with web applications

### stdio

**Best for**: Local integrations, command-line tools

**Characteristics**:
- Standard input/output stream communication
- Simple setup, no network configuration needed
- Runs as a subprocess of the client

**Use when**:
- Building tools for local development environments
- Integrating with desktop applications
- Single-user, single-session scenarios

**Note**: stdio servers should NOT log to stdout (use stderr for logging)

**Deploying a remote HTTP server (field notes):**
- Prefer a **stateless `POST /mcp`** endpoint — simpler to scale than stateful sessions.
- When the consumer (e.g. an n8n "MCP Client Tool" node, another container) lives on the
  **same Docker network**, connect via the platform's **internal DNS**
  (`<project>_<service>:3000/mcp`) and **publish no host port**. This keeps the server
  off the public internet entirely while remaining reachable by sibling containers.
- Bind consideration: a container that must be reached by *other containers* needs to
  listen on `0.0.0.0` inside the container — but only expose it on the **internal network**,
  never map it to a public host port. Reserve `127.0.0.1`-only binding for single-process
  local use. (See DNS Rebinding Protection below for local-listener hardening.)
- Ship an unauthenticated **`/health`** endpoint so orchestrators and you can verify the
  container is up without a full MCP handshake.
- For private container images, configure the registry **pull credential** on the service
  (image can stay private; the platform authenticates the pull) rather than making the
  image public.
- **Registry package visibility ≠ repository visibility.** On registries like GHCR the
  container package has its *own* visibility toggle, separate from the source repo. Making
  the Git repo public does NOT publish the image — a public repo can still ship a private
  package, and the platform's anonymous pull then fails with 403/`unauthorized` (often
  surfacing as a 500 on a deploy-trigger). Either flip the *package's* visibility to public
  or (preferred) set a pull credential. Verify with an anonymous pull, not by eyeballing the
  repo page.

**Path-based routing & multi-server hubs (field notes):**
- To run many MCP servers behind **one hostname**, route by path — a neutral host plus a
  per-service prefix: `server.example.com/<service>/mcp` (e.g. `/n8n/mcp`, `/omie/mcp`).
  Prefer this over one subdomain per server: one DNS record, one TLS cert, one place to add
  the next server. Keep `/mcp` (the protocol's canonical route) only at the *end* of the
  path — don't bake "mcp" into the host too (`mcp.example.com/n8n/mcp` repeats it).
- **Dual-mount the routes** — serve them at root `/` **and** under a configurable
  `BASE_PATH` at the same time (see the Router example in the TypeScript guide). Root keeps
  same-network callers (`<project>_<service>:3000/mcp`) and the container HEALTHCHECK
  working; `BASE_PATH` is what the public hub path maps to. This beats a proxy `stripPrefix`
  because it needs no proxy-specific middleware and the internal path never changes.
- **Match `BASE_PATH` to the proxy's forwarded path.** Many reverse proxies (Traefik/EasyPanel
  domain routes, etc.) forward the request path **as-is**, without stripping the prefix. So a
  route on `server.example.com/<service>/...` arrives at the container as `/<service>/...` —
  set `BASE_PATH=/<service>` so the dual-mount serves it. If your proxy *does* strip, leave
  `BASE_PATH` unset and the root mount handles it.
- **Keep the MCP endpoint un-proxied by anti-bot layers.** Behind Cloudflare-style proxies,
  set the record to **DNS-only (grey cloud)**, not proxied. The proxy's bot protection can
  answer a local MCP client's direct handshake with an interstitial/challenge (Cloudflare
  returns **HTTP 1010**), which clients like Claude Desktop can't solve. DNS-only lets TLS
  terminate at your own reverse proxy (Let's Encrypt) and the handshake through.
- Some platforms **drop the registry pull credential when you rename the image source** on a
  service — re-add it after any image-name change, or the next deploy fails the pull.

### Transport Selection

| Criterion | stdio | Streamable HTTP |
|-----------|-------|-----------------|
| **Deployment** | Local | Remote |
| **Clients** | Single | Multiple |
| **Complexity** | Low | Medium |
| **Real-time** | No | Yes |

---

## Security Best Practices

### Authentication and Authorization

**OAuth 2.1**:
- Use secure OAuth 2.1 with certificates from recognized authorities
- Validate access tokens before processing requests
- Only accept tokens specifically intended for your server

**API Keys**:
- Store API keys in environment variables, never in code
- Validate keys on server startup
- Provide clear error messages when authentication fails

### Capability-Based Access Control (Permission Tiers)

Not every caller of a shared MCP server should be able to mutate — let alone delete —
data. When a single deployed server is reached by many clients (e.g. an n8n agent, a
chatbot, several teammates), gate capability in **tiers** rather than exposing every
tool to everyone.

**Recommended three-tier model** (`read < write < admin`):
- **read** — query tools only (list/get/schema). ALWAYS available to everyone; safe by default.
- **write** — read + create/update tools.
- **admin** — write + destructive/delete tools.

**Key design points learned in the field:**

1. **Resolve the level per request, not per process.** For streamable HTTP, read a
   secret header (e.g. `X-MCP-Key`) on each request and map it to a level. One running
   server then serves read-only, write, and admin callers simultaneously — the caller's
   key decides, not a redeploy.

   ```typescript
   // per-request in the HTTP handler
   const provided = req.headers["x-mcp-key"];
   const level = resolveLevel(Array.isArray(provided) ? provided[0] : provided);
   const server = buildServer(level);   // registers only the tools this level may use
   ```

2. **Filter the tools/list by level — don't just reject at call time.** Register only
   the tools a level is allowed to use, so a read-only caller never even *sees* the
   delete tool. This is cleaner than returning "forbidden" errors and prevents the model
   from wasting turns attempting disallowed actions.

3. **Layer global kill-switches on top of the per-key logic.** Coarse env switches
   (`READ_ONLY=1`, `ALLOW_DELETE=1`) should win over any key, so an operator can hard-lock
   a deployment regardless of which keys leak. Order: global read-only → key match → default read.

   ```typescript
   export function resolveLevel(providedKey?: string): PermLevel {
     if (READ_ONLY) return "read";               // global kill switch always wins
     const key = (providedKey || "").trim();
     if (ADMIN_KEY && key === ADMIN_KEY) return "admin";
     if (WRITE_KEY && key === WRITE_KEY) return "write";
     return "read";                              // no/unknown key -> safe default
   }
   ```

4. **Fail safe, default to read.** No key, wrong key, or missing config → read-only.
   Never default an unauthenticated caller to write.

5. **Ship an introspection tool** (e.g. `{service}_permissions`) that reports the level
   granted to *this* session and which actions are allowed. When a write/delete fails,
   the model calls it and self-corrects ("ask an admin for a write key") instead of
   retrying blindly.

6. **stdio = the operator's own machine.** For local stdio there is no header; grant
   whatever the env allows. The per-key gate is a property of the *remote* transport.

**Why tiers instead of separate servers:** deploying three servers (read/write/admin) triples
ops surface and secrets. One server + per-request key resolution gives the same isolation
with a single image and a single deploy.

**Owner/operator gets admin by default — bake this into the build procedure, not as an
afterthought:**

1. **The instance owner and their agent (the one building/operating the server) are
   provisioned with the `admin` key the moment the server goes live** — on every session
   config that talks to it (e.g. `~/.claude.json` `mcpServers`, `~/.hermes/config.yaml`
   `mcp_servers`), not just one of them. `read`/`write` tiers exist to gate *other* callers
   (teammates, clients, a shared chatbot) — they are never the owner's own default. Treat
   "wire the owner in with admin, everywhere it's registered" as a checklist item of the
   deploy step itself, alongside `/health` and `tools/list` validation — not a follow-up
   the human has to request.
2. **Admin capability ≠ standing authorization for destructive calls.** Even holding the
   admin key, delete/revoke/drop-style tools always require the human's explicit
   go-ahead on that specific action before executing — admin unlocks what the agent
   *can* reach, not a blanket license to act unattended on anything irreversible.
3. **Config drift across sessions is a real, recurring failure mode — audit for it.**
   A server can be correctly configured with the admin key in one place (e.g. the
   gateway's `mcp_servers` yaml) while a *different* session config still points at a
   stale entry — a dead local `stdio` path from before the service moved to a hosted
   HTTP endpoint, an empty key, or a lower-tier key. This reads to the user as "no access
   in that session" even though the server itself is fine. When granting/auditing access,
   check **every** place the server is registered (all session configs, not just the one
   you're currently in) and reconcile them to the same transport + key.
4. **A server not yet registered anywhere in the gateway is not a permissions problem —
   it's an undeployed server.** If a service only exists inside a private network (e.g.
   reachable from one container to another on the same Docker network, but never
   published behind the shared MCP gateway/hostname), "give me admin on it" first
   requires exposing it the same way the other servers are exposed (hosted HTTP + `/mcp`
   path + issued keys) — flag this as a deploy/infra task distinct from a config edit,
   don't try to fake access to it. But verify before concluding this: check whether the
   service is already deployed and publicly routed (curl the expected
   `https://<gateway-host>/<service>/mcp` path — a `401` means the route exists and only
   auth is missing, which is a *config* fix, not a deploy task; a `404`/connection error
   means it genuinely isn't exposed yet). Don't declare "not deployed" from grepping
   session configs alone — the server can be live and routed while simply absent from
   this particular gateway's config file, which reads identically to "never deployed" until
   you actually probe the endpoint.
5. **Two entry points per server, by design: local/internal (owner + automations) and
   public (teammates/clients).** The owner and their agent should reach every self-hosted
   MCP over the *fastest, lowest-friction* path available — same tailnet (Tailscale) or
   the private Docker/VPC network — with the admin key and no OAuth dance, since that
   path is never handed to a third party. The public hostname (`server.<domain>/<service>/mcp`
   + OAuth or issued keys) exists for teammates and clients and is the *only* path they
   ever get. Building only the public path and then using it for the owner's own local
   automations too (e.g. n8n on the same VPS calling out to the internet to reach a
   sibling container) works but adds needless latency/egress and a shared-fate dependency
   on the public proxy being up. When standing up a new MCP, publish the container's port
   on the tailnet (or same private network as the caller) in addition to the public proxy
   route, and default the owner's own session configs/workflows to the internal path —
   reserve the public URL for callers that must cross a trust boundary.

### Input Validation

- Sanitize file paths to prevent directory traversal
- Validate URLs and external identifiers
- Check parameter sizes and ranges
- Prevent command injection in system calls
- Use schema validation (Pydantic/Zod) for all inputs

### Error Handling

- Don't expose internal errors to clients
- Log security-relevant errors server-side
- Provide helpful but not revealing error messages
- Clean up resources after errors

### DNS Rebinding Protection

For streamable HTTP servers running locally:
- Enable DNS rebinding protection
- Validate the `Origin` header on all incoming connections
- Bind to `127.0.0.1` rather than `0.0.0.0`

---

## Tool Annotations

Provide annotations to help clients understand tool behavior:

| Annotation | Type | Default | Description |
|-----------|------|---------|-------------|
| `readOnlyHint` | boolean | false | Tool does not modify its environment |
| `destructiveHint` | boolean | true | Tool may perform destructive updates |
| `idempotentHint` | boolean | false | Repeated calls with same args have no additional effect |
| `openWorldHint` | boolean | true | Tool interacts with external entities |

**Important**: Annotations are hints, not security guarantees. Clients should not make security-critical decisions based solely on annotations.

---

## Error Handling

- Use standard JSON-RPC error codes
- Report tool errors within result objects (not protocol-level errors)
- Provide helpful, specific error messages with suggested next steps
- Don't expose internal implementation details
- Clean up resources properly on errors

Example error handling:
```typescript
try {
  const result = performOperation();
  return { content: [{ type: "text", text: result }] };
} catch (error) {
  return {
    isError: true,
    content: [{
      type: "text",
      text: `Error: ${error.message}. Try using filter='active_only' to reduce results.`
    }]
  };
}
```

---

## Field Notes: Wrapping Real-World APIs

Hard-won lessons from wrapping messy, under-documented, human-facing APIs. The through-line:
**the MCP server should present a clean, human-shaped interface to the model and silently
absorb the API's quirks** — never make the model learn the vendor's internal payload rules.

### Absorb API quirks silently in the server

Real APIs reject payloads for reasons the docs don't explain. The server — not the model —
should handle these:
- **Exact payload key casing.** An API's *error message* may show a human-spaced field name
  ("The formula expression field is required") while the JSON body demands strict camelCase
  (`formulaExpression`). Trust the wire format, not the prose in the error.
- **Synthesize required-but-unstable structures.** Some APIs won't auto-generate stable IDs
  for sub-objects (e.g. option UUIDs for a select/label field) and return `422` without them.
  Generate them server-side.
- **Non-null typed defaults.** Fields typed as number/date often reject `null` defaults that
  a generic "empty settings" object would send. Emit type-correct defaults per field type.

The test: the model sends a friendly request; the server does whatever ugly translation the
vendor needs. If the model has to know a vendor-internal rule, push that knowledge down into
the server.

### Reverse-engineer undocumented payloads by round-trip

When docs are incomplete, don't guess the payload shape:
1. Create the object **manually in the vendor's UI** (a formula, a view, a relation).
2. **GET it back** through the API and inspect the exact stored structure.
3. Mirror that structure in your tool.

This reveals things docs omit — e.g. that a formula stores attribute references as `@<UUID>`
rather than by display name. The round-trip is the source of truth.

### Translate human names ↔ internal IDs

Vendor APIs address everything by opaque UUID; models (and humans) think in names. Make the
server bidirectional:
- **Inputs:** accept human field names, label values, and member names; resolve them to UUIDs
  internally. Accept natural expressions (e.g. `({End} - {Start}) / 30`) and rewrite `{Name}`
  tokens to the vendor's `@<UUID>` form before sending.
- **Outputs:** map UUIDs back to display names so results are readable.

Expose a **`get_schema`-style read tool** that returns the board/resource's field names,
types, and label options up front — the model calls it FIRST to learn the exact strings to
use, instead of guessing UUIDs.

### Know the required/primary field

Most "create record" endpoints have exactly one mandatory attribute (a title/name). Identify
it and default/validate it in the create tool, so item creation never fails on a missing
primary field. Document which field it is in the tool description.

### Gate bulk/scaffolding writes behind a plan

For tools that create many objects at once (scaffold a whole board: folders, attributes,
views, items), have the tool — or the surrounding agent prompt — **return a blueprint for
approval before writing**. Mass-creating 80 wrong items is far more expensive to undo than
one preview step. Pair this with the delete tool being admin-only (see permission tiers).

## Testing Requirements

Comprehensive testing should cover:

- **Functional testing**: Verify correct execution with valid/invalid inputs
- **Integration testing**: Test interaction with external systems
- **Security testing**: Validate auth, input sanitization, rate limiting
- **Performance testing**: Check behavior under load, timeouts
- **Error handling**: Ensure proper error reporting and cleanup

---

## Documentation Requirements

- Provide clear documentation of all tools and capabilities
- Include working examples (at least 3 per major feature)
- Document security considerations
- Specify required permissions and access levels
- Document rate limits and performance characteristics
