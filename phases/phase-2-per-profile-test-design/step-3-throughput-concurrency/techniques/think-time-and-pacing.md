# Technique: Think Time & Pacing

**ISTQB CT-PT reference:** 4.2.5 — workload realism component.

## What It Is

**Think time** is the pause a real user takes between actions before issuing the next request. **Pacing** is the total time a virtual user's iteration takes to complete one full pass through its scenario (execution time + think time).

## When to Use

For every transaction/scenario in the profile, always.

## When NOT to Use

Not applied literally to pure backend/batch transactions with no human in the loop — those are paced by their own processing logic, not human think time.

## How to Apply

1. Where session-level analytics, HAR captures, or session-replay data exist (see Step 1's `resources/data-gathering-sources.md`), derive think time directly from observed inter-request gaps within a session, per transaction step.
2. Where no such data exists, estimate based on the cognitive complexity of the step, and label the estimate explicitly.
3. Use randomized/distributed think time (drawn from a range or distribution) rather than a single fixed constant — a fixed value across all VUs creates artificial synchronization ("thundering herd").
4. Calculate pacing as: (execution time — sum of transaction response times, using a representative percentile such as P50) + (sum of think times between steps).
5. Never set think time to zero. If more load is needed, add more virtual users; don't distort the model by removing a required realism factor.

## Output

A think-time table per transaction step (range or distribution) and a calculated pacing value per scenario.

## Example

| Step                                 | Think Time             | Source                        |
| ------------------------------------ | ---------------------- | ----------------------------- |
| Product list → click product         | 3–8s (uniform random)  | Session clickstream analytics |
| Product detail → Add to Cart         | 5–15s (uniform random) | Same source                   |
| Cart page → Proceed to Checkout      | 4–10s (uniform random) | Same source                   |
| Payment form → Confirm Payment click | 8–20s (uniform random) | Session data                  |

**Pacing calculation:** Sum of transaction execution times (P50) ≈ 5.9s. Sum of think times (midpoints) ≈ 41s. **Total pacing ≈ 47s per full iteration.**
