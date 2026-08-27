# Profile Analysis — Guest Customer: Quick Checkout (PROF-02)

## Context

- Profile scope (from Phase 1): Browse (FR-1.1–1.3), Cart (FR-2.1–2.2), guest manual payment checkout (FR-2.4, FR-2.5). Excludes any account-bound flow, saved payment methods. Web only — README §2 states guest checkout is not currently supported in the mobile app.
- Actor(s) (from Phase 1): Guest Customer.

## 1. Protocol & System Analysis

Same underlying hops as PROF-01 (§5.1: API Gateway → Product Catalog/Cart/Order Service → Payment Gateway), with one structural difference: **no mobile app client** for this profile (web only, per §2) — so the client-tier hop is HTTPS/REST from web browser only, not web+mobile. This is carried forward into Persona Behavioral Detailing below rather than repeated as a full protocol table, since the internal hops are identical to PROF-01's.

The Order Service's pessimistic-locking rework (§5.2) applies identically here — Guest checkout uses the same Order Service code path as Registered checkout for Submit Order, so this profile inherits the same elevated risk flag.

## 2. Transaction Identification

**Parent transaction: "Guest Checkout"** — Start: click "Proceed to Checkout" as guest. End: order confirmation rendered.

| Transaction                   | Boundary                                                                      | Sync/Async      | Why nested                                                                   | Difference from PROF-01                                                                                                                                                                                                                                                                                             |
| ----------------------------- | ----------------------------------------------------------------------------- | --------------- | ---------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Browse Catalog                | Same as PROF-01                                                               | Sync            | Own NFR                                                                      | Identical transaction, shared endpoint                                                                                                                                                                                                                                                                              |
| Search                        | Same as PROF-01                                                               | Sync            | Own NFR                                                                      | Identical transaction, shared endpoint                                                                                                                                                                                                                                                                              |
| Add to Cart                   | Same as PROF-01                                                               | Sync            | Own NFR                                                                      | Identical transaction, shared endpoint                                                                                                                                                                                                                                                                              |
| Select Delivery Slot          | Same as PROF-01                                                               | Sync            | Part of Checkout budget                                                      | Identical                                                                                                                                                                                                                                                                                                           |
| Enter Payment Details (Guest) | Manual card-detail form submission → details validated client-side and staged | Sync            | **Guest-specific — replaces PROF-01's "Apply Saved Payment/Promo"** (FR-2.4) | Structurally different from PROF-01: no saved-method selection, no promo-code step (README doesn't state whether guests can use promo codes; `[ASSUMPTION - assumed guests cannot, since FR-2.3's promo mention is scoped to "registered customers may apply," implying guests may not; flagged for confirmation]`) |
| Confirm Payment               | Submit → gateway response                                                     | Sync, 3rd-party | Same hard 12s timeout constraint as PROF-01                                  | Identical downstream behavior, different upstream data entry                                                                                                                                                                                                                                                        |
| Submit Order                  | Payment confirmed → order persisted                                           | Sync            | Same elevated risk as PROF-01 (shared Order Service code path)               | Identical                                                                                                                                                                                                                                                                                                           |

## 3. Persona Behavioral Detailing

- **Session shape:** `[ASSUMPTION]` — guest sessions are typically shorter and more purpose-driven than registered sessions (less browsing, more direct-to-purchase intent, consistent with general e-commerce guest-checkout behavior patterns); estimated ~5 minutes average session, 3–4 transactions/session. Needs confirmation against the product analytics platform (README §6).
- **Channel/device mix:** 100% web — this is a **sourced fact**, not an assumption (README §2: "guest checkout is not currently supported in the mobile app"). This is a meaningful, confirmed difference from PROF-01's mixed-channel assumption.
- **Temporal pattern:** `[ASSUMPTION]` — assumed to follow the same evening-peak pattern as PROF-01 (§4.1.4's 1,800-session target is system-wide across both Registered and Guest), absent guest-specific temporal data.
- **Behavioral variability:** `[ASSUMPTION]` — guest checkout is generally understood industry-wide to have a lower cart-to-completion conversion rate than registered checkout (no saved payment method means more checkout friction), but no GreenCart-specific figure is available; flagged as needing confirmation, and treated as distinct from PROF-01's conversion assumption rather than reusing the same figure.

## 4. Task Frequency Mapping

Using the same anchor fact as PROF-01 (§4.1.4's ≥1,800 concurrent sessions) with the complementary 30% Guest allocation established in Phase 1's operational-profiles.md: **≈540 concurrent Guest sessions at peak.**

| Transaction     | Estimated Frequency (peak hr, derived) | Growth-Adjusted (+40% per §4.1.5) | Basis                                                                                |
| --------------- | -------------------------------------- | --------------------------------- | ------------------------------------------------------------------------------------ |
| Browse Catalog  | ~14,000/hr                             | ~19,600/hr                        | `[ASSUMPTION]` derived from 540 concurrent sessions × estimated iteration rate       |
| Search          | ~5,200/hr                              | ~7,280/hr                         | Same basis                                                                           |
| Add to Cart     | ~2,600/hr                              | ~3,640/hr                         | Same basis                                                                           |
| Confirm Payment | ~420/hr                                | ~588/hr                           | Same basis — lower conversion than PROF-01 per the behavioral-variability flag above |
| Submit Order    | ~400/hr                                | ~560/hr                           | Same basis                                                                           |

Same weak-sourcing caveat as PROF-01 applies throughout — this table is back-calculated from a concurrency NFR, not directly measured, and is flagged accordingly.

## 5. Transaction Mix Design

Total (growth-adjusted, estimated): 19,600 + 7,280 + 3,640 + 588 + 560 = 31,668/hr.

| Transaction     | Frequency  | Mix %    | UBP Flag                                                                                                                                                                                                                                                   |
| --------------- | ---------- | -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Browse Catalog  | 19,600     | 61.9%    | —                                                                                                                                                                                                                                                          |
| Search          | 7,280      | 23.0%    | —                                                                                                                                                                                                                                                          |
| Add to Cart     | 3,640      | 11.5%    | —                                                                                                                                                                                                                                                          |
| Confirm Payment | 588        | 1.9%     | Shares PROF-01's hard-timeout risk (same gateway)                                                                                                                                                                                                          |
| Submit Order    | 560        | 1.8%     | Shares PROF-01's Order Service rework risk (same code path) — **but this profile's lower priority ranking (Phase 1) reflects that PROF-01 should be tested first despite this shared risk, since PROF-01 carries higher business share of that same risk** |
| **Total**       | **31,668** | **100%** |                                                                                                                                                                                                                                                            |

Note the mix shape (dominated by Browse/Search, small Checkout share) is nearly identical in proportion to PROF-01's — this is expected, since both profiles fundamentally represent the same "shop then maybe check out" behavior pattern, differing mainly in absolute volume and the specific mechanics of the payment step.

## Step 1 AI Gate Self-Check Summary

Every transaction has a boundary, with explicit call-outs where this profile differs structurally from PROF-01 (guest payment entry replacing saved-payment selection; web-only channel as a sourced fact rather than an assumption) rather than presenting a lazily-duplicated copy of PROF-01's analysis. The promo-code-for-guests ambiguity is flagged rather than assumed silently either way. The shared Order Service risk is explicitly cross-referenced to PROF-01 rather than re-derived independently. Proceeding to Step 2.
