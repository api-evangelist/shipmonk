---
name: Send inbound stock to ShipMonk (receiving)
description: Create and update an ASN-keyed receiving with typed carton/pallet LPN identifiers, track it to receipt, and simulate check-in discrepancies in sandbox.
api: openapi/shipmonk-receivings-openapi.yml
operations: [post-v1-integrations-receiving, get-v1-integrations-receiving, get-v1-integrations-receivings-list, get-v1-integrations-receipts-list, post-v1-integrations-sandbox-complete-receiving]
generated: '2026-08-02'
method: generated
source: openapi/_original/shipmonk-openapi.json + https://apidocs.shipmonk.com/reference/post-v1-integrations-receiving
---

# Send inbound stock to ShipMonk (receiving)

A receiving is an inbound shipment into a ShipMonk warehouse. This is the ERP-push direction.
Authenticate with `Api-Key`.

## Create or update — `post-v1-integrations-receiving`

`POST /v1/integrations/receiving`. The endpoint is an upsert with a two-step key resolution:

- `asn` is **required**. `receiving_key` is optional but is resolved **first**.
- Both supplied: look up by `receiving_key`; if found, update its ASN when different. If not
  found, look up by `asn`; if found, set `receiving_key` on it. If neither matches, create with
  both as unique identifiers.
- Only `asn` supplied: look up by `asn`; update if found, otherwise create.
- Only `receiving_key` supplied: **rejected** — `asn` is required.

### Line keys

Supply `line_key` on each item whenever a receiving can have more than one line for the same
product and lot. `line_key` must be unique per receiving and lets you target a specific line
later (for example to change expected quantity). Lines without a `line_key` are matched by
product + lot; if several keyless lines share a product and lot they cannot be told apart and
are replaced as a group with your request treated as the source of truth. Repeating the same
product and lot in one POST groups them and sums the quantities.

### Carton and pallet identifiers (LPN)

Packaging identifiers use a generic typed LPN vocabulary — `{ type, value }`, where `type` is
currently always `SSCC` (the GS1 Serial Shipping Container Code). Rules tightened in v1.020:

- every carton must carry at least one identifier;
- identifier values must be unique across all cartons **and** pallets of a receiving.

Items reference their packaging through `carton_key`, and loose units reference a pallet
through `loose_units_pallet_key`.

**Breaking change to know about:** the item/line-level `sscc` field was deprecated in v1.018
and **removed in v1.023**. Sending it now fails validation with `Unrecognized key "sscc"`.
Attach the SSCC at packaging level and reference it from the item.

`items[].lot` (or any of its properties) may be null.

## Read it back

- `get-v1-integrations-receiving` (`GET /v1/integrations/receiving`) — supply **either**
  `receivingKey` or `asn`. If you supply both they must both match the same receiving;
  supplying one is strongly advised.
- `get-v1-integrations-receivings-list` (`GET /v1/integrations/receivings-list`) — filter on
  `asn`, `warehouseId`, `createdAt`, `updatedAt`, with `page` / `pageSize`.
- `get-v1-integrations-receipts-list` (`GET /v1/integrations/receipts-list`) — the recorded
  check-in results, filterable on `asn`, `warehouseId`, `completedAt`.

## Simulate check-in in sandbox — `post-v1-integrations-sandbox-complete-receiving`

`POST /v1/integrations/sandbox/complete-receiving`, sandbox only. Identify the target with
`receiving_key` or `asn`, then pick a `completion_mode`:

| mode | effect |
|---|---|
| `fully_received` | accepts all expected units |
| `short_received` | fewer units than expected — may leave the receiving open or raise an exception |
| `excess_received` | more units than expected |
| `partially_received` | partial receipt |

Test all four. The discrepancy outcomes map to the receiving states you will see in
production: partial, short, over, complete.

## Related

- Sandbox: `sandbox/shipmonk-sandbox.yml`
- Events (`receivingStatusChange`): `asyncapi/shipmonk-webhooks-asyncapi.yml`
