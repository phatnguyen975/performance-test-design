# Performance Test Case Specification — Guest Customer: Quick Checkout (PROF-02)

**Status:** Approved (example)
**Related step documents:** p1-profile-analysis.md · p2-load-profile.md · p3-workload-numbers.md · p4-test-data-specification.md

## 1. Test Case Header

- **Test Case ID:** PROF-02-TC
- **Objective:** Validate that the guest-checkout journey (web only) meets its NFR targets under guest-share evening-peak concurrency, and tolerates stress on the Submit Order/Confirm Payment path shared with PROF-01's Order Service code.
- **Test Type(s):** Load (full mix) · Stress (Submit Order, Confirm Payment) · Scalability (full mix, 3 plateaus)
- **Preconditions:**
  - Web-only test client (no mobile app — confirmed scope limitation)
  - ≥600 synthetic test-mode payment card sets
  - Shared delivery-slot pool with PROF-01 — coordinate test scheduling if both profiles run concurrently
  - ≥3,000-SKU catalog sample (shared with PROF-01)

## 2. Scenario Flow Summary

1. Browse Catalog _(think 3–6s)_
2. Search _(think 4–8s)_
3. Add to Cart _(think 5–12s)_ — captures `cart_id`
4. **Decision point:** ~6% of iterations continue to checkout `[ASSUMPTION]`; remainder end here
5. Select Delivery Slot — captures `delivery_slot_reservation_id`
6. Enter Payment Details (manual, think 8–20s — no saved-method option)
7. Confirm Payment — captures `payment_authorization_id`
8. Submit Order — captures `order_id`

## 3. Load Profile Summary

| Test Type                              | Shape                                                                                |
| -------------------------------------- | ------------------------------------------------------------------------------------ |
| Load                                   | Ramp-up 6×5min (30min) → steady 60min at Guest-share target → ramp-down 10min        |
| Stress (Submit Order, Confirm Payment) | +15% every 10min, hold 5min/step. Same stop conditions as PROF-01 (shared code path) |
| Scalability                            | 3 plateaus (100%/140%/170%), scaled to this profile's own baseline                   |

Baseline Testing deliberately not repeated here — already covered once for the shared Order Service code path via PROF-01's Baseline Test.

## 4. Workload Numbers

- **Workload model:** Open (all transactions)
- **Virtual users:** 600 concurrent (30% share of §4.1.4's system-wide target)
- **Target throughput:** Submit Order ≈560/hr `[weakly-sourced estimate]`

| Step                         | Think Time |
| ---------------------------- | ---------- |
| Product list → click product | 3–6s       |
| Product detail → Add to Cart | 4–8s       |
| Cart → Select Delivery Slot  | 5–12s      |
| Slot → Enter Payment Details | 8–20s      |

## 5. Test Data Requirements

| Field                        | Required Volume                               | Source                | Notes                                           |
| ---------------------------- | --------------------------------------------- | --------------------- | ----------------------------------------------- |
| `guest_session_id`           | Generated fresh per iteration                 | Script-generated      | Never reused (no persistent guest identity)     |
| `product_id`                 | Shared 3,000-SKU sample with PROF-01          | Same source           | —                                               |
| `search_term`                | Shared 300-term pool with PROF-01             | Same source           | —                                               |
| `delivery_slot_id`           | Regenerated per run, shared pool with PROF-01 | Test-env slot script  | **Coordinate with PROF-01 if run concurrently** |
| `guest_payment_card_details` | 600 unique                                    | Gateway test-card set | —                                               |

## 6. Correlation Points

| Value                          | Extracted From       | Used In                               |
| ------------------------------ | -------------------- | ------------------------------------- |
| `cart_id`                      | Add to Cart          | Select Delivery Slot, Confirm Payment |
| `delivery_slot_reservation_id` | Select Delivery Slot | Submit Order                          |
| `payment_authorization_id`     | Confirm Payment      | Submit Order                          |
| `order_id`                     | Submit Order         | (Not consumed further)                |

No login/session-token correlation (no account).

## 7. Acceptance Criteria

Identical to PROF-01's, since the source NFR document does not differentiate Guest from Registered on these shared endpoints:

| Transaction     | P95                              | P99                                     | Error Rate |
| --------------- | -------------------------------- | --------------------------------------- | ---------- |
| Browse Catalog  | ≤600ms                           | ≤1400ms `[ASSUMPTION]`                  | ≤0.5%      |
| Search          | ≤600ms                           | ≤1400ms `[ASSUMPTION]`                  | ≤0.5%      |
| Add to Cart     | ≤400ms                           | ≤950ms `[ASSUMPTION]`                   | ≤0.1%      |
| Confirm Payment | ≤3000ms (shared Checkout budget) | ≤6000ms, must clear 12s gateway timeout | ≤0.1%      |
| Submit Order    | ≤3000ms (shared Checkout budget) | ≤6000ms                                 | ≤0.1%      |

## 8. Verification Points & Error Handling

Same pattern as PROF-01: Confirm Payment requires `authorization.status == "approved"`, not just HTTP 200. Submit Order requires `order.id` present. **No automatic retry on Confirm Payment or Submit Order** — same duplicate-charge/duplicate-order risk (shared Order Service code path).

## 9. Open Questions / Assumptions Requiring Confirmation

1. Same weak-sourcing caveat on Task Frequency Mapping as PROF-01 (back-calculated, not APM-measured).
2. Guest conversion rate (~6%) is an unverified, profile-specific estimate — lower than PROF-01's ~10% based only on general industry pattern, not GreenCart data.
3. Whether guests can use promo codes at all is unconfirmed (README's FR-2.3 scopes promo mention to registered customers, implying no, but this is inferred, not stated).
4. Delivery-slot pool contention between this profile and PROF-01 during concurrent test execution needs explicit test-scheduling coordination.
5. Persona session shape and temporal pattern are estimated, pending real guest-specific analytics.
