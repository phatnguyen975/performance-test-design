# Technique: Throughput Reconciliation

**ISTQB CT-PT reference:** 4.2.5 — reconciliation/output component.

## What It Is

The final synthesis step of Step 3: reconciling the overall VU count (from Little's Law) against Step 1's transaction mix to produce a **per-transaction throughput and concurrency table** — the concrete numbers Step 4 needs.

## When to Use

As the last step of Step 3, always, after the workload model and VU count have been established.

## When NOT to Use

Don't skip this reconciliation even when the overall VU count "seems obviously right" — the per-transaction breakdown is what later steps actually consume.

## How to Apply

1. Take the finalized VU count from Little's Law Application.
2. Apply Step 1's transaction mix percentages to derive the expected per-transaction request rate this VU population generates at steady state.
3. Cross-check this derived rate against Step 1's source frequency data — a significant mismatch signals an error earlier in the chain (unit mismatch, wrong pacing value, mix percentage misapplied) and must be resolved before proceeding.
4. Produce the final combined table: per-transaction target throughput, workload model, VU count contributing to it.
5. Note any transaction whose achieved throughput under this VU/pacing combination would fall short of Step 2's acceptance-criteria throughput target — a planning-stage warning that the scenario may need adjustment before test execution.

## Output

The final workload numbers table for the profile, combining VU count, throughput per transaction, workload model, and pacing.

## Example

Using VU count (550) and pacing (47s) from Little's Law Application:

Achieved iteration rate: 550 / 47 ≈ 11.70 iterations/sec ≈ 42,128 iterations/hr.

| Transaction     | Target (Step 1, growth-adj.) | Achieved (550 VU @ 47s) | Model | Reconciliation                                                                                                                                 |
| --------------- | ---------------------------- | ----------------------- | ----- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| Submit Order    | 4,440/hr                     | ≈42,128/hr              | Open  | Far above historical target — consistent with the NFR-driven promotional-event scenario this VU count was designed for, not typical-day volume |
| Confirm Payment | 4,610/hr                     | ≈42,128/hr              | Open  | Same — this VU count represents a specific high-traffic scenario per Little's Law's Method B reconciliation note                               |

**Result:** the 550-VU configuration is explicitly a **promotional-event-scale** design, not a typical-day design — this distinction must carry forward clearly into Step 4 and the final Test Case Specification so an implementer doesn't mistake this for the profile's everyday load. Consider, if a typical-day Load Test is also needed, running Little's Law Application a second time using only Method A's historical figure to produce a separate, smaller-scale test case — the root skill's Design Rules permit re-running a step's calculation for a documented reason like this without contaminating the current profile's approved design.
