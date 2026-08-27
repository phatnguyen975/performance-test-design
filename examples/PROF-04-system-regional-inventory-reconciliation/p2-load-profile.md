# Load Profile — System: Regional Inventory Reconciliation (PROF-04)

## 1. Test Type Selection

| Test Type      | Targets                                                                                            | Reason                                                                                                                                                                                                                         |
| -------------- | -------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Volume Testing | Inventory Reconciliation                                                                           | Explicitly a data-volume-centric transaction (per Step 1's record-count discussion); README §4.4.2's "complete within window, ≤0.5% failure rate" NFR is fundamentally a volume-and-duration constraint, not a concurrency one |
| Load Testing   | Full sequence (all 3 sub-transactions), run as three overlapping-window instances (one per region) | Default — but "load" here means realistically overlapping regional windows, not concurrent human users; validates §4.4.2's window/failure-rate NFR under the real overlap pattern from §5.2                                    |
| Stress Testing | Inventory Reconciliation                                                                           | Directly targets §5.2's documented lock-contention risk — deliberately increase the degree of regional-window overlap beyond what's currently observed to find the point where contention causes the job to miss its window    |

Scalability, Spike, Soak, Baseline, Capacity: **considered, not selected.** Spike/Soak don't fit a single nightly scheduled job with no sustained human-driven traffic. Scalability wasn't selected because no growth projection in README §4 was stated for this specific batch job (§4.1.5's growth figure is explicitly scoped to customer-facing sessions). Baseline wasn't selected because there's no "full mix to isolate from" in the way customer-facing profiles have — this profile's three sub-transactions already run in isolation from any other traffic by nature of being a dedicated overnight batch window.

## 2. Load Shape Design

This profile's "load shape" is fundamentally different from a human-traffic profile — it's not a ramp-up/steady-state/ramp-down curve, it's a **scheduling and overlap simulation.**

**Load Testing (realistic overlap):**

- Simulate all three regions' actual local 01:00–04:00 windows, converted to UTC, with their real partial overlap as derived from each region's time zone (this is a scheduling simulation, not a VU ramp — "load" is expressed as simultaneous job execution, not concurrent users)
- Each region's Reconciliation sub-transaction runs against the estimated record volume from Step 1 (15,000–40,000 SKUs, flagged as an assumption)
- Duration: the full 3-hour window per region, since §4.4.2's NFR is scoped to "complete within the window," which requires observing the whole window, not a sampled steady-state slice

**Volume Testing (Inventory Reconciliation):**

- Run the Reconciliation sub-transaction against a range of record-volume assumptions (the low end and high end of Step 1's 15,000–40,000 estimate, plus a stretch scenario at 60,000 to check headroom) to observe how duration and failure rate scale with volume — directly informs whether the assumed volume range is itself safely within the window
- Not driven by a ramp/steady-state pattern — driven by dataset size as the independent variable, with job duration and failure rate as the dependent variables being observed

**Stress Testing (contention risk):**

- Deliberately shift the simulated regional windows to increase their UTC overlap beyond the current real-world pattern (e.g., simulate a scenario where all three regions' windows fully coincide, as a worst-case beyond today's partial overlap) — this directly stresses the shared-primary-DB contention risk flagged in §5.2
- Stop condition: the point at which any region's Reconciliation sub-transaction fails to complete within its 3-hour window, or the failure rate exceeds §4.4.2's 0.5% threshold

## 3. NFR to Acceptance Criteria Mapping

| Sub-transaction          | Criterion                                                                                   | Source                                                                                                    |
| ------------------------ | ------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| WMS Extract Ingestion    | No explicit NFR found                                                                       | `[ASSUMPTION - flagged, needs business confirmation on acceptable ingestion latency once a file arrives]` |
| Inventory Reconciliation | Must complete within the region's 01:00–04:00 local window; ≤0.5% record-level failure rate | §4.4.2                                                                                                    |
| Discrepancy Flagging     | No explicit NFR found beyond being part of the overall window/failure-rate constraint above | §4.4.2 (treated as covered by the parent constraint, not separately targeted)                             |

This profile has no response-time-percentile criteria at all (no P95/P99 makes sense for a scheduled batch job measured by window-completion and failure-rate instead) — this is a structural difference from every other profile in this example set, not an omission.

## Step 2 AI Gate Self-Check Summary

Test type selection explicitly explains why the usual defaults (Load as universal default in the strict sense, Baseline, Scalability) don't straightforwardly apply, rather than silently forcing them; the load shape is explicitly reframed as a scheduling/overlap simulation rather than a VU ramp, since a human-traffic-shaped curve would misrepresent this profile; acceptance criteria correctly omit percentile-based targets since none apply, with the reasoning stated rather than left implicit. The WMS Extract Ingestion NFR gap is flagged. Proceeding to Step 3.
