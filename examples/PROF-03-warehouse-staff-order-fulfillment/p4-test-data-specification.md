# Test Data Specification — Warehouse Staff: Order Fulfillment (PROF-03)

## 1. Data Parameterization Specification

| Field                                          | Type/Format                                    | Required Volume                                                                                                                             | Source                                                                                       | Reuse Policy                                                                                                                                                                                                                                                                                                                                       | Uniqueness Constraint |
| ---------------------------------------------- | ---------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------- |
| `picker_device_id`                             | Handheld device/staff identifier               | ≥ 75 unique (matching Picker/Packer headcount from Step 3)                                                                                  | `[ASSUMPTION - synthetic identifiers, since real device IDs weren't in the source document]` | Cycle through pool (each simulated picker keeps one device ID for a full test run, matching real single-shift behavior)                                                                                                                                                                                                                            | No                    |
| `dispatcher_id`                                | Staff identifier                               | ≥ 5 unique                                                                                                                                  | Synthetic                                                                                    | Cycle through pool                                                                                                                                                                                                                                                                                                                                 | No                    |
| `order_id` / `picking_list_id`                 | Existing test order records queued for picking | ≥ 200 pre-seeded orders per test run (sized to sustain the 90-minute steady-state at the estimated cycle rate without exhausting the queue) | Test-environment seeded order backlog, generated to represent realistic order compositions   | Consumed once per picking cycle (an order is picked once, not repeatedly) — **queue must be replenished if the test run exceeds the seeded backlog's capacity, calculated from Step 3's ~298 cycles/hr × 1.5hr ≈ 447 orders needed for the full steady-state window, exceeding the 200 minimum above; revise seed count to ≥450 before execution** |
| `sku_barcode` (within each order's line items) | Existing catalog SKU barcode                   | Drawn from the same ≥3,000-SKU catalog sample shared with PROF-01/02/04                                                                     | Same source                                                                                  | Each order references ~12 SKUs (Step 3's average-items-per-order assumption)                                                                                                                                                                                                                                                                       | No                    |

**Note on the queue-sizing correction above:** this is a concrete example of a downstream step (Data Parameterization) catching an inconsistency in an earlier step's assumption (Step 2's 90-minute steady-state duration combined with Step 3's cycle-rate figure) that neither step alone would have surfaced — exactly the kind of cross-step check this skill's process is designed to produce.

## 2. Correlation Mapping Specification

| Value                                                                                | Source Step/Field                                               | Destination Step(s)/Field                                                                                                                                                                                                     | Lifetime                                                                                                                                                                                                                                                                    |
| ------------------------------------------------------------------------------------ | --------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `picking_list_id`                                                                    | Picking List Retrieval response → `list.id`                     | Every Item Scan within that cycle → body `picking_list_id`                                                                                                                                                                    | Duration of one full picking cycle                                                                                                                                                                                                                                          |
| Scan validation state (implicit — not a single extracted value, but a running count) | Each Item Scan response confirms/rejects against the order line | Pack Confirmation requires all order lines confirmed before it can succeed                                                                                                                                                    | Duration of one full picking cycle                                                                                                                                                                                                                                          |
| `pack_confirmation_id`                                                               | Pack Confirmation response → `pack.id`                          | Dispatch Assignment (performed by a **different actor**, the Dispatcher, reading from a shared packed-orders queue rather than receiving this value via direct request chaining the way PROF-01/02's correlation points work) | From Pack Confirmation until Dispatch Assignment — **this is a queue-mediated handoff between two different simulated actors, not a same-session correlation**, worth flagging as a structurally different correlation pattern than every other profile in this example set |

## 3. Data Diversity Rules

**`sku_barcode`:** Shares the same cache-sensitivity profile as PROF-01/02/04's `product_id`/`sku_id` fields (same underlying Inventory Service, per Step 1's cross-profile dependency note) — apply the same two-tier weighted distribution already established for the shared catalog sample, rather than deriving a separate distribution independently.

**`order_id`/`picking_list_id`:** Not a cache-sensitivity concern — the corrected ≥450 volume (see Data Parameterization above) is driven by queue-exhaustion avoidance, not cache realism.

## 4. Script Blueprint Specification

### Initialization

Each simulated Picker/Packer authenticates with its `picker_device_id`. No customer-style login/session-token flow applies (staff authentication is a separate, simpler mechanism not detailed in the source document — `[ASSUMPTION]` modeled as a device-level credential, flagged for confirmation against the actual Warehouse API's auth mechanism).

### Main Flow

**Picker/Packer track** (75 simulated actors, closed-model, repeating cycles for the test's steady-state duration):

1. Picking List Retrieval (captures `picking_list_id`)
2. Item Scan Confirmation × ~12 (looped, injecting `picking_list_id` and each order line's `sku_barcode`)
3. Pack Confirmation (requires all 12 scans confirmed; captures `pack_confirmation_id`)

**Dispatcher track** (5 simulated actors, independently polling a shared packed-orders queue, not chained to any single Picker/Packer's cycle): 4. Dispatch Assignment/Confirmation (consumes an available `pack_confirmation_id` from the queue)

These two tracks run as **separate concurrent populations within the same test**, not a single linear script — this structural fact must be reflected in however the eventual script is implemented, since forcing Dispatch into the same per-VU loop as Picking/Packing would misrepresent the real, queue-mediated handoff between two different staff roles.

### Verification Points

- Picking List Retrieval: HTTP 200 **and** list contains at least one order line (an empty list is a distinguishable failure mode, not just a fast response).
- Item Scan Confirmation: HTTP 200 **and** `scan.validated == true` — a scan can return 200 with a "mismatch" result (wrong item scanned), which must be distinguished from a true successful validation, directly relevant to BG-3's error-rate goal.
- Pack Confirmation: HTTP 200 **and** all associated order lines show `validated == true` — reject a pack confirmation attempted with incomplete scans as a verification failure, not a pass.
- Dispatch Assignment: HTTP 200 **and** `dispatch.status == "confirmed"`.

### Error Handling

A rejected Item Scan (wrong item) should **not** abort the picking cycle — a real picker would re-scan the correct item, so the script should model a bounded retry (`[ASSUMPTION]` up to 2 re-scan attempts) before treating it as a genuine failure requiring escalation. This is a deliberate exception to the "no automatic retry" pattern used elsewhere in this example set, justified because a mis-scan is a normal, expected, recoverable real-world event for this specific transaction — not a side-effect-bearing action like a payment, and explicitly not analogous to the no-retry reasoning applied to Confirm Payment/Submit Order in PROF-01/02.

### Cleanup/Termination

Release any test orders left in a partially-picked state at test end, so they don't pollute the next test run's queue.

## Step 4 AI Gate Self-Check Summary

The queue-sizing inconsistency caught between Step 2 and Step 3 is resolved explicitly with a corrected figure, not silently left inconsistent. The two-track (Picker/Packer vs. Dispatcher) structure is called out explicitly as a scripting-relevant structural fact, not flattened into a single linear flow that would misrepresent the real handoff. The bounded-retry exception for Item Scan is explicitly justified against this skill's usual no-retry default, rather than either blindly applying the default or blindly allowing retries without reasoning. Proceeding to final Test Case Specification, then Human Review Gate 2.
