---
name: Create and fulfill a ShipMonk order
description: Push an outbound customer order into ShipMonk, release it for fulfillment, track it to shipment, and simulate the warehouse leg in sandbox.
api: openapi/shipmonk-orders-openapi.yml
operations: [post-v1-integrations-order, post-v1-integrations-submit-order, get-v1-integrations-orders, get-v1-integrations-orders-list, post-v1-integrations-sandbox-complete-order]
generated: '2026-08-02'
method: generated
source: openapi/_original/shipmonk-openapi.json + https://apidocs.shipmonk.com/docs/creating-an-order
---

# Create and fulfill a ShipMonk order

Base URL `https://api.shipmonk.com` (sandbox `https://sandbox.shipmonk.dev`).
Authenticate every call with the `Api-Key` header. There are no scopes — the key acts on any
store on the account.

## Before you start

- Decide the `order_key` once and persist it. It is the only duplicate-suppression mechanism
  ShipMonk offers: re-posting with the same `store_id` + `order_key` **updates** the order,
  while re-posting the same order with a *different* `order_key` **duplicates** it.
- Do not send `warehouse` unless you have a reason to. ShipMonk routes to the optimal
  warehouse on inventory availability and shipping cost when the field is omitted.
- For international destinations that require it (Mexico, Brazil, South Korea and others),
  collect `recipient_tax_id` at checkout. Omitting it does not fail the request — the order
  silently lands in the "Recipient Tax ID Required" Action Required state and may be rejected
  by the carrier.

## Steps

1. **Create or update the order** — `post-v1-integrations-order`
   (`POST /v1/integrations/order`).
   Set `order_status` to one of `unfulfilled`, `cancelled`, `fulfilled`, `onHold`.
   Use `custom_data` for any correlation payload you want echoed back on the shipment
   notification webhook. Leave `submit_at` unset to let store settings decide when the order
   is released.

2. **Release it for fulfillment** — `post-v1-integrations-submit-order`
   (`POST /v1/integrations/submit-order`).
   Pass a `submit_at` in the past or now to release immediately; pass a future `submit_at` to
   reschedule — inventory is reserved immediately and the order enters the queue at the
   scheduled time. After submission, edit restrictions apply: during picking, items cannot be
   changed; once packed, nothing on the order can be changed.

3. **Read it back** — `get-v1-integrations-orders`
   (`GET /v1/integrations/orders`) with `orderKey` (add `storeId` when order keys are not
   unique across stores), or `orderNumber`.

4. **Poll the list sparingly** — `get-v1-integrations-orders-list`
   (`GET /v1/integrations/orders-list`) supports `page`, `pageSize`, `sortOrder`, plus
   `orderStatus`, `orderType`, `updatedAtStart/End`, `shippedAtStart/End` filters.
   **This endpoint is rate-limited to 1,000 requests per 30 minutes** — 100x stricter than
   every other endpoint. Prefer webhooks over polling. Unfiltered it returns up to 1,000,000
   orders; with any filter applied, a maximum of 10,000.

5. **In sandbox only, simulate the warehouse** — `post-v1-integrations-sandbox-complete-order`
   (`POST /v1/integrations/sandbox/complete-order`). The order must already be submitted with
   available inventory, a valid address and a valid shipping method. A shipment notification
   follows after the store's configured delay (default 20 minutes). This endpoint cannot be
   used in production.

## Rules that will bite you

- **Cancellation window** — an order can only be cancelled from `unfulfilled`. Once it reaches
  `pick_in_progress` the only way to recall it is to issue a return.
- **Errors are undocumented** — the OpenAPI declares only `200` for every operation. Treat any
  non-2xx as opaque and log the whole body. See `errors/shipmonk-problem-types.yml` for the
  failure modes ShipMonk documents in prose.
- **Rate limits** — 429 responses carry a `Retry-After` header as an RFC 1123 *date*, not a
  number of seconds. Parse it as a date and back off exponentially on sustained 429s.
- **Naming is mixed** — bodies are snake_case, most query parameters are camelCase.

## Related

- Conventions: `conventions/shipmonk-conventions.yml`
- Events: `asyncapi/shipmonk-webhooks-asyncapi.yml`
- Rate limits: `rate-limits/shipmonk-rate-limits.yml`
