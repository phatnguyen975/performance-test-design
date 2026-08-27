# Load Profile — Warehouse Staff: Order Fulfillment (PROF-03)

## 1. Test Type Selection

| Test Type        | Targets                           | Reason                                                                                                                                                                                                                   |
| ---------------- | --------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Load Testing     | Full mix (all 4 transactions)     | Default — validates NFR compliance at estimated peak staffing/shift volume                                                                                                                                               |
| Baseline Testing | Item Scan Confirmation (isolated) | Highest-volume, highest-importance transaction in this profile (Step 1 UBP flag); establishing its isolated latency distribution is valuable given it has no prior isolated measurement mentioned in the source document |

Stress Testing: **considered, not selected.** Unlike PROF-01/02, this profile's load is fundamentally staffing-bounded (a fixed number of Pickers/Packers physically present in a warehouse) — there is no realistic mechanism by which "traffic" exceeds normal levels the way an open-model customer surge can, since the workforce size itself is the ceiling. A Stress Test designed to push load beyond what real staffing could ever generate would test a scenario with no real-world analog. This is a deliberate, reasoned non-selection, not an oversight.

Spike Testing: **considered, not selected**, for a related reason — no plausible trigger for a sudden staffing-driven traffic spike exists in this profile's operational model (staffing changes gradually, shift to shift, not in seconds).

Scalability Testing: **considered, not selected for this release cycle** — §4.1.5's growth projection was explicitly determined in Step 1 not to apply to warehouse staffing levels; without a stated growth target for this profile specifically, there's no NFR-driven basis to select Scalability Testing here (this could change if a future release states a warehouse-capacity growth target).

Volume Testing: not applicable — this profile has no large-data-batch dimension (that's PROF-04's concern).

## 2. Load Shape Design

**Load Testing:**

- This profile's "ramp-up" represents shift start, not a traffic ramp in the usual sense — `[ASSUMPTION - modeled as a 15min ramp representing staff logging into their handheld devices at shift start, followed by steady-state representing an active shift]`
- Steady state: 90 minutes, representing a sustained active-picking period within a shift (longer than the customer-facing profiles' 60min, since warehouse shifts are a longer, more uniform activity period without the same peak/off-peak shape)
- Ramp-down: not particularly meaningful for this profile (staff don't "ramp down" activity at the end of a shift the way traffic tapers) — `[ASSUMPTION - a brief 10min tail included mainly for symmetry with the other profiles' shape structure, acknowledged as the weakest-justified element of this shape]`

**Baseline Testing (Item Scan Confirmation, isolated):**

- Fixed concurrency representing a single picker's continuous scanning rate, scaled up to a small multi-picker baseline (`[ASSUMPTION]` 20 concurrent simulated pickers)
- Hold 20 minutes
- Purpose: establish Item Scan's own latency distribution before combining into the full-mix Load Test, given its outsized importance per Step 1's UBP flag

## 3. NFR to Acceptance Criteria Mapping

| Transaction                      | P95                                                                                                                                                                                                                            | P99                                    | Error Rate                                                                                                                                                                                                                                                     | Source                                                                                |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| Item Scan Confirmation           | ≤300ms                                                                                                                                                                                                                         | `[ASSUMPTION - ≤700ms, no P99 stated]` | ≤0.2%                                                                                                                                                                                                                                                          | §4.2.1, §4.5.3                                                                        |
| Picking List Retrieval           | ≤2000ms                                                                                                                                                                                                                        | `[ASSUMPTION - ≤4000ms]`               | `[ASSUMPTION - no explicit error-rate NFR for this specific transaction; §4.5's error-rate NFRs (4.5.1–4.5.3) don't explicitly list Picking List Retrieval; estimated ≤0.5% by analogy to Browse/Search's general-transaction rate, flagged for confirmation]` | §4.2.2                                                                                |
| Pack Confirmation                | `[ASSUMPTION - no explicit NFR found for this transaction; estimated ≤500ms P95 by analogy to Item Scan's speed-sensitivity, since packing staff are also mid-task workers per the same operational-cost reasoning in §4.5.3]` | `[ASSUMPTION]`                         | `[ASSUMPTION - ≤0.5%]`                                                                                                                                                                                                                                         | No direct source — flagged prominently                                                |
| Dispatch Assignment/Confirmation | `[ASSUMPTION - no explicit NFR found, as already flagged in Phase 1's gap log]`                                                                                                                                                | `[ASSUMPTION]`                         | `[ASSUMPTION]`                                                                                                                                                                                                                                                 | No direct source — flagged prominently, consistent with Phase 1's carried-forward gap |

This profile has substantially more NFR gaps than PROF-01/02/04 — two of its four transactions (Pack Confirmation, Dispatch Assignment) have no source-document NFR at all. This is stated plainly rather than smoothed over, and should be treated as a priority item for stakeholder follow-up before this profile's test case is finalized in a real engagement.

## Step 2 AI Gate Self-Check Summary

Stress and Spike Testing's non-selection is justified with profile-specific reasoning (staffing-bounded ceiling) rather than a copy-pasted rationale from the customer-facing profiles. The load shape explicitly acknowledges where its structure is weakly justified (the ramp-down tail) rather than presenting it with false confidence. NFR gaps for Pack Confirmation and Dispatch Assignment are surfaced prominently rather than filled in with unlabeled invented numbers — the estimates provided are explicitly marked as analogical reasoning, not sourced targets. Proceeding to Step 3 with these gaps carried forward as a priority flag.
