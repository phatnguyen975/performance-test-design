# Technique: Load Shape Design

**ISTQB CT-PT reference:** 4.2.4 — Creating Load Profiles.

## What It Is

Designs the shape of the activity curve for each test type selected: how load ramps up, holds steady, ramps down, and — for Spike/Stress — the specific perturbation pattern layered on top.

## When to Use

For every test type selected in Test Type Selection — each needs its own shape.

## When NOT to Use

Don't design a generic "one shape fits all tests" — a Stress Test's ramp keeps climbing past where a Load Test would stop; reusing the Load Test's ramp truncates the Stress Test's ability to find a breaking point.

## Shape Components, by Test Type

- **Load Testing:** Ramp-up gradual (stepped increases over 10–30 min, avoiding an artificial "thundering herd" at t=0) → Steady state held long enough for statistically stable percentile metrics (typically 30–60 min minimum, longer for lower-volume transactions to accumulate enough samples) → Ramp-down gradual, to observe clean resource release.
- **Stress Testing:** Ramp-up continues in steps _beyond_ expected peak, holding briefly at each step to observe where response time/error rate degrades disproportionately. No fixed steady-state target. Include a recovery-observation period after ramp-down.
- **Spike Testing:** Baseline load, then a sharp, short increase (seconds to minutes) to a multiple of normal volume, then return to baseline. Spike duration should reflect the real-world trigger. Explicitly measure recovery time to pre-spike baseline.
- **Soak/Endurance Testing:** Ramp-up to a sustained, moderate (not peak) load level. Steady state held for an extended period (commonly 8–72 hours depending on the degradation pattern under investigation). Track resource-utilization trend lines continuously, not just response time — degradation frequently shows there before response time.
- **Capacity Testing:** Similar ramp-up to Stress Testing, but the stop condition is defined by the _first_ acceptance-criteria violation (not a more extreme breaking point) — the goal is finding the maximum sustainable throughput within acceptable criteria, not finding total failure.

## How to Apply

1. For each selected test type, specify: ramp-up pattern (steps, duration/step), steady-state duration, ramp-down, and any perturbation parameters.
2. Base ramp-up pace on real-world traffic-build patterns where known (from Step 1's frequency data if it shows how quickly traffic actually builds during a comparable real event).
3. For infrastructure with auto-scaling, align ramp-up step intervals with the auto-scaler's reaction time.
4. State explicit stop conditions for Stress/Capacity Testing so the test has a defined end.

## Output

A load shape specification per test type: ramp-up steps/duration, steady-state duration, ramp-down, perturbation parameters, and stop conditions where relevant.

## Example

- **Load Testing:** Ramp-up 6×5min steps (30min total) → Steady state 60min at growth-adjusted target volume → Ramp-down 10min linear.
- **Stress Testing (Confirm Payment, Submit Order):** Ramp +20% over Load Test target every 10min, hold 5min/step. Stop: error rate >5% OR P95 >3x passing P95 for 2 consecutive steps. Recovery observation: 15min at zero added load.
- **Spike Testing (Browse Catalog, Search, Add to Cart):** Baseline = Load Test steady state, held 10min → spike to 4x baseline over 60s (matching observed flash-sale opening-rush pace from Step 1 data) → hold 5min → return over 60s → hold 15min to measure recovery.
