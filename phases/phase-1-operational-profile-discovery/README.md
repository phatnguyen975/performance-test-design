# Phase 1 — Operational Profile Discovery

**ISTQB CT-PT reference:** Chapter 4.2.3 — Identifying Operational Profiles (system-wide discovery aspect). Extended with practical requirements-analysis technique beyond the syllabus's summary treatment.

## Purpose

Read the system's available documentation exactly once, and produce a single, reviewable list of every Operational Profile the system needs performance-tested — numbered, named, scoped, and prioritized. This is the only phase where the _whole system_ is in view at once; every subsequent step in Phase 2 works on one profile in isolation.

Getting this phase right matters disproportionately: a profile missed here is a testing gap that won't surface until Phase 2 is long finished for the profiles that _were_ found, and a profile scoped too broadly here (e.g., blending two distinct user populations) corrupts every calculation downstream in Phase 2 for that profile.

## Prerequisite

At least one system document (see root `SKILL.md` → Input). More document types available means a more defensible discovery — but this phase does not block on having every document type; it proceeds with what exists and flags gaps explicitly.

## Steps

Work through these three techniques **in order**:

1. **System Document Analysis** — `techniques/system-document-analysis.md` → Extract user roles, business flows, NFR-relevant constraints, and system boundaries from every available document.
2. **Profile Boundary Definition** — `techniques/profile-boundary-definition.md` → Partition what was extracted into discrete, non-overlapping Operational Profiles, each scoped to one user population and one coherent set of tasks.
3. **Profile Prioritization** — `techniques/profile-prioritization.md` → Order the resulting profile list by business criticality, risk, and dependency, so Phase 2 tackles them in a defensible sequence.

## Supporting Material

- `resources/profile-types-reference.md` — a reference catalog of common Operational Profile categories (customer-facing, backoffice/admin, system-to-system, batch/scheduled) to check the discovered list against for completeness
- `output-template.md` — the structure for `operational-profiles.md`
- `output-quality-checklist.md` — **run this before presenting to Human Review Gate 1**
- `best-practices.md`
- `anti-patterns.md`

## Output

Fill in `output-template.md` to produce `operational-profiles.md` — the deliverable of this entire phase.

## Handing off to Phase 2

Phase 2 needs, at minimum, from this phase's output: each profile's ID, name, one-line scope description, and priority order. Confirm the list has passed Human Review Gate 1 (see root `SKILL.md`) before starting Phase 2 for Profile #1.
