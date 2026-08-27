# Load Profile — Registered Customer: Browse & Checkout (PROF-01)

## 1. Test Type Selection

| Test Type           | Targets                       | Reason                                                                                                                                                                                                                                                                 |
| ------------------- | ----------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Load Testing        | Full mix (all 5 transactions) | Default — validates NFR compliance at evening-peak volume (README §4.1.4)                                                                                                                                                                                              |
| Stress Testing      | Submit Order, Confirm Payment | Step 1 UBP flags: recent untested pessimistic-locking rework (§5.2); hard 12s external gateway timeout                                                                                                                                                                 |
| Scalability Testing | Full mix                      | README §4.1.5 explicitly requires validating a projected +40% growth within 6 months of the Q3 expansion — this is a scalability question, not just a load question, since it concerns how the system behaves as load scales, not only whether it meets today's target |
| Baseline Testing    | Submit Order (isolated)       | First isolated measurement of the reworked pessimistic-locking checkout path, before combining into the full-mix Load Test — directly motivated by §5.2's "not load tested since rework" flag                                                                          |
| Spike Testing       | _(considered, not selected)_  | No NFR language or Step 1 signal suggests a spike pattern (no flash-sale/promotional-surge NFR was found for this profile, unlike a typical retail platform) — explicitly not selected rather than defaulted in                                                        |
| Soak Testing        | _(considered, not selected)_  | No documented history of long-running degradation for this specific service; considered but deprioritized relative to the higher-signal Stress/Scalability needs above given finite test-design effort for this example                                                |

## 2. Load Shape Design

**Load Testing:**

- Ramp-up: 6×5min steps (30min total), matching the evening-peak build (6–9pm window per §4.1.4 — a 3-hour window, so a 30-minute ramp is a small, reasonable fraction of it)
- Steady state: 60min at growth-adjusted target volume
- Ramp-down: 10min linear

**Stress Testing (Submit Order, Confirm Payment):**

- Ramp +15% over Load Test target every 10min, hold 5min/step
- Stop condition: error rate >5% OR P95 >3x the Load Test's passing P95 for 2 consecutive steps, OR the moment Confirm Payment's P99 approaches the Payment Gateway's hard 12s timeout with less than 1s margin (a structural stop condition specific to this transaction's external constraint)
- Recovery observation: 15min at zero added load

**Scalability Testing (full mix):**

- Three discrete load plateaus: 100% of current Load Test target, 140% (the §4.1.5 growth target), and 170% (an extrapolated additional headroom check, 30 percentage points beyond the stated target, to observe the degradation curve's shape rather than stopping exactly at the known target)
- Each plateau held 30min after a 10min ramp between plateaus
- Track response-time-vs-load curve shape across the three plateaus, not just pass/fail at each

**Baseline Testing (Submit Order, isolated):**

- Single transaction only, no surrounding mix
- Ramp to a modest fixed concurrency (50 VUs) over 5min, hold 20min
- Purpose: establish this transaction's own latency distribution in isolation, for later comparison against its behavior inside the full-mix Load Test (contention effects become visible as the difference between the two)

## 3. NFR to Acceptance Criteria Mapping

| Transaction     | P95                                                             | P99                                                                                                                     | Error Rate | Source                                                               |
| --------------- | --------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- | ---------- | -------------------------------------------------------------------- |
| Browse Catalog  | ≤600ms                                                          | `[ASSUMPTION - no P99 stated, estimated ≤1400ms as ~2.3x P95, a common ratio absent better data]`                       | ≤0.5%      | §4.1.1, §4.5.2                                                       |
| Search          | ≤600ms                                                          | `[ASSUMPTION - same basis, ≤1400ms]`                                                                                    | ≤0.5%      | §4.1.1, §4.5.2                                                       |
| Add to Cart     | ≤400ms                                                          | `[ASSUMPTION - ≤950ms, same basis]`                                                                                     | ≤0.1%      | §4.1.2, §4.5.1 (Cart is checkout-path per §4.5.1's explicit listing) |
| Confirm Payment | ≤3000ms (shares Checkout's overall §4.1.3 budget)               | ≤6000ms, **with an explicit sub-constraint that it must stay under the Payment Gateway's hard 12s timeout with margin** | ≤0.1%      | §4.1.3, §4.3.1, §4.5.1                                               |
| Submit Order    | ≤3000ms (shares Checkout's overall budget with Confirm Payment) | ≤6000ms                                                                                                                 | ≤0.1%      | §4.1.3, §4.5.1                                                       |

**Note on Confirm Payment / Submit Order sharing the Checkout-level P95/P99 budget:** §4.1.3 states a single P95/P99 for "Checkout completion" as one end-to-end measurement, not per-sub-transaction targets. This mapping applies the same end-to-end figures to both nested transactions as an upper bound (neither may individually consume the entire budget in practice, but no document specifies how the budget splits between them) — **flagged as an open question for Human Review Gate 2**, since a real engagement should get explicit business input on how the 3s/6s budget is expected to divide between the payment-gateway-bound step and the order-persistence step.

## Step 2 AI Gate Self-Check Summary

Load Testing selected by default; Stress, Scalability, and Baseline all trace to specific §5.2/§4.1.5 signals; Spike and Soak were explicitly considered and not selected, with reasons stated rather than silently omitted. Every response-time criterion uses P95 at minimum; P99 gaps are flagged as assumptions rather than invented with false confidence. The Confirm Payment criterion was cross-checked against the Payment Gateway's hard 12s timeout from Step 1. The ambiguity in how the Checkout-level budget divides between Confirm Payment and Submit Order is flagged explicitly rather than resolved by guessing. Proceeding to Step 3.
