# Output Template — `test-case-spec.md` (Step 4 Final, Condensed Deliverable)

This is the **final deliverable** of the entire per-profile design process — the document an implementer will actually work from to write test scripts. It is deliberately **condensed**: it does not repeat the full reasoning trails from the four step outputs. Link back to them by filename for anyone who wants the full derivation; state only the essential, actionable content here.

**Include:** final decisions and numbers. **Exclude:** step-by-step reasoning, source-citation prose, and self-check summaries that belong in the step output files.

```markdown
# Performance Test Case Specification — [Profile Name] ([Profile ID])

**Status:** [Draft / Pending Human Review / Approved]
**Related step documents:** `p1-profile-analysis.md` · `p2-load-profile.md` · `p3-workload-numbers.md` · `p4-test-data-specification.md`

## 1. Test Case Header

- **Test Case ID:** [ID]
- **Objective:** [one or two sentences]
- **Test Type(s):** [from Step 2, list only]
- **Preconditions:** [environment state, data seeding, feature flags, sandbox credentials needed]

## 2. Scenario Flow Summary

[Ordered list of transactions in one iteration, each with its think time — condensed from Step 4's Script Blueprint Main Flow]

## 3. Load Profile Summary

[Per test type: load shape parameters only — condensed from Step 2]

## 4. Workload Numbers

- Workload model: [open/closed, per transaction if it varies]
- Virtual users: [final number] — [note explicitly if this represents a specific scenario, e.g. promotional-event scale, vs. typical-day load]
- Target throughput: [per transaction, condensed table]
- Think time: [range per step, condensed table]

[Condensed from Step 3 — numbers only, not the Little's Law derivation]

## 5. Test Data Requirements

[Condensed table: Field | Required Volume | Source | Notes (uniqueness/diversity flags only) — condensed from Step 4's Data Parameterization + Diversity Rules]

## 6. Correlation Points

[Condensed table: Value | Extracted From | Used In — condensed from Step 4's Correlation Mapping, lifetime notes only where non-obvious]

## 7. Acceptance Criteria

[Table: Transaction | P95 | P99 | Error Rate — condensed from Step 2]

## 8. Verification Points & Error Handling

[Condensed from Step 4's Script Blueprint]

## 9. Open Questions / Assumptions Requiring Confirmation

[Every item across all four steps labeled `[ASSUMPTION]` — consolidated here]
```

## Notes on Filling This In

- Section 9 is not optional even if empty — state "None outstanding" explicitly if every number was sourced.
- If any AI Gate required a fix-and-rerun cycle, this final document reflects only the corrected, final state — not a changelog.
- Keep tables condensed to what an implementer needs to configure a script and its data.
