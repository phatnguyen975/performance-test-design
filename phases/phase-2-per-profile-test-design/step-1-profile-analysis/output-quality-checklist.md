# Output Quality Checklist — Step 1: Profile Analysis (AI Gate)

- [ ] Every transaction has an explicit start-event → end-event boundary.
- [ ] Every transaction with a distinct external dependency, its own NFR, or a known common-failure-mode risk was considered for nested sub-transaction treatment.
- [ ] Every asynchronous hop has a corresponding transaction with an explicit completion-detection method.
- [ ] Persona Behavioral Detailing did not redefine the actor/persona already assigned in Phase 1.
- [ ] Every frequency number has a cited source (with time window and confidence level) or an explicit `[ASSUMPTION]` label.
- [ ] Any applied growth factor has a cited source; no growth factor was invented.
- [ ] The transaction mix percentages sum to 100% (or the rounding discrepancy is noted and negligible).
- [ ] Low-volume, high-importance transactions are explicitly flagged (UBP flag), not silently absorbed into the blended mix.
- [ ] The transaction mix was sanity-checked against Persona Behavioral Detailing's session-shape/conversion data.
- [ ] No content in this step's output assumes or references a specific load-testing tool.

If any item fails, fix it here before moving to Step 2.
