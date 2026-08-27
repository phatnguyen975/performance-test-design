# Output Template — `p4-test-data-specification.md` (Step 4 Intermediate Deliverable)

This is the **verbose, inspectable** artifact — kept separate from the final `test-case-spec.md`.

```markdown
# Test Data Specification — [Profile Name] ([Profile ID])

## 1. Data Parameterization Specification

[Per transaction, table: Field | Type/Format | Required Volume | Source | Reuse Policy | Uniqueness Constraint]

## 2. Correlation Mapping Specification

[Per multi-step transaction, table: Value | Source Step/Field | Destination Step(s)/Field | Lifetime]

## 3. Data Diversity Rules

[Per data-volume-sensitive field: Pool Size | Distribution Shape (uniform / two-tier weighted / Zipfian) | Reasoning]
[Explicit note for any field where diversity concerns don't apply, and why]

## 4. Script Blueprint Specification

### Initialization

### Main Flow

### Verification Points

### Error Handling

### Cleanup/Termination

## Step 4 AI Gate Self-Check Summary

[Confirmation output-quality-checklist.md was run, and its result]
```
