# Technique: Little's Law Application

**Grounding:** ISTQB CT-PT 4.2.5 (core calculation), extended with the dual-method cross-check common in capacity-planning practice — deriving target throughput two independent ways and reconciling them, rather than trusting a single derivation path.

## What It Is

Little's Law is a queueing-theory result: **N = λ × W**, where:

- **N** = average number of concurrent items in a system (concurrent virtual users/sessions)
- **λ** (lambda) = arrival rate (target throughput)
- **W** = average time each item spends in the system (pacing — execution time + think time)

This gives a defensible starting point for how many virtual users a test needs to configure to achieve a target throughput, rather than picking a VU count arbitrarily.

## When to Use

Whenever a target throughput needs translating into a virtual-user count. Also as a sanity check on any VU count arrived at by other means.

## When NOT to Use

Not a substitute for actually measuring real concurrency once the test runs. Not directly applicable to a closed-model backend transaction paced by a fixed worker pool — for those, the worker pool size _is_ N already, defined directly.

## How to Apply

1. **Derive λ two independent ways and reconcile them:**
   - **Method A — from Step 1's peak-hour frequency data:** λ = growth-adjusted peak-hour transaction frequency, converted to a per-second rate.
   - **Method B — from any independently-stated NFR concurrency/throughput target:** if the NFR document states a target directly (e.g., "must support 500 concurrent checkout sessions"), derive the implied λ from that target using W (rearranged: λ = N_target / W) and compare against Method A.
   > Use whichever value is **higher** as the design target unless there's a specific, documented reason to do otherwise — designing to the lower of two credible estimates risks under-provisioning the test against a real requirement that Method A's historical data might not fully capture (e.g., historical data predates a recent marketing push that the NFR document's target already accounts for).
2. **Take the pacing value (W) from Think Time & Pacing, in the same time unit as λ.**
3. **Calculate N = λ × W.**
4. **Round up to a sensible whole number and add a safety margin (commonly 5–10%)** to account for real-world pacing variance.
5. **Cross-check against business intuition** — if N is implausibly large or small relative to the known user base size, re-examine λ or W for an error (a common mistake is a unit mismatch) before proceeding.

## Output

The calculated concurrent virtual-user count (N), showing both λ derivations, which one was used and why, the W value, the arithmetic, and the safety-margin-adjusted final figure.

## Example

**Method A (from Step 1 frequency data):** Scenario-iteration arrival rate ≈ Submit Order frequency = 4,440/hr (growth-adjusted) = 1.233 iterations/sec.

**Method B (from NFR):** NFR-DOC-03 §5 states "must support 500 concurrent checkout sessions at peak" — with W = 47s, implied λ = 500 / 47 ≈ 10.64 iterations/sec.

**Reconciliation:** Method B's implied rate (10.64/sec) is substantially higher than Method A's historical rate (1.233/sec) — this is a real discrepancy worth surfacing explicitly rather than averaging away, since it suggests the NFR's concurrency target was set with a specific future event in mind (e.g., a major promotional campaign) that historical APM data doesn't reflect. **Use Method B's higher, NFR-derived figure as the design target**, and note the discrepancy explicitly as a flag for Human Review Gate 2 — the business should confirm whether 500 concurrent sessions is the correct target before this number drives infrastructure-affecting test design.

**N = λ × W = 10.64 × 47 ≈ 500** (consistent by construction, since Method B was derived by rearranging the same formula from the NFR's own stated concurrency figure).

Adding a 10% safety margin: 500 × 1.10 = **550 concurrent virtual users (final).**

**Sanity check:** 550 concurrent VUs against a ~180,000 active-user base is a plausible, if aggressive, slice — consistent with the NFR document's framing that this target represents a specific high-traffic event scenario, not typical-day concurrency.
