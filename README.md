# ShipMonk

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

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
