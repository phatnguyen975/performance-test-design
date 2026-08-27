# Output Quality Checklist — Step 2: Load Profile Design (AI Gate)

- [ ] Load Testing is selected by default for the full mix.
- [ ] Every additional test type traces to a specific NFR statement or Step 1 UBP flag.
- [ ] Each selected test type has its own distinct load shape.
- [ ] Ramp-up pacing is gradual, based on real traffic-build data where possible.
- [ ] Stress/Capacity Testing (if selected) has an explicit, measurable stop condition.
- [ ] Every transaction has a mapped acceptance criterion, or an explicit flagged gap.
- [ ] Every response-time criterion is expressed as a percentile (P95 minimum).
- [ ] Acceptance criteria are cross-checked against any hard technical constraint from Step 1.
- [ ] Ramp-up pacing is aligned with auto-scaler reaction intervals, if applicable.
- [ ] No content assumes or references a specific load-testing tool.

If any item fails, fix it here before moving to Step 3.
