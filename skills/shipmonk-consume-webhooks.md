---
name: Consume ShipMonk webhooks
description: Subscribe to ShipMonk's four event types, authenticate and verify deliveries, and handle its retry and split-shipment edge cases instead of polling the throttled orders list.
api: asyncapi/shipmonk-webhooks-asyncapi.yml
operations: [webhook-order-shipment-notification, webhook-order-status-change, webhook-return-status-change, webhook-receiving-status-change]
generated: '2026-08-02'
method: generated
source: openapi/_original/shipmonk-openapi.json (webhooks block) + https://apidocs.shipmonk.com/reference/webhooks
---

# Consume ShipMonk webhooks

Webhooks are the right way to track ShipMonk state. The alternative — polling
`get-v1-integrations-orders-list` — is capped at 1,000 requests per 30 minutes.

## Subscribe

Configure a "Webhooks" integration in the ShipMonk app under
Account Settings > Integrations > New Integration. A subscription delivers events for **every
store on the account by default** and can optionally be restricted to specific stores
(documented in v1.022). The shipment-notification endpoint URL is set per store in the store
detail of ShipMonk OMS.

## Authenticate and verify

- **HTTP Basic authentication (username + password) is the only supported scheme.** Your
  endpoint must accept it.
- Verify integrity with the `X-Sm-Signature` header — an HMAC-SHA512 over the payload.
- Return `200` (2xx). Any other status is treated as a failure.

## Retry semantics

Failed deliveries are retried with exponential backoff: up to 8 attempts starting at 30s and
doubling — 30s, 1m, 2m, 4m, 8m, 16m, 32m, 60m, capped at 60 minutes — spanning roughly two
hours (changelog v1.021). Older documentation described a fixed 5-minute cadence retried
across three days; the backoff schedule is the current published policy. Make your handler
idempotent and acknowledge fast.

## The four events

| Event | Fires on | Payload |
|---|---|---|
| `orderShipmentNotification` | order packed, batch completed, or wholesale order picked up | tracking + packing information, `packages[]` |
| `orderStatusChange` | any change of an order `processing_status` | the same shape as the Get Order response |
| `returnStatusChange` | any change of a return `status` | one return object, same shape as a returns-list element |
| `receivingStatusChange` | any change of a receiving `status` | one receiving object |

Order processing statuses that fire the change event: `back-order`, `unable_to_submit`,
`queued_to_submit`, `subscription`, `package_forwarding`, `on_hold`, `submitted`,
`pick_in_progress`, `pack_in_progress`, `packed`, `awaiting_pick_up`,
`awaiting_carrier_processing`, `en_route`, `delivered`, `undeliverable`,
`shipped_untrackable`, `cancellation_requested`, `cancelled`.
Return statuses: `status_created`, `in_progress`, `en_route`, `returned`.
Receiving statuses: `awaiting`, `in_progress`, `arrived`, `received`.

## Shipment-notification edge cases

- **Delay.** Notifications are sent after a per-store configurable delay, default 20 minutes.
- **Multiple packages.** An order too large for one packaging ships as several packages; each
  has its own tracking number inside the `packages` array.
- **Split shipments.** If inventory for a SKU is unavailable the order can be split manually in
  the app, and a Shipment Notification is then sent per shipment. But if a split order ships
  across several days, the notification is sent **only once**, when the first package leaves.
- **`fulfilled_quantity` vs `ordered_quantity`.** These differ on partially fulfilled orders.
  For a product of 1x Item A + 1x Item B, a customer ordering 2 whose shipment contains 1 of
  Item A and 2 of Item B yields `ordered_quantity = 2` and `fulfilled_quantity = 1`.
- **Correlation.** Whatever you put in the order's `custom_data` is echoed back here.

## Testing

In sandbox, drive events with the simulation endpoints:
`post-v1-integrations-sandbox-complete-order`, `post-v1-integrations-sandbox-complete-receiving`,
`post-v1-integrations-sandbox-complete-return`. For D2C orders there is a ~1-minute (up to 5)
gap between `awaiting_pick_up` and `en_route`, during which an awaiting-shipment webhook is
delivered to mirror real fulfillment.

## Related

- Event model: `asyncapi/shipmonk-webhooks-asyncapi.yml`
- Conventions: `conventions/shipmonk-conventions.yml`
