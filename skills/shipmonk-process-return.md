---
name: Process a ShipMonk return (RMA)
description: Create an RMA, track it through the return lifecycle, and simulate arrival, grading and disposition in sandbox.
api: openapi/shipmonk-returns-openapi.yml
operations: [post-v1-integrations-returns, get-v1-integrations-returns-list, get-v1-integrations-returns, post-v1-integrations-sandbox-complete-return]
generated: '2026-08-02'
method: generated
source: openapi/_original/shipmonk-openapi.json + https://apidocs.shipmonk.com/reference/post-v1-integrations-returns
---

# Process a ShipMonk return (RMA)

Returns are also the recall mechanism: once an order reaches `pick_in_progress` it can no
longer be cancelled, and issuing a return is the only way to stop it. Authenticate with
`Api-Key`.

## Create or update — `post-v1-integrations-returns`

`POST /v1/integrations/returns`. Specify the warehouse, the return reason, the expected items
and the desired action (for example return to inventory or mark as damaged). ShipMonk assigns
the `rma` identifier and tracks expected versus received quantities, including lot
information.

## Read returns — use the list, not the single-get

- `get-v1-integrations-returns-list` (`GET /v1/integrations/returns-list`) is the endpoint to
  use. Filters: `rma`, `return_status`, `warehouse_id`, `return_reason`, `created_at`,
  `updated_at`, `desired_action`, with `page` / `page_size`.
  **Note the snake_case query parameters** — the returns surface differs from the rest of the
  API, which uses camelCase query parameters.
- `get-v1-integrations-returns` (`GET /v1/integrations/returns`, requires `rma`) is
  **deprecated** — use the list with an `rma` filter instead.

## Return statuses

`status_created` → `in_progress` → `en_route` → `returned`. Each transition fires a
`returnStatusChange` webhook whose payload is one return object in the same shape as an
element of the returns list.

## Simulate the whole lifecycle in sandbox — `post-v1-integrations-sandbox-complete-return`

`POST /v1/integrations/sandbox/complete-return`, sandbox only. It drives arrival, check-in,
receiving, grading and completion in one call.

Completion mode changes how many units are received of the **first** item in the return; all
other items are always fully received:

| mode | effect | constraint |
|---|---|---|
| `fully_received` | receives the complete expected quantity | none |
| `short_received` | receives one less unit than expected | expected quantity > 1 |
| `excess_received` | receives one more unit than expected | none |
| `partially_received` | receives only 1 unit regardless of expected quantity | expected quantity > 1 |

Dispositions: `returned_to_inventory`, `dispose`, `donate`, `return_to_merchant` (no
constraints), and `reworked` (requires the rework feature enabled on the account).

## Related

- Sandbox: `sandbox/shipmonk-sandbox.yml`
- Lifecycle and deprecations: `lifecycle/shipmonk-lifecycle.yml`
