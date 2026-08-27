# Performance Test Case Specification — Registered Customer: Browse & Checkout (PROF-01)

**Status:** Approved (example)
**Related step documents:** p1-profile-analysis.md · p2-load-profile.md · p3-workload-numbers.md · p4-test-data-specification.md

## 1. Test Case Header

- **Test Case ID:** PROF-01-TC
- **Objective:** Validate that the registered-customer browse-to-checkout journey meets its NFR targets under evening-peak concurrency, tolerates elevated stress on the recently-reworked Submit Order path, and scales acceptably toward the Q3 expansion's +40% growth target.
- **Test Type(s):** Load (full mix) · Stress (Submit Order, Confirm Payment) · Scalability (full mix, 3 plateaus: 100%/140%/170%) · Baseline (Submit Order, isolated)
- **Preconditions:**
  - Test environment seeded with ≥1,390 test customer accounts, each with a saved payment method
  - ≥3,000-SKU representative catalog sample loaded
  - Delivery-slot generation script run fresh before each test run (slots are time-bound, not reusable across runs)
  - Payment gateway sandbox/test-mode active
  - ≥200 single-use test promo codes seeded; reseed between repeat runs

## 2. Scenario Flow Summary

1. Browse Catalog _(think 3–7s)_
2. Search _(think 4–10s)_
3. Add to Cart _(think 5–12s)_ — captures `cart_id`
4. **Decision point:** ~10% of iterations continue to checkout `[ASSUMPTION, pending real conversion data]`; remainder end here (models cart abandonment)
5. Select Delivery Slot _(think 3–8s)_ — captures `delivery_slot_reservation_id`
6. Apply Saved Payment / Promo
7. Confirm Payment _(think 6–15s)_ — captures `payment_authorization_id`
8. Submit Order — captures `order_id`

## 3. Load Profile Summary

| Test Type                              | Shape                                                                                                                                                                                             |
| -------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Load                                   | Ramp-up 6×5min (30min) → steady 60min → ramp-down 10min                                                                                                                                           |
| Stress (Submit Order, Confirm Payment) | +15% over Load target every 10min, hold 5min/step. Stop: error rate >5% or P95 >3x baseline for 2 steps, or Confirm Payment P99 within 1s of the 12s gateway timeout. 15min recovery observation. |
| Scalability (full mix)                 | 3 plateaus: 100% / 140% (§4.1.5 growth target) / 170% (headroom check), 30min each after 10min ramps between them                                                                                 |
| Baseline (Submit Order, isolated)      | Fixed 50 VU, 5min ramp, 20min hold, no surrounding mix                                                                                                                                            |

## 4. Workload Numbers

- **Workload model:** Open (uniform across all transactions in this profile)
- **Virtual users:** 1,390 concurrent (representing session-level concurrency per NFR §4.1.4, not order-completion rate — see item 2 in Section 9)
- **Target throughput:** Submit Order ≈1,414/hr `[weakly-sourced estimate, see Section 9]`

| Step                          | Think Time |
| ----------------------------- | ---------- |
| Product list → click product  | 3–7s       |
| Product detail → Add to Cart  | 4–10s      |
| Cart → Select Delivery Slot   | 5–12s      |
| Delivery Slot → Payment/Promo | 3–8s       |
| Payment form → Confirm        | 6–15s      |

## 5. Test Data Requirements

| Field                        | Required Volume                            | Source                                    | Notes                                                             |
| ---------------------------- | ------------------------------------------ | ----------------------------------------- | ----------------------------------------------------------------- |
| `customer_id`                | 1,390 unique                               | Desensitized production extract           | —                                                                 |
| `product_id`                 | 3,000 unique, weighted 50/50 hot/long-tail | `[ASSUMPTION - placeholder distribution]` | Replace with real access-distribution data before final execution |
| `search_term`                | 300 unique                                 | `[ASSUMPTION - no query log available]`   | —                                                                 |
| `delivery_slot_id`           | Regenerated per run                        | Test-env slot-generation script           | **Time-bound, not a static reusable pool**                        |
| `saved_payment_method_token` | 1,390, 1:1 with `customer_id`              | Test-env seeded                           | —                                                                 |
| `promo_code`                 | 200+ unique                                | Pre-generated batch                       | **Single-use — reseed between runs**                              |

## 6. Correlation Points

| Value                          | Extracted From       | Used In                                              |
| ------------------------------ | -------------------- | ---------------------------------------------------- |
| `session_token`                | Login (precondition) | All requests                                         |
| `cart_id`                      | Add to Cart          | Select Delivery Slot, Payment/Promo, Confirm Payment |
| `delivery_slot_reservation_id` | Select Delivery Slot | Submit Order                                         |
| `payment_authorization_id`     | Confirm Payment      | Submit Order                                         |
| `order_id`                     | Submit Order         | (Not consumed further in this profile)               |

## 7. Acceptance Criteria

| Transaction     | P95                              | P99                                                 | Error Rate |
| --------------- | -------------------------------- | --------------------------------------------------- | ---------- |
| Browse Catalog  | ≤600ms                           | ≤1400ms `[ASSUMPTION]`                              | ≤0.5%      |
| Search          | ≤600ms                           | ≤1400ms `[ASSUMPTION]`                              | ≤0.5%      |
| Add to Cart     | ≤400ms                           | ≤950ms `[ASSUMPTION]`                               | ≤0.1%      |
| Confirm Payment | ≤3000ms (shared Checkout budget) | ≤6000ms, must clear 12s gateway timeout with margin | ≤0.1%      |
| Submit Order    | ≤3000ms (shared Checkout budget) | ≤6000ms                                             | ≤0.1%      |

## 8. Verification Points & Error Handling

- Browse/Search/Add to Cart: HTTP 200 + expected content field.
- Select Delivery Slot: HTTP 200 **and** `slot.reservation_id` present (distinguish true reservation from a "fully booked" 200 response).
- Confirm Payment: HTTP 200 **and** `authorization.status == "approved"`.
- Submit Order: HTTP 201 **and** `order.id` present.
- **No automatic retry on Confirm Payment or Submit Order** — duplicate-charge/duplicate-order risk. Any verification failure aborts only the current iteration, logged with transaction name + step.

## 9. Open Questions / Assumptions Requiring Confirmation

1. **Task Frequency Mapping (Step 1) is weakly sourced** — back-calculated from the stated concurrency NFR rather than a direct APM export; replace with real transaction-count data before final execution.
2. **Session concurrency (1,390) vs. order-completion rate (~1,414/hr) represent different populations** — the ~10% cart-to-checkout conversion rate used to reconcile them is an unverified assumption; confirm against real conversion data.
3. Persona session shape, channel/device mix, and P99 response-time targets are all estimated, not sourced from the product analytics platform referenced in README §6 — that platform should be queried directly in a real engagement.
4. It is unconfirmed how the Checkout-level P95/P99 budget (§4.1.3) is intended to split between Confirm Payment and Submit Order specifically — currently both are held to the full end-to-end figure as an upper bound; needs business input.
5. `product_id` and `search_term` diversity weighting uses a placeholder distribution pending real product-analytics access-pattern data.
6. Delivery-slot-hold expiry timing (affecting `delivery_slot_reservation_id`'s lifetime) is assumed, not confirmed against actual system configuration.
