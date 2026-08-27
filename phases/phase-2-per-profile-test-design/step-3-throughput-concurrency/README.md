# Step 3 — Throughput & Concurrency Analysis

**ISTQB CT-PT reference:** Chapter 4.2.5 — Analyzing Throughput and Concurrency.

## Purpose

Turn Step 2's load shape and acceptance criteria into concrete workload numbers: how many virtual users, at what think time, generating what throughput, under which workload model (open vs. closed).

## Prerequisite

Step 2's completed and AI-Gate-passed Load Profile: test type(s), load shape, per-transaction acceptance criteria including any target throughput.

## Steps

1. **Open vs. Closed Model Selection** — `techniques/open-vs-closed-model.md`
2. **Think Time & Pacing** — `techniques/think-time-and-pacing.md`
3. **Little's Law Application** — `techniques/littles-law-application.md`
4. **Throughput Reconciliation** — `techniques/throughput-reconciliation.md`

## Supporting Material

- `resources/workload-math-reference.md`
- `output-template.md`
- `output-quality-checklist.md` — **run this before moving to Step 4**
- `best-practices.md`
- `anti-patterns.md`

## Output

Fill in `output-template.md` to produce `p3-workload-numbers.md`.

## Handing off to Step 4

Step 4 needs: final VU count (total and per-transaction), think time values, target TPS/RPS, and the workload model (open/closed) per transaction.
