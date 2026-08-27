# Step 2 — Load Profile Design

**ISTQB CT-PT reference:** Chapter 4.2.4 — Creating Load Profiles. Also draws on the performance test types catalog (ISTQB CT-PT Module 1) and metrics catalog (Module 1, Chapter 2.1).

## Purpose

Decide **what kind of test** this profile needs (Load, Stress, Spike, Soak/Endurance, Scalability, Volume, Capacity, Baseline) and **what shape the load takes over time**. Also translate this profile's NFRs into concrete, measurable, per-transaction acceptance criteria. This step decides the _shape_; Step 3 decides the _numbers_ inside that shape.

## Prerequisite

Step 1's completed and AI-Gate-passed Profile Analysis: transaction list with boundaries, transaction mix %, and UBP risk flags.

## Steps

1. **Test Type Selection** — `techniques/test-type-selection.md` → Choose which test type(s) this profile's transactions need — not every profile needs every type, and not every transaction within a profile needs the same type(s).
2. **Load Shape Design** — `techniques/load-shape-design.md` → For each selected test type, design the load curve.
3. **NFR to Acceptance Criteria Mapping** — `techniques/nfr-to-acceptance-criteria-mapping.md` → Convert NFRs into specific, measurable, per-transaction thresholds.

## Supporting Material

- `resources/load-generation-approaches.md`
- `resources/performance-metrics-reference.md`
- `output-template.md`
- `output-quality-checklist.md` — **run this before moving to Step 3**
- `best-practices.md`
- `anti-patterns.md`

## Output

Fill in `output-template.md` to produce `p2-load-profile.md`.

## Handing off to Step 3

Step 3 needs: the chosen test type(s), the steady-state duration per type, and a target throughput or concurrent-user figure to validate against (from Step 1's growth-adjusted frequencies), since Step 3's Little's Law calculation reconciles against a duration and a rate.
