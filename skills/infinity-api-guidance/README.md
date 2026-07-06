# Infinity API Guidance

This skill acts as the definitive architectural guide for integrating any AI Agent, automation, or custom script with the Infinity (StartInfinity) project management API.

## What it does
It enforces battle-tested development patterns for Infinity, including:
1. **Authentication & Versioning**: Teaches the AI the correct base URL, the mandatory `X-API-Version` header (whose absence surfaces as a misleading `Unauthenticated` error), JWT token rotation order, and the dedicated-integration-user pattern.
2. **Value Writing**: Prevents the classic `404` trap (there is no `POST /values` route — values are upserted via `PUT` on the item) and documents the exact `data` format per attribute type.
3. **Webhook Engineering**: Real captured payload shapes per event type (they differ!), board-wide scope guardrails, automation-cascade dedupe patterns, and the "capture mode" staging technique.
4. **Comments & Time Tracking**: HTML handling, comment retention caveats, and the write-only time-tracking endpoints (entries cannot be listed — ids must be persisted client-side).
5. **n8n Patterns**: What the community node covers vs. when to fall back to HTTP Request nodes, and the credential quirks between them.

## References
It includes detailed `.md` reference files so the AI reads the exact payload shapes and endpoint catalog before generating code:
- `references/webhooks.md` — payload shapes per event, cascade/dedupe handling, operational rules.
- `references/endpoints.md` — endpoint quick reference with methods, bodies, and common error signatures.

## License & Copyright
Copyright (c) 2026 Daniel A. Silva de la Garza / DASG Consulting Ltda. (CNPJ: 61.628.969/0001-04). All rights reserved.

Licensed for personal and educational (non-commercial) use only. See [LICENSE](./LICENSE) for the full terms.
