# Output Template — `operational-profiles.md` (Phase 1 Deliverable)

```markdown
# Operational Profiles — [System Name]

## Source Documents

[List every document actually used, with version/date if known. State explicitly which document types from root SKILL.md's Input table were NOT available.]

## 1. System Document Analysis Summary

### Actors

[Table: Actor | Primary/Secondary | Functional Role | Notes]

### Business Events / Flows

[Table: Flow | Source Document | Associated Actor(s)]

### NFR/Constraint Extraction

[Table: NFR | Applies To | Source | Gap flag if vague/missing]

### System Boundary Summary

[Brief protocol/tier map at system-wide granularity]

### Gap Log

[Explicit list of anything a complete discovery would need but wasn't available]

## 2. Operational Profile List

[Table: ID | Name | Actor(s) | Scope (included/excluded) | Business events assigned]

### Coverage Cross-Check

[Confirmation every business event maps to exactly one profile — list any that mapped to zero or more than one, and how it was resolved]

## 3. Prioritization

[Table: Order | ID | Name | Business Criticality | Risk Exposure | Volume | Priority Justification]

## Phase 1 AI Gate Self-Check Summary

[Confirmation output-quality-checklist.md was run, and its result]
```
