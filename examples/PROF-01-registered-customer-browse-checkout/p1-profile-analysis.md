# Profile Analysis — Registered Customer: Browse & Checkout (PROF-01)

## Context

- Profile scope (from Phase 1): Browse (FR-1.1–1.3), Cart (FR-2.1–2.2), Checkout with saved payment/promo (FR-2.3, FR-2.5). Excludes guest-specific flows and loyalty/rewards.
- Actor(s) (from Phase 1): Registered Customer.

## 1. Protocol & System Analysis

| Hop                                         | Protocol              | Sync/Async                                                                                                                         |
| ------------------------------------------- | --------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| Web/Mobile App → API Gateway                | HTTPS/REST            | Sync                                                                                                                               |
| API Gateway → Product Catalog Service       | HTTPS/REST (internal) | Sync                                                                                                                               |
| API Gateway → Cart Service                  | HTTPS/REST (internal) | Sync                                                                                                                               |
| API Gateway → Order Service                 | HTTPS/REST (internal) | Sync                                                                                                                               |
| Order Service → Payment Gateway (3rd party) | HTTPS/REST            | Sync, 8s/99% target, **hard 12s timeout** (README §4.3.1)                                                                          |
| Order Service → Order-status event queue    | AMQP                  | Async, fire-and-forget from Order Service's perspective; downstream Notification Service and WMS integration consume independently |
| Order Service → Inventory Service           | HTTPS/REST (internal) | Sync — **pessimistic locking as of the recent rework (README §5.2)**, flagged as elevated risk                                     |

## 2. Transaction Identification

**Parent transaction: "Complete Checkout"** — Start: click "Place Order." End: order confirmation rendered.

