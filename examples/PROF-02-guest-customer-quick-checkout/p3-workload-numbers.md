# Workload Numbers — Guest Customer: Quick Checkout (PROF-02)

## 1. Open vs. Closed Model Selection

| Transaction        | Model | Justification                                                                                                                                                                                                  |
| ------------------ | ----- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| All 5 transactions | Open  | Same justification as PROF-01 — public traffic arrives independently of response speed; identical reasoning applies since Guest traffic has the same fundamental arrival characteristics as Registered traffic |

## 2. Think Time & Pacing

| Step                                         | Think Time                                 | Source                                                                                                                                                                                                                                                                                                 |
| -------------------------------------------- | ------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Product list → click product                 | 3–6s (slightly faster than PROF-01's 3–7s) | `[ASSUMPTION]` — guest sessions are more purpose-driven per Step 1's behavioral flag; narrower, faster-skewed range reflects that                                                                                                                                                                      |
| Product detail → Add to Cart                 | 4–8s                                       | `[ASSUMPTION]` — same basis                                                                                                                                                                                                                                                                            |
| Cart → Select Delivery Slot                  | 5–12s                                      | Same as PROF-01 — slot review process is identical regardless of account status                                                                                                                                                                                                                        |
| Select Delivery Slot → Enter Payment Details | 8–20s (**longer than PROF-01's 3–8s**)     | `[ASSUMPTION]` — manual card entry (no saved method) takes meaningfully longer than selecting a saved payment method or promo field; this is the most significant think-time difference from PROF-01 and directly reflects the guest-specific "Enter Payment Details" transaction identified in Step 1 |

**Pacing calculation:** Execution time (P50 proxy, same as PROF-01 since underlying endpoints are shared): Browse ~300ms + Search ~300ms + Add to Cart ~200ms + Confirm Payment ~1500ms + Submit Order ~1500ms ≈ 3.8s. Think time (midpoints): 4.5 + 6 + 8.5 + 14 = 33s. **Total pacing ≈ 36.8s per full iteration** — notably different from PROF-01's 40.3s, driven mainly by the longer payment-entry think time offsetting the slightly faster browsing pace.

## 3. Little's Law Application

**Method A (Step 1's estimated frequency):** λ ≈ Submit Order growth-adjusted = 560/hr = 0.156/sec. Same weak-sourcing caveat as PROF-01.

**Method B (from NFR):** Phase 1's operational-profiles.md allocated 30% of §4.1.4's 1,800-concurrent-session target to Guest = **540 concurrent sessions**, directly NFR-derived (via the same allocation logic used in PROF-01, applied to the complementary share).

**Reconciliation:** Method B used as the design target, same reasoning as PROF-01 — directly NFR-sourced beats back-calculated.

**N = 540 concurrent sessions.**

Cross-check: implied λ = N / W = 540 / 36.8s ≈ 14.7 iterations/sec ≈ 52,800/hr — again far exceeds the Submit-Order-based estimate (560/hr), for the same reason as PROF-01 (not every session reaches Submit Order). This confirms the same abandonment-modeling requirement identified in PROF-01 applies here too, independently derived rather than assumed to be true just because PROF-01 found it.

**Adding 10% safety margin: 540 × 1.10 ≈ 594, rounded to 600 concurrent virtual users (final).**

**Sanity check:** 600 concurrent Guest sessions is plausible relative to PROF-01's 1,390 (a roughly 30/70 split, consistent with the allocation assumption) — internally consistent with Phase 1's stated allocation, though that allocation itself remains an assumption pending real data.

## 4. Throughput Reconciliation

| Transaction  | Target (Step 1, weak source) | Achieved (600 VU @ 36.8s, full-checkout assumption)   | Model | Reconciliation                                                                                                      |
| ------------ | ---------------------------- | ----------------------------------------------------- | ----- | ------------------------------------------------------------------------------------------------------------------- |
| Submit Order | 560/hr                       | ≈58,700/hr (if 100% of iterations reach Submit Order) | Open  | Same large mismatch pattern as PROF-01, for the same reason — requires the same conditional-flow modeling in Step 4 |

**Resolution:** same approach as PROF-01 — Step 4's Script Blueprint must model a conversion rate rather than assuming every iteration checks out. Given Step 1's flag that guest conversion is generally understood to be lower than registered conversion, this profile's assumed conversion rate should be set lower than PROF-01's ~10% placeholder — `[ASSUMPTION - estimated at 6% for this profile, pending real data]`.

## Step 3 AI Gate Self-Check Summary

Every technique applied independently to this profile's own data rather than copying PROF-01's numbers wholesale — think time, Little's Law inputs, and the conversion-rate assumption all show profile-specific reasoning even where the underlying pattern (e.g., the session-vs-completion-rate mismatch) recurs from PROF-01. The model selection, arithmetic, and safety margin are all shown explicitly. Proceeding to Step 4.
