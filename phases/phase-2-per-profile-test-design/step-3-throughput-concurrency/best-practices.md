# Best Practices — Step 3: Throughput & Concurrency Analysis

- Default to the open workload model for any transaction where arrivals are genuinely independent of system response speed.
- Always use randomized/distributed think time, never a fixed constant.
- Never set think time to zero — add virtual users instead if more load is needed.
- Always derive λ two independent ways (historical data and any stated NFR target) and reconcile explicitly — don't rely on a single derivation path.
- Always show the λ, W, and arithmetic explicitly when applying Little's Law.
- Cross-check every derived number against Step 1's source frequency data before finalizing.
- State explicitly which percentile of execution time was used for pacing (commonly P50).
- Flag explicitly when a calculated VU count represents a specific scenario (e.g., a promotional event) rather than typical-day load, so it isn't misapplied later.
- Recognize when a transaction is paced by non-human logic (a backend worker pool) and model its concurrency directly from its worker-pool size.
