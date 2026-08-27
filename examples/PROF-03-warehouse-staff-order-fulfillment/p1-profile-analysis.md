# Profile Analysis — Warehouse Staff: Order Fulfillment (PROF-03)

## Context

- Profile scope (from Phase 1): Picking list retrieval, item scan, pack confirmation (FR-3.1–3.3), dispatch assignment/confirmation (FR-3.4). Merged profile (Picker/Packer + Dispatcher) per Phase 1's coherence-test justification.
- Actor(s) (from Phase 1): Warehouse Picker/Packer, Delivery Dispatcher.

## 1. Protocol & System Analysis

| Hop                                                              | Protocol                                                                          | Sync/Async                                                                                                                                                                                                                                                                                         |
| ---------------------------------------------------------------- | --------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Warehouse handheld scanner → Warehouse API                       | HTTPS/REST (dedicated API, separate from customer-facing Gateway per README §5.1) | Sync                                                                                                                                                                                                                                                                                               |
| Warehouse API → Inventory Service                                | HTTPS/REST (internal)                                                             | Sync — **this is the same Inventory Service used by the customer-facing Order Service (§5.1)**, meaning this profile's Item Scan transaction and PROF-01/02's Submit Order both ultimately touch the same backend service, though through separate API entry points                                |
| Warehouse API → Order Service (for pack/dispatch status updates) | HTTPS/REST (internal)                                                             | Sync                                                                                                                                                                                                                                                                                               |
| Order Service → order-status event queue                         | AMQP                                                                              | Async — same queue PROF-01/02 write to; dispatch confirmation (FR-3.4) is what ultimately triggers the customer-facing "your order has shipped" notification, connecting this profile's output to those profiles' downstream effect, even though it's out of every profile's own transaction scope |

The shared Inventory Service dependency is a notable cross-profile technical fact: this profile's Item Scan volume and PROF-01/02's checkout volume both contend for the same backend resource, even though they're entirely separate Operational Profiles by actor and business context. This doesn't change any single profile's own design, but is worth flagging for whoever eventually schedules test execution across multiple profiles' test cases.

## 2. Transaction Identification

**No single "parent transaction" spanning this entire profile** — unlike the checkout profiles, Picking, Packing, and Dispatch are performed by different individuals (Pickers/Packers vs. Dispatchers) at different points in an order's lifecycle, not by one person in one continuous session. Each is its own top-level transaction:

| Transaction                      | Boundary                                                                  | Sync/Async | Notes                                                                                                                                               |
| -------------------------------- | ------------------------------------------------------------------------- | ---------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| Picking List Retrieval           | Picker requests next assignment → list rendered on handheld device        | Sync       | Own NFR (§4.2.2)                                                                                                                                    |
| Item Scan Confirmation           | Scan a barcode → system validates against order line, confirms or rejects | Sync       | Own NFR (§4.2.1) — **highest-frequency transaction in this profile by a wide margin**, performed dozens of times per order per §4.2.1's own framing |
| Pack Confirmation                | All items picked → packing staff confirm packaging complete               | Sync       | No item-level repetition like Item Scan — one confirmation per order                                                                                |
| Dispatch Assignment/Confirmation | Dispatcher assigns a packed order to a driver → confirms dispatch         | Sync       | Lowest volume (§2: "low headcount, small transaction volume")                                                                                       |

## 3. Persona Behavioral Detailing

