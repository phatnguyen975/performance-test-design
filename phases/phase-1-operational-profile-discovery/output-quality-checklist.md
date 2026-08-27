# Output Quality Checklist — Phase 1 (AI Gate, before Human Review Gate 1)

- [ ] Every available system document was actually read and reflected in the System Document Analysis summary — none silently skipped.
- [ ] Every document type listed in root `SKILL.md`'s Input table that was NOT available is explicitly stated as absent, not just omitted from the output.
- [ ] Every actor is classified as primary or secondary.
- [ ] Every business event/flow traces to a specific source document.
- [ ] Every NFR is tagged with the actor/flow it applies to, or flagged as system-wide/vague if it can't be tagged precisely.
- [ ] The gap log explicitly lists anything a complete discovery would need but wasn't available in the provided documents.
- [ ] Every profile satisfies the three-part coherence test (one population, plausibly-shared usage context, shared order-of-magnitude frequency/criticality) — none were merged or split without checking all three.
- [ ] Every profile has a scope statement that states both what's included AND what's explicitly excluded.
- [ ] The coverage cross-check confirms every business event maps to exactly one profile — any exceptions found are explicitly resolved, not left ambiguous.
- [ ] The discovered profile list was checked against `resources/profile-types-reference.md` for category completeness (customer-facing, backoffice, system-to-system, batch/scheduled, event-driven).
- [ ] Every profile has a priority order with a justification shown against all four factors (business criticality, risk exposure, volume, dependency/urgency) — not just volume.
- [ ] No content in this phase's output assumes or references a specific load-testing tool.

If any item fails, fix it here before presenting `operational-profiles.md` to Human Review Gate 1.
