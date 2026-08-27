# Load Profile — Guest Customer: Quick Checkout (PROF-02)

## 1. Test Type Selection

| Test Type           | Targets                       | Reason                                                                                                            |
| ------------------- | ----------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| Load Testing        | Full mix (all 5 transactions) | Default — validates NFR compliance at guest-share of evening-peak volume                                          |
| Stress Testing      | Submit Order, Confirm Payment | Shares PROF-01's Order Service rework risk and gateway timeout constraint (same underlying code path, per Step 1) |
| Scalability Testing | Full mix                      | Same §4.1.5 growth NFR applies system-wide, covering Guest sessions as part of the 40% growth target              |

Baseline Testing: **not separately selected for this profile** — PROF-01's Baseline Test on Submit Order already isolates that transaction's behavior on the shared code path; re-running an identical isolated baseline under this profile would be redundant. This is noted explicitly as a deliberate cross-profile reuse decision, not an oversight — the root skill's Design Rules permit designing profiles separately while still recognizing genuine technical overlap through explicit cross-referencing, as long as each profile's own specification remains complete and traceable (this profile's Stress Test still independently validates its own mix and volume against the shared risk, it just doesn't duplicate the isolated single-transaction baseline).

Spike, Soak: considered, not selected — same reasoning as PROF-01 (no spike-pattern NFR signal found; no documented long-running degradation history specific to this profile beyond what PROF-01's Soak consideration already covers, and Soak wasn't selected there either).

## 2. Load Shape Design

**Load Testing:** Ramp-up 6×5min (30min) → steady 60min at growth-adjusted Guest-share target volume → ramp-down 10min. Same overall shape as PROF-01's Load Test, run as this profile's own independent test (not combined with PROF-01's — per root skill Design Rules, profiles are never merged into one design cycle even when they share underlying code).

**Stress Testing (Submit Order, Confirm Payment):** Same shape parameters as PROF-01's Stress Test (+15% every 10min, same stop conditions) — reusing the same _shape design_ is appropriate here since it's testing the same underlying shared code path's behavior under stress, even though it's driven by this profile's own (smaller) Guest-specific volume as the starting baseline, not PROF-01's.

**Scalability Testing:** Same three-plateau structure as PROF-01 (100%/140%/170%), scaled to this profile's own Guest-share baseline.

## 3. NFR to Acceptance Criteria Mapping

| Transaction     | P95                              | P99                                                 | Error Rate | Source                                                                                                                         |
| --------------- | -------------------------------- | --------------------------------------------------- | ---------- | ------------------------------------------------------------------------------------------------------------------------------ |
| Browse Catalog  | ≤600ms                           | ≤1400ms `[ASSUMPTION, same basis as PROF-01]`       | ≤0.5%      | §4.1.1, §4.5.2 — identical targets to PROF-01 since these are shared endpoints with no guest/registered differentiation stated |
| Search          | ≤600ms                           | ≤1400ms `[ASSUMPTION]`                              | ≤0.5%      | §4.1.1, §4.5.2                                                                                                                 |
| Add to Cart     | ≤400ms                           | ≤950ms `[ASSUMPTION]`                               | ≤0.1%      | §4.1.2, §4.5.1                                                                                                                 |
| Confirm Payment | ≤3000ms (shared Checkout budget) | ≤6000ms, must clear 12s gateway timeout with margin | ≤0.1%      | §4.1.3, §4.3.1, §4.5.1 — identical constraint to PROF-01 (same gateway)                                                        |
| Submit Order    | ≤3000ms (shared Checkout budget) | ≤6000ms                                             | ≤0.1%      | §4.1.3, §4.5.1                                                                                                                 |

No NFR in the source document differentiates Guest from Registered response-time targets — the same criteria apply to both profiles' shared endpoints. This is stated explicitly (a confirmed absence of differentiation) rather than left ambiguous.

## Step 2 AI Gate Self-Check Summary

Test type selection explicitly justifies the deliberate non-duplication of PROF-01's Baseline Test rather than silently omitting it without explanation. Load shapes reuse PROF-01's shape _design_ where the underlying risk is genuinely shared, while still running as this profile's own independent test against its own volume — consistent with the root skill's prohibition on merging profiles. Acceptance criteria are confirmed identical to PROF-01's where the source document doesn't differentiate, rather than inventing a guest-specific number. Proceeding to Step 3.
