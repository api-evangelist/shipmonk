---
name: Sync ShipMonk inventory and catalog
description: Create products and reliably iterate ShipMonk inventory levels using the frozen-cursor search endpoints instead of the drifting products list.
api: openapi/shipmonk-products-openapi.yml
operations: [post-v1-integrations-product, post-v1-integrations-products-search, get-v1-integrations-products-search-paginate, get-v1-products]
generated: '2026-08-02'
method: generated
source: openapi/_original/shipmonk-openapi.json + https://apidocs.shipmonk.com/docs/sync-shipmonk-inventory-levels
---

# Sync ShipMonk inventory and catalog

Authenticate with the `Api-Key` header against `https://api.shipmonk.com`.

## Create a product — `post-v1-integrations-product`

`POST /v1/integrations/product`.

- SKUs must be unique within the account; a duplicate is rejected with `400`.
- **Product data is immutable via the API once created** — later changes must be made in the
  ShipMonk web app. Get it right on the first write.
- `required_packaging` declares the *minimum* packaging a product needs. ShipMonk picks the
  smallest packaging that satisfies every item in a shipment, so one box-requiring item forces
  the whole order into a box.
- Fragile products **must** set `required_packaging` to `box`; anything else returns `400`.
- `country_of_origin` takes an ISO 3166-1 alpha-2 code.
- `type` is the structured enum (`pick_and_pack`, `insert`, `packaging`). The legacy
  `product_type` field is deprecated and scheduled for removal on 2026-09-30.

## Iterate the catalog reliably — search, then paginate

Use this pair, not the plain list, whenever you need every product.

1. `post-v1-integrations-products-search` (`POST /v1/integrations/products/search`) — apply
   your filters and sort. The result set is **frozen at request time**, so you can walk the
   whole set without missing products. It returns a cursor **valid for 1 hour**.
2. `get-v1-integrations-products-search-paginate`
   (`GET /v1/integrations/products/search/paginate`) — pass `cursor` (required) and
   `pageSize`. Each response carries `nextCursor`; follow it until exhausted. The response
   body is the same shape as the legacy products list.

A typical inventory sync filters on updated-since (for example yesterday 08:00 to now) and
walks the cursor. Note that `inventory_on_hand_last_updated_at` records only the *latest*
update, so a product last touched today will not appear in a yesterday-only window even if it
also changed yesterday. Overlap your windows.

## The endpoint to avoid for full walks — `get-v1-products`

`GET /v1/products` supports `search`, `page`, `pageSize`, `sortBy`, `sortOrder`, `status`, and
created/updated date ranges — but **results are not consistent across paginated calls**.
Inventory changes during fulfillment reorder products, so items shift pages and can reappear.
Use it for lookups, not for iteration.

## Reading inventory

Product responses carry an aggregated `inventory` object across warehouses. The
classifications are: expected (inbound), quarantined, allocated (reserved to queued or
submitted orders), available (unallocated, includes inbound), on hand (physically in a
warehouse, excludes inbound), and final (available less action-required and backorders — a
negative final inventory is the unit shortfall against open orders).

Allocation is FIFO; when a lot carries an expiration date it becomes FEFO. Lot handling is off
by default and must be enabled on the account.

## Related

- Conventions and pagination detail: `conventions/shipmonk-conventions.yml`
- Data model: `data-model/shipmonk-data-model.yml`