| Transaction                 | Boundary                                                       | Sync/Async      | Why nested                                                                                                                                                                           |
| --------------------------- | -------------------------------------------------------------- | --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Browse Catalog              | Click category → product list rendered                         | Sync            | High volume, own NFR (§4.1.1)                                                                                                                                                        |
| Search                      | Submit query → results rendered                                | Sync            | Own NFR (§4.1.1), separate backend path (search index, implied by FR-1.2's autocomplete)                                                                                             |
| Add to Cart                 | Click "Add" → cart badge updated                               | Sync            | Own NFR (§4.1.2)                                                                                                                                                                     |
| Select Delivery Slot        | Choose slot → slot reserved/confirmed                          | Sync            | No explicit NFR (falls under general Checkout P95/P99 per §4.1.3 read as covering the full checkout path) — treated as part of Checkout's own timing budget, not separately targeted |
| Apply Saved Payment / Promo | Select saved method or enter promo → cart total re-rendered    | Sync            | Registered-only differentiator (FR-2.3); own measurable step distinct from Guest's manual entry (PROF-02)                                                                            |
| Confirm Payment             | Submit → Payment Gateway response received                     | Sync, 3rd-party | Isolate external latency; hard 12s timeout is a structural constraint (§4.3.1)                                                                                                       |
| Submit Order                | Payment confirmed → order record persisted, inventory reserved | Sync            | **Elevated risk** — pessimistic inventory locking rework, untested (§5.2); known DB contention candidate                                                                             |

Order-status notification (async, fire-and-forget to SMS/Push) is **not** modeled as its own transaction within this profile — it has no customer-facing wait state and no NFR was found governing it from the customer's perspective; it is noted here so its absence isn't mistaken for an oversight.

## 3. Persona Behavioral Detailing

- **Session shape:** `[ASSUMPTION - not present in README §6's listed analytics scope as summarized; product analytics platform is stated to cover this but exact figures weren't reproduced in the source document provided]`. For this example, estimated at ~9 minutes average session, 5–7 transactions/session, based on typical grocery-delivery session patterns — **flagged for confirmation against the actual product analytics platform data before this design is finalized in a real engagement.**
- **Channel/device mix:** Not explicitly broken down in the source document. `[ASSUMPTION]` estimated 55% mobile app / 45% web, pending actual analytics.
- **Temporal pattern:** README §4.1.4 explicitly states evening peak 6–9pm local time as the concurrency-target window — used directly (sourced, not assumed) as the peak window for this profile.
- **Behavioral variability:** Not quantified in the source document (e.g., cart-to-checkout conversion rate). `[ASSUMPTION]` — flagged as a gap; Task Frequency Mapping below uses order-placement volume as the anchor figure instead of a derived conversion rate, to avoid compounding assumptions.

## 4. Task Frequency Mapping

`[ASSUMPTION - the source README does not include raw APM transaction counts, only the concurrency target (§4.1.4) and the growth projection (§4.1.5). Figures below are back-calculated from those two stated facts plus the registered-account base (§1), not independently measured — this is weaker sourcing than a direct APM export and is labeled as such throughout.]`

Anchor fact used: §4.1.4 states ≥1,800 concurrent active shopping sessions system-wide at evening peak. Registered vs. Guest split is not stated in the source document; `[ASSUMPTION]` — allocated 70% Registered / 30% Guest based on the 210,000-registered-account base (§1) being the dominant channel, consistent with typical grocery-delivery registered-vs-guest ratios. This yields **≈1,260 concurrent Registered sessions at peak.**

| Transaction     | Estimated Frequency (peak hr, derived) | Growth-Adjusted (+40% per §4.1.5) | Basis                                                                                                |
| --------------- | -------------------------------------- | --------------------------------- | ---------------------------------------------------------------------------------------------------- |
| Browse Catalog  | ~28,000/hr                             | ~39,200/hr                        | `[ASSUMPTION]` derived from 1,260 concurrent sessions × estimated iteration rate (see Step 3 pacing) |
| Search          | ~11,000/hr                             | ~15,400/hr                        | `[ASSUMPTION]` same derivation basis                                                                 |
| Add to Cart     | ~4,200/hr                              | ~5,880/hr                         | `[ASSUMPTION]` same derivation basis                                                                 |
| Confirm Payment | ~1,050/hr                              | ~1,470/hr                         | `[ASSUMPTION]` same derivation basis                                                                 |
| Submit Order    | ~1,010/hr                              | ~1,414/hr                         | `[ASSUMPTION]` same derivation basis                                                                 |

**This entire table is a placeholder-quality estimate pending real APM data** — in an actual engagement, this is exactly the kind of figure that should be replaced with a direct APM export before Human Review Gate 2, and is flagged accordingly in Section 9 of this profile's final Test Case Specification.

## 5. Transaction Mix Design

Total (growth-adjusted, estimated): 39,200 + 15,400 + 5,880 + 1,470 + 1,414 = 63,364/hr.

| Transaction     | Frequency  | Mix %    | UBP Flag                                                                                                                                                                                    |
| --------------- | ---------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Browse Catalog  | 39,200     | 61.9%    | —                                                                                                                                                                                           |
| Search          | 15,400     | 24.3%    | —                                                                                                                                                                                           |
| Add to Cart     | 5,880      | 9.3%     | —                                                                                                                                                                                           |
| Confirm Payment | 1,470      | 2.3%     | **High importance despite low share** — hard 12s external timeout, direct revenue impact                                                                                                    |
| Submit Order    | 1,414      | 2.2%     | **High importance, elevated risk** — recent untested pessimistic-locking rework (§5.2); highest-priority transaction in this profile per Phase 1's overall PROF-01 prioritization rationale |
| **Total**       | **63,364** | **100%** |                                                                                                                                                                                             |

## Step 1 AI Gate Self-Check Summary

Every transaction has an explicit boundary; Submit Order and Confirm Payment are flagged for Step 2's attention per the coherence between this table and Phase 1's stated risk rationale; the async notification hop is explicitly addressed as out-of-scope rather than silently omitted. **The frequency data in Section 4 is weakly sourced** (back-calculated from a concurrency target rather than direct APM export) — this is flagged prominently rather than presented with false confidence, consistent with root Core Principle 1. Proceeding to Step 2 with this caveat carried forward explicitly.
