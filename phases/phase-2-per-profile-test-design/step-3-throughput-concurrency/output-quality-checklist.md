# Output Quality Checklist — Step 3: Throughput & Concurrency Analysis (AI Gate)

- [ ] Every transaction has an explicit open/closed workload model decision, with justification.
- [ ] Think time is a range/distribution per step, not a fixed constant, for every human-driven transaction; none use zero.
- [ ] Pacing is calculated explicitly, showing execution-time and think-time components separately, with the percentile used stated.
- [ ] λ was derived both from historical/measured data (Method A) and from any independently-stated NFR target (Method B), where Method B data exists.
- [ ] Any discrepancy between Method A and Method B is surfaced explicitly, not silently resolved by averaging or discarding the lower figure.
- [ ] Little's Law's arithmetic (N = λ × W) is fully visible, not just a final number.
- [ ] A safety margin was applied and stated explicitly.
- [ ] The final VU count was sanity-checked against the known population size.
- [ ] The per-transaction throughput table reconciles against Step 1's source frequency data within a reasonable, explained margin.
- [ ] Any transaction paced by non-human logic is modeled from its worker-pool size, not forced through human think-time-based calculation.
- [ ] It is stated explicitly whether the final VU count represents typical-day load or a specific higher-traffic scenario.
- [ ] No content assumes or references a specific load-testing tool.

If any item fails, fix it here before moving to Step 4.
