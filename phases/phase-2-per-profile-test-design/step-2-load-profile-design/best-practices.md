# Best Practices — Step 2: Load Profile Design

- Always select Load Testing by default, then add other types deliberately based on Step 1's UBP flags and NFR language.
- Design a distinct load shape per test type — never reuse a Load Test's shape for Stress or Spike.
- Base ramp-up pacing on real traffic-build patterns when available.
- Always express response-time acceptance criteria as percentiles (P95 minimum), never a bare average.
- Cross-reference hard technical constraints from Step 1 against acceptance criteria being set.
- State explicit stop conditions for Stress/Capacity Testing.
- Flag any transaction with no discoverable NFR explicitly rather than inventing an "industry standard" threshold.
