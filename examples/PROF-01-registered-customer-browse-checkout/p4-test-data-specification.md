# Test Data Specification — Registered Customer: Browse & Checkout (PROF-01)

## 1. Data Parameterization Specification

| Field                               | Type/Format                                                     | Required Volume                                                                                                                  | Source                                                                                                                                                                   | Reuse Policy                             | Uniqueness Constraint                                                                                                       |
| ----------------------------------- | --------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| `customer_id`                       | Existing registered account                                     | ≥ 1,390 unique (matching final VU count from Step 3)                                                                             | Desensitized extract from GreenCart's 210,000-account production table                                                                                                   | Cycle through pool                       | No                                                                                                                          |
| `product_id`                        | Existing catalog SKU                                            | ≥ 3,000 unique                                                                                                                   | `[ASSUMPTION - actual catalog size not stated in source README; 3,000 chosen as a reasonable grocery-catalog sample size, needs confirmation against real catalog size]` | Cycle through pool                       | No                                                                                                                          |
| `search_term`                       | String matching real query patterns                             | ≥ 300 unique                                                                                                                     | `[ASSUMPTION - no search-log source available; would need product analytics platform export per README §6]`                                                              | Cycle through pool                       | No                                                                                                                          |
| `delivery_slot_id`                  | Valid slot ID for customer's region, current day or next 3 days | Regenerated per test run to match the environment's actual available slots (not a static pool — slots are time-bound and expire) | Test-environment slot-generation script, seeded fresh before each run                                                                                                    | Cycle through currently-valid slots only | **Yes — slots become invalid once fully booked or past; script must re-query available slots, not use a stale static list** |
| `saved_payment_method_token`        | Existing saved payment method reference                         | ≥ 1,390 unique                                                                                                                   | Test-environment seeded saved-payment records, one per test `customer_id`                                                                                                | Fixed 1:1 with `customer_id`             | No                                                                                                                          |
| `promo_code` (subset of iterations) | Alphanumeric, system format                                     | ≥ 200 unique, single-use                                                                                                         | Pre-generated test-mode batch, seeded before run                                                                                                                         | **Single-use — reseed between runs**     | **Yes**                                                                                                                     |

## 2. Correlation Mapping Specification

| Value                          | Source Step/Field                                                                     | Destination Step(s)/Field                                                                                                                                | Lifetime                                                                                                                                                       |
| ------------------------------ | ------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `session_token`                | Login (precondition, not part of this profile's timed transactions) → `session.token` | All subsequent requests → `Authorization` header                                                                                                         | Session duration                                                                                                                                               |
| `cart_id`                      | Add to Cart response → `cart.id`                                                      | Select Delivery Slot, Apply Payment/Promo, Confirm Payment → body `cart_id`                                                                              | Until Submit Order completes                                                                                                                                   |
| `delivery_slot_reservation_id` | Select Delivery Slot response → `slot.reservation_id`                                 | Submit Order → body `delivery_slot_reservation_id`                                                                                                       | Until Submit Order completes or reservation expires (test environment's slot-hold timeout — not stated in source doc, `[ASSUMPTION]` flagged for confirmation) |
| `payment_authorization_id`     | Confirm Payment response → `authorization.id`                                         | Submit Order → body `payment_authorization_id`                                                                                                           | Single-use                                                                                                                                                     |
| `order_id`                     | Submit Order response → `order.id`                                                    | Not consumed further within this profile's scope (order-status notification is async/fire-and-forget, out of this profile's transaction list per Step 1) | N/A beyond Submit Order                                                                                                                                        |

## 3. Data Diversity Rules

**`product_id`:** No production access-distribution data available in the source document (unlike the illustrative technique-file examples elsewhere in this skill, which assumed a known top-500-SKU concentration). `[ASSUMPTION]` — pending real data, apply a conservative two-tier weighting (50% of draws from an assumed "popular items" sub-pool of 150 SKUs, 50% uniformly across the remainder) as a placeholder more realistic than pure uniform distribution, but flagged as needing replacement with real product-analytics-derived weighting before this design is finalized.

**`search_term`:** Same gap — no query-frequency data available. Uniform distribution across the 300-term pool used as a placeholder, flagged for the same reason.

**`customer_id`:** No cache-sensitivity concern beyond general session-cache warm effects; pool size (1,390, matching VU count) is adequate since this is a Load/Stress/Scalability test set, not a long-duration Soak test where a larger pool would matter more (Soak was explicitly not selected for this profile in Step 2).

**`delivery_slot_id`:** Not a caching concern — flagged instead under Data Parameterization's uniqueness constraint (slots are inherently time-bound and consumable, not a stable pool to draw from repeatedly).

## 4. Script Blueprint Specification

### Initialization

Authenticate as the assigned `customer_id`; capture `session_token`. Abort iteration (log as setup failure, not a transaction failure) if authentication fails.

### Main Flow

Per Step 3's flagged reconciliation issue, **not every iteration proceeds to Submit Order** — the blueprint models this explicitly:

1. Browse Catalog (weighted `product_id` draw)
2. Search (uniform `search_term` draw)
3. Add to Cart (captures `cart_id`)
4. **Decision point:** `[ASSUMPTION - 10% of iterations, pending real conversion-rate data per Step 3's flag]` proceed to checkout; the remainder end the iteration here, modeling cart abandonment
5. _(For the ~10% proceeding)_ Select Delivery Slot (captures `delivery_slot_reservation_id`)
6. Apply Saved Payment / Promo _(promo applied on the subset using `promo_code`)_
7. Confirm Payment (injects `cart_id`; captures `payment_authorization_id`)
8. Submit Order (injects `delivery_slot_reservation_id`, `payment_authorization_id`; captures `order_id`)

Think time per Step 3's ranges applied between every step, including before the abandonment decision point (i.e., abandoning users still "think" before leaving, matching real behavior more closely than an instant drop-off).

### Verification Points

- Browse/Search/Add to Cart: HTTP 200 + expected content field present.
- Select Delivery Slot: HTTP 200 **and** `slot.reservation_id` present (a slot request can return 200 with a "fully booked" body state — this must be distinguished from a true reservation).
- Confirm Payment: HTTP 200 **and** `authorization.status == "approved"`.
- Submit Order: HTTP 201 **and** `order.id` present.

### Error Handling

Any verification failure aborts the remainder of that iteration only, logged with transaction name and step. **No automatic retry on Confirm Payment or Submit Order** — duplicate-charge/duplicate-order risk, consistent with this profile's elevated risk flags from Step 1.

### Cleanup/Termination

None required beyond the natural end of iteration.

## Step 4 AI Gate Self-Check Summary

Every field classified; uniqueness/diversity concerns flagged including explicit acknowledgment of missing real distribution data; every correlation point has a corresponding verification check; retry-safety explicitly addressed for Confirm Payment and Submit Order; the abandonment-modeling decision from Step 3's reconciliation flag is carried into the Main Flow rather than silently dropped; no tool-specific syntax present. This intermediate spec is kept distinct from the final condensed spec. Proceeding to final Test Case Specification, then Human Review Gate 2.
