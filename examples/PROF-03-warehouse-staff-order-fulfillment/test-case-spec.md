# Performance Test Case Specification — Warehouse Staff: Order Fulfillment (PROF-03)

**Status:** Approved (example)
**Related step documents:** p1-profile-analysis.md · p2-load-profile.md · p3-workload-numbers.md · p4-test-data-specification.md

## 1. Test Case Header

- **Test Case ID:** PROF-03-TC
- **Objective:** Validate that warehouse picking, packing, and dispatch operations meet their response-time and reliability targets at estimated peak staffing levels, with particular attention to Item Scan Confirmation given its outsized volume and its direct tie to release goal BG-3 (reducing pick errors).
- **Test Type(s):** Load (full mix) · Baseline (Item Scan Confirmation, isolated)
- **Preconditions:**
  - ≥450 pre-seeded test orders in the picking queue (corrected from an initial 200 during Step 4's cross-check — see PROF-03's p4 document for the reconciliation)
  - ≥75 synthetic picker device credentials, ≥5 dispatcher credentials
  - Shared catalog sample (same ≥3,000-SKU set used across all four profiles)

## 2. Scenario Flow Summary

**Two separate concurrent tracks, not one linear flow:**

**Picker/Packer track** (per cycle): Picking List Retrieval → Item Scan Confirmation × ~12 (think 8–20s between scans) → Pack Confirmation.

**Dispatcher track** (independent, queue-mediated): Dispatch Assignment/Confirmation, consuming completed packs from a shared queue — not chained to any single Picker/Packer's session.

## 3. Load Profile Summary

| Test Type                      | Shape                                                                                                       |
| ------------------------------ | ----------------------------------------------------------------------------------------------------------- |
| Load                           | 15min shift-start ramp → 90min steady state → 10min tail (weakly-justified symmetry element, noted as such) |
| Baseline (Item Scan, isolated) | ~20 concurrent simulated pickers, 20min hold                                                                |

Stress, Spike, Scalability, Volume: all explicitly considered and not selected — this profile's load is staffing-bounded, not arrival-driven, so these test types don't have a realistic analog here (see p2-load-profile.md for the full reasoning).

## 4. Workload Numbers

- **Workload model:** Closed — population-limited (fixed warehouse headcount), a structurally different justification than either PROF-01/02's open model or PROF-04's scheduled-closed model
- **Concurrency (N):** 80 total (75 Pickers/Packers + 5 Dispatchers) — derived directly from staffing estimates, not calculated via Little's Law
- **Cycle pacing:** ~15.1 minutes per full picking cycle (List Retrieval → 12 scans → Pack Confirmation)

## 5. Test Data Requirements

| Field                        | Required Volume                                   | Source                  | Notes                                                            |
| ---------------------------- | ------------------------------------------------- | ----------------------- | ---------------------------------------------------------------- |
| `picker_device_id`           | 75 unique                                         | Synthetic               | —                                                                |
| `dispatcher_id`              | 5 unique                                          | Synthetic               | —                                                                |
| `order_id`/`picking_list_id` | **≥450** (corrected from an initial 200 estimate) | Test-env seeded backlog | Queue must sustain the full 90min steady-state at ~298 cycles/hr |
| `sku_barcode`                | Shared 3,000-SKU sample                           | Same as other profiles  | Two-tier weighted distribution                                   |

## 6. Correlation Points

| Value                  | Extracted From         | Used In                                                                                                             |
| ---------------------- | ---------------------- | ------------------------------------------------------------------------------------------------------------------- |
| `picking_list_id`      | Picking List Retrieval | Every Item Scan in that cycle                                                                                       |
| `pack_confirmation_id` | Pack Confirmation      | Dispatch Assignment — **queue-mediated handoff between two different actor tracks, not a same-session correlation** |

## 7. Acceptance Criteria

| Transaction            | P95                                   | P99                    | Error Rate           |
| ---------------------- | ------------------------------------- | ---------------------- | -------------------- |
| Item Scan Confirmation | ≤300ms                                | ≤700ms `[ASSUMPTION]`  | ≤0.2%                |
| Picking List Retrieval | ≤2000ms                               | ≤4000ms `[ASSUMPTION]` | ≤0.5% `[ASSUMPTION]` |
| Pack Confirmation      | ≤500ms `[ASSUMPTION - no source NFR]` | —                      | ≤0.5% `[ASSUMPTION]` |
| Dispatch Assignment    | No NFR found — flagged since Phase 1  | —                      | —                    |

## 8. Verification Points & Error Handling

- Item Scan: HTTP 200 **and** `scan.validated == true` (distinguish a validated scan from a "mismatch" result returned with 200).
- Pack Confirmation: HTTP 200 **and** all order lines show `validated == true` — reject if attempted with incomplete scans.
- Dispatch: HTTP 200 **and** `dispatch.status == "confirmed"`.
- **Exception to this skill's usual no-retry default:** a mis-scanned item allows up to 2 bounded re-scan attempts before being treated as a genuine failure — justified because this is a normal, recoverable real-world event, unlike a payment/order-submission side effect.

## 9. Open Questions / Assumptions Requiring Confirmation

1. **Staffing figures (75 Pickers/Packers, 5 Dispatchers) are entirely estimated** — README §6 confirms real WMS operational logs exist covering this exact data; obtain them before finalizing this design.
2. Pack Confirmation and Dispatch Assignment have **no source-document NFR at all** — their acceptance criteria above are analogical estimates, not sourced targets, and should be a priority follow-up with stakeholders.
3. An internal inconsistency was found between Step 1's Picking-List-Retrieval frequency estimate (~2/picker/hr) and Step 3's cycle-time-derived rate (~4/picker/hr) — both are assumptions, and this discrepancy itself is evidence real data is needed rather than either figure being individually trustworthy.
4. Whether §4.1.5's +40% growth NFR applies to warehouse staffing levels is unconfirmed — this design assumed it does not, based on the NFR's stated scope, but this reading should be confirmed with the business.
5. Staff authentication mechanism (modeled here as a simple device credential) is assumed, not confirmed against the actual Warehouse API's real auth design.
