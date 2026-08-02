# ShipMonk

ShipMonk is a third-party logistics (3PL) and ecommerce fulfillment provider operating
fulfillment centres across the United States, Canada, Mexico, the United Kingdom and Czechia.
Its public API is a single REST surface at `api.shipmonk.com` covering orders, products and
inventory, receivings, returns and warehouses, with four HTTP webhook events and a
contract-only sandbox that ships warehouse simulation endpoints.

- Developer hub: https://apidocs.shipmonk.com/
- OpenAPI 3.1: https://apidocs.shipmonk.com/openapi/public_api.json
- MCP server: https://apidocs.shipmonk.com/mcp
- Status: https://status.shipmonk.com

## What is in this repo

| Directory | What it holds |
|---|---|
| `openapi/` | ShipMonk's own OpenAPI 3.1.0 description, verbatim in `_original/`, plus one document per tag |
| `asyncapi/` | AsyncAPI 3.0 model of the four webhook events, derived from the spec's `webhooks` block |
| `overlays/` | OpenAPI Overlay 1.0.0 files carrying our annotations (rate limits, idempotency keys, environments) |
| `authentication/`, `conventions/`, `errors/`, `rate-limits/`, `lifecycle/`, `changelog/` | Runtime semantics: how you authenticate, how repeat writes are made safe, how lists page, how errors and throttling are signalled, how the contract is versioned |
| `sandbox/` | The sandbox environment and its three warehouse simulation endpoints |
| `data-model/`, `conformance/` | The fulfillment entity graph and cross-cutting standards conformance |
| `security/`, `well-known/` | Domain security probes, security.txt, RFC 9727 api-catalog, disclosure and compliance posture |
| `mcp/`, `skills/`, `agentic-access/`, `llms/` | Agent-facing surface: the live MCP server, its crosswalk to REST, five packaged agent skills, and the docs `llms.txt` |
| `packages/` | Registry probe result — ShipMonk publishes no first-party API client library |

## Notable findings

- ShipMonk self-publishes a real **OpenAPI 3.1.0** description (19 operations, 186 schemas,
  4 declared webhooks) and an **RFC 9727 `/.well-known/api-catalog`** linkset pointing at it.
- A **live MCP server** answers `tools/list` anonymously on the docs host — but with five
  spec-driven meta tools, not one tool per operation.
- **No `Idempotency-Key` header.** Idempotency is a natural-key upsert model
  (`store_id` + `order_key`, `receiving_key`/`asn`, unique `sku`).
- **No 4xx/5xx responses are described anywhere in the spec** — only `200`. Error payload
  shapes are not machine-readable; `errors/` captures only what the docs state in prose.
- **No A2A agent card.** Every `/.well-known/agent-card.json` probe either 404s or returns the
  app's HTML shell; nothing was recorded.
- **No first-party SDK** in any language.