- **Session shape:** Structurally different from customer profiles — warehouse staff work continuous shifts, not discrete "sessions" in the e-commerce sense. `[ASSUMPTION]` — modeled instead as a per-shift repetition count: a Picker/Packer performs an estimated 40–70 Item Scans per hour during an active shift (derived from typical grocery-warehouse picking rates), with Picking List Retrieval and Pack Confirmation occurring once per order batch, not once per item.
- **Channel/device mix:** 100% dedicated handheld scanner devices — a **sourced fact** (§5.1's dedicated Warehouse API implies dedicated hardware, consistent with FR-3.2's "scan barcode" requirement).
- **Temporal pattern:** `[ASSUMPTION]` — warehouse activity is likely correlated with order-placement volume but lagged (picking happens after an order is placed, not simultaneously), and bounded by staffing shift hours (§4.4.3's 5am–11pm operating window). No precise correlation-lag figure is available from the source document.
- **Behavioral variability:** README §1 (BG-3) states current picking errors run ~2.1% of orders requiring post-pick correction — this is a **sourced, quantified fact** directly relevant to this profile's error-rate expectations, distinct from an assumption.

## 4. Task Frequency Mapping

This profile's frequency is fundamentally staffing-bounded (closed model, addressed fully in Step 3) rather than open-arrival — but the same per-transaction frequency figures are still needed for Step 2's load shape and Step 4's data sizing.

`[ASSUMPTION - no WMS-side headcount-per-shift or concurrent-active-picker figures are stated in the source document; README §6 confirms WMS operational logs exist covering picking/packing throughput, which would be the correct real source, but weren't reproduced in the provided document]`.

Estimated for a single regional warehouse's peak shift (`[ASSUMPTION]` estimated 25 concurrent active Pickers/Packers per warehouse × 3 regions = 75 total, needs confirmation against real WMS staffing data):

| Transaction                      | Estimated Frequency (peak hr, system-wide across 3 warehouses) | Basis                                                                                                                           |
| -------------------------------- | -------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| Picking List Retrieval           | ~450/hr                                                        | `[ASSUMPTION]` ~2 list retrievals per picker per hour (batches of orders per list) × 75 pickers × 3                             |
| Item Scan Confirmation           | ~4,000/hr                                                      | `[ASSUMPTION]` ~55/hr per picker (midpoint of the 40–70 estimate) × 75 pickers                                                  |
| Pack Confirmation                | ~600/hr                                                        | `[ASSUMPTION]` roughly tracks completed order-batches, derived from list-retrieval-to-completion ratio                          |
| Dispatch Assignment/Confirmation | ~180/hr                                                        | `[ASSUMPTION]` low headcount per §2; estimated small dispatcher team (5 concurrent across all 3 regions) each processing ~12/hr |

No growth-adjustment factor applied here — §4.1.5's stated +40% growth projection is explicitly scoped to "concurrent sessions and order throughput" for customer-facing traffic (§4.1.5's own wording); it isn't stated to apply to warehouse staffing levels, which are a separate operational/hiring decision not addressed in this document. This distinction is stated explicitly rather than silently applying the customer-facing growth figure to a staffing-bounded profile where it doesn't obviously belong.

## 5. Transaction Mix Design

Total (peak hr): 450 + 4,000 + 600 + 180 = 5,230/hr.

| Transaction                      | Frequency | Mix % | UBP Flag                                                                                                                                                                                                                         |
| -------------------------------- | --------- | ----- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Item Scan Confirmation           | 4,000     | 76.5% | **High volume AND high importance** — directly tied to BG-3 (reducing the 2.1% pick-error rate); a slow or unreliable scan action has an outsized operational cost since it blocks a physical worker mid-task (§4.5.3's framing) |
| Pack Confirmation                | 600       | 11.5% | —                                                                                                                                                                                                                                |
| Picking List Retrieval           | 450       | 8.6%  | —                                                                                                                                                                                                                                |
| Dispatch Assignment/Confirmation | 180       | 3.4%  | Lowest volume; also the transaction with no explicit NFR (flagged in Phase 1's gap log)                                                                                                                                          |

Unlike PROF-01/02's mix (dominated by low-stakes browsing with a small high-stakes checkout tail), this profile's mix is dominated by a transaction (Item Scan) that is simultaneously the highest-volume AND one of the highest-importance items — a notably different risk shape worth carrying forward explicitly into Step 2.

## Step 1 AI Gate Self-Check Summary

This profile correctly departs from the single-parent-transaction pattern used in the checkout profiles, since no such single flow exists here; the shared Inventory Service dependency with PROF-01/02 is flagged as a cross-profile technical fact without altering this profile's own design; the growth-factor scoping question (does §4.1.5 apply to staffing?) is resolved explicitly rather than silently defaulting either way; the §1 BG-3 error-rate fact is correctly used as sourced data, not treated as an assumption. Proceeding to Step 2.
