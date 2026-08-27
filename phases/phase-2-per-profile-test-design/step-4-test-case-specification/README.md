# Step 4 — Test Case Specification

**ISTQB CT-PT reference:** Chapter 4.2.6 — Basic Structure of a Performance Test Script (structural/logical aspects only), and Chapter 4.2.7 — Implementing Performance Test Scripts (specification aspects only: parameterization, correlation; **not** the code-writing aspects).

## Purpose

Produce the data requirements and logical script structure a future implementer needs, then merge everything from Steps 1–3 into one condensed, final **Performance Test Case Specification**. This step does not write test scripts or tool-specific code.

## Prerequisite

Step 3's completed and AI-Gate-passed workload numbers.

## Steps

1. **Data Parameterization Specification** — `techniques/data-parameterization-specification.md`
2. **Correlation Mapping Specification** — `techniques/correlation-mapping-specification.md`
3. **Data Diversity Rules** — `techniques/data-diversity-rules.md`
4. **Script Blueprint Specification** — `techniques/script-blueprint-specification.md`

## Supporting Material

- `resources/test-data-types-reference.md`
- `output-template-data-spec.md` — for the intermediate `p4-test-data-specification.md`
- `output-template-test-case-spec.md` — for the final, condensed `test-case-spec.md`
- `output-quality-checklist.md` — **run this before this profile returns to Human Review Gate 2**
- `best-practices.md`
- `anti-patterns.md`

## Output — Two Distinct Documents

1. **`p4-test-data-specification.md`** (intermediate, verbose) — the full output of Steps 1–3 above, kept for review/audit.
2. **`test-case-spec.md`** (final, condensed) — merges the _essential_ outputs of all four Phase 2 steps plus this step's Script Blueprint into the single document an implementer will actually work from.

## After This Step

Return to the root `SKILL.md`'s Human Review Gate 2 for this profile — present all four step outputs plus the final spec together.
