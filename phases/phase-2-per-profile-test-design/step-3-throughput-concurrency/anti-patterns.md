# Anti-Patterns — Step 3: Throughput & Concurrency Analysis

- Defaulting to a closed workload model out of tooling convenience without checking whether the transaction's real arrival pattern is open — risks coordinated omission.
- Using a single fixed think-time constant across all virtual users, creating synchronized request bursts.
- Setting think time to zero "to generate more load faster."
- Deriving λ only one way (e.g., only from historical data) when an independently-stated NFR concurrency target also exists — silently ignoring the higher of the two risks under-designing the test.
- Picking a virtual-user count with no visible Little's Law calculation behind it.
- Mixing time units without converting.
- Using a strict percentile (P95/P99) for the execution-time component of pacing, understating achievable throughput per VU.
- Skipping the reconciliation step against Step 1's source frequency data.
- Failing to flag that a calculated VU count represents a specific high-traffic scenario rather than typical-day load, risking its misuse as a default target elsewhere.
