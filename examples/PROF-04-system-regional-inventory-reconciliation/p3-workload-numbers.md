# Workload Numbers — System: Regional Inventory Reconciliation (PROF-04)

## 1. Open vs. Closed Model Selection

| Sub-transaction                                                       | Model                                                                                                        | Justification                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| --------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| WMS Extract Ingestion, Inventory Reconciliation, Discrepancy Flagging | **Closed** — but not in the "fixed VU pool" sense; in the sense of a fixed, scheduled, non-arriving workload | This entire profile is triggered on a fixed schedule (once per region per night), not by independently-arriving traffic. There is no "arrival rate" to model as open — the workload's timing is entirely deterministic (01:00 local trigger). This is the clearest possible case for a closed-style model per Step 3's own guidance: "a genuinely fixed population... that cannot generate new requests while waiting" — here the "population" is the scheduler itself, triggering exactly three job instances per night, no more, no less. |

This profile is a direct illustration of why the Open vs. Closed technique's "when to use closed" criteria exist — not every workload has a meaningful open/closed choice at all; some are simply scheduled, and forcing an open-model framing onto a fixed-schedule batch job would misrepresent it.

## 2. Think Time & Pacing

**Not applicable in the human sense.** Per Step 1's Persona Behavioral Detailing, there is no human actor and therefore no think time. The structural equivalent here is the **scheduling interval and window duration**, not a pacing calculation:

- Trigger: fixed at 01:00 local time per region, no variance to model.
- Window: 3 hours (01:00–04:00 local), per §4.4.2 — this is the "duration budget" this profile's test must observe, analogous to pacing but fixed by schedule rather than calculated from execution+think time.

No Little's Law application follows from this — Little's Law relates concurrency, arrival rate, and time-in-system for a population of independently-timed items; a fixed, scheduled, single-instance-per-region job has no such population to model. This is stated explicitly rather than forcing the formula onto a case where it doesn't apply.

## 3. Concurrency — Regional Overlap Modeling (Replaces Little's Law for This Profile)

Since Little's Law doesn't apply, the relevant "concurrency" question for this profile is: **how much do the three regions' reconciliation windows overlap in UTC, and what does the shared-primary-DB contention risk (§5.2) imply for that overlap?**

`[ASSUMPTION - exact regional time zones aren't stated in the source README beyond "three metropolitan regions" (§1); for this example, assumed to be three US time zones (Pacific, Central, Eastern) as a plausible fit for a US-based grocery delivery platform, needs confirmation against GreenCart's actual regions]`.

With that assumption: Eastern's 01:00–04:00 local = 05:00–08:00 UTC (EST, ignoring DST for simplicity). Central's 01:00–04:00 local = 06:00–09:00 UTC. Pacific's 01:00–04:00 local = 08:00–11:00 UTC.

**Overlap analysis:** Eastern and Central overlap 06:00–08:00 UTC (2 hours of their respective 3-hour windows). Central and Pacific overlap 08:00–09:00 UTC (1 hour). Eastern and Pacific do not overlap at all under this assumed time-zone set. This means **at most two regions' Reconciliation sub-transactions run concurrently against the shared primary DB at any given time** under this assumed configuration — never all three simultaneously. This is a materially different, and less severe, contention scenario than "all three regions overlap," and is exactly the kind of concrete finding this profile's Load Testing (realistic overlap) and Stress Testing (forced full overlap) shape from Step 2 are designed to compare against each other.

## 4. Throughput Reconciliation — Record-Volume Framing (Replaces the Usual Transaction-Rate Table)

Since this profile has no arrival-rate throughput in the usual sense, "throughput" here is reframed as **records processed per unit time within the window**, which is what actually determines whether §4.4.2's window constraint is met:

| Region Pair (per assumed overlap analysis above) | Concurrent Record Volume (2 regions' worth, using Step 1's 15,000–40,000/region estimate) | Model            | Note                                                                                                                  |
| ------------------------------------------------ | ----------------------------------------------------------------------------------------- | ---------------- | --------------------------------------------------------------------------------------------------------------------- |
| Eastern + Central (2hr overlap)                  | 30,000–80,000 combined records processing concurrently against shared primary DB          | Closed/scheduled | This is the realistic worst-case concurrent load under the assumed time-zone configuration                            |
| Central + Pacific (1hr overlap)                  | 30,000–80,000 combined, shorter overlap window                                            | Closed/scheduled | Lower risk than the Eastern+Central overlap due to shorter duration                                                   |
| All three (Stress Testing's forced scenario)     | 45,000–120,000 combined                                                                   | Closed/scheduled | Not realistic under current assumed time zones, but the explicit worst-case Step 2's Stress Test is designed to probe |

## Step 3 AI Gate Self-Check Summary

This step correctly identifies that Open vs. Closed Model Selection's closed-model criteria fit this profile precisely, and explains why in profile-specific terms rather than applying the label mechanically; Think Time & Pacing and Little's Law are explicitly stated as not applicable, with the reasoning shown, rather than silently skipped or force-fitted; the regional time-zone assumption is flagged prominently since it drives every subsequent number in this section; the reframed "concurrency" (regional overlap) and "throughput" (record volume) both trace back to §5.2's documented risk and Step 1's record-volume estimate. Proceeding to Step 4 with the time-zone assumption carried forward as a priority item for Human Review Gate 2.
