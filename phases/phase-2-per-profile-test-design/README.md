# Phase 2 — Per-Profile Test Case Design

**ISTQB CT-PT reference:** Chapter 4.2.1 through 4.2.7, applied per individual Operational Profile.

## Purpose

Take one Operational Profile — approved out of Phase 1's `operational-profiles.md` — and design its complete performance test case: what to test, how the load behaves, how many virtual users/what throughput, and what test data is required. This phase is a **loop**: it runs once, completely, per profile, in the priority order Phase 1 established, with a human review checkpoint after every single profile before the next one begins.

## Prerequisite

Phase 1's `operational-profiles.md` has passed Human Review Gate 1 (see root `SKILL.md`).

## The Loop

```
FOR EACH profile IN operational-profiles.md, IN PRIORITY ORDER:

    STEP 1 — Profile Analysis         → step-1-profile-analysis/README.md
    STEP 2 — Load Profile Design      → step-2-load-profile-design/README.md
    STEP 3 — Throughput & Concurrency → step-3-throughput-concurrency/README.md
    STEP 4 — Test Case Specification  → step-4-test-case-specification/README.md

    PRESENT all four step outputs + the final Test Case Specification together
    ASK the human to approve this profile, or request changes

    IF changes requested:
        Identify which step introduced the issue
        Fix that step's output
        Re-run every step DOWNSTREAM of the fix (not the steps before it)
        Re-present and ask again

    IF approved:
        Mark this profile DONE
        Continue to the next profile in the list (if any remain)
```

## Why Four Steps, Not Four Independent Phases

These four steps are **not** independent — each one's input is the previous one's output, for this specific profile:

- Step 1 produces the transaction list and mix that Step 2's test-type selection and acceptance-criteria mapping depend on.
- Step 2 produces the load shape and target throughput that Step 3's Little's Law calculation depends on.
- Step 3 produces the VU count and workload model that Step 4's data-volume and script-structure specification depend on.

Running them out of order, or skipping one, produces numbers in a later step that don't reconcile with an earlier step's output — this is why the root `SKILL.md`'s Design Rules explicitly forbid skipping or reordering them.

## Technique Selection Within Each Step

Every step contains multiple techniques, and **not every technique applies to every transaction** within a profile. Each technique file states explicit "When to Use / When NOT to Use" criteria — read them per-transaction, not per-profile. A profile with six transactions might need Stress Testing techniques applied to two of them and not the other four; this is normal and expected, not a shortcut.

## Handing Off Between Steps

Each step's own `README.md` states exactly what it needs from the step before it, under a "Prerequisite" heading, and what it must hand off, under a "Handing off to Step N+1" heading. Confirm these explicitly when moving between steps — don't assume a step's output is sufficient without checking it against the next step's stated prerequisite.

## After All Four Steps

Return to the root `SKILL.md`'s Human Review Gate 2 for this profile. Once approved, check whether more profiles remain in `operational-profiles.md`'s priority order — if so, restart this loop for the next one; if not, the design process for this system is complete.
