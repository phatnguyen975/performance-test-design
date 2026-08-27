# Resource: Workload Math Reference

## Little's Law

```
N = λ × W
λ = N / W (rearranged)
W = N / λ (rearranged)
```

N = concurrent items; λ = arrival rate; W = average time in system (pacing).

## Pacing (W)

```
W = execution_time + think_time
execution_time = sum of transaction response times in one iteration (use P50 for planning)
think_time     = sum of think-time pauses between steps in one iteration
```

## Converting Time Units

```
requests/hour ÷ 3600 = requests/second
requests/second × 3600 = requests/hour
```

Always double-check units before plugging into Little's Law.

## Safety Margin

```
Final VU count = ceil(N × (1 + margin))     margin commonly 5–10%
```

## Dual-Method λ Reconciliation

```
λ_A = derived from historical/measured frequency (Step 1 data)
λ_B = derived from an independently-stated NFR concurrency/throughput target
Use max(λ_A, λ_B) as the design target unless documented otherwise; always
surface the discrepancy explicitly rather than silently averaging or discarding
the lower estimate.
```

## Sanity-Check Heuristics

- N should never exceed the realistic size of the population the profile represents.
- If N comes out much smaller than expected, check whether λ was accidentally computed from a sub-flow's rate rather than the full scenario's iteration rate.
- Recomputing λ = N / W using the final rounded VU count should reproduce something close to the original target — if not, the rounding/margin step introduced more distortion than intended.
