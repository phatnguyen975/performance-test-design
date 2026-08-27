# Best Practices — Step 1: Profile Analysis

- Cite the exact data source, time window, and confidence level for every frequency number — a HAR-sample-derived estimate and a full-APM measured count are not the same strength of evidence and should be labeled differently.
- Name transactions the way an implementer will recognize them — this name is used verbatim through Step 4's script blueprint and the final Test Case Specification.
- Decide nested transaction boundaries around what has its own NFR or its own external dependency, not around arbitrary technical granularity.
- Never re-decide the persona/actor assignment in Persona Behavioral Detailing — that belongs to Phase 1; if it seems wrong, raise it as a proposed change to `operational-profiles.md` instead of silently overriding it here.
- Apply a growth factor only when it has a cited source (NFR document, business case) — never invent one.
- Cross-check the transaction mix against Persona Behavioral Detailing's session-shape data (e.g., conversion rates) before finalizing — an inconsistency here usually means one of the two techniques used a different population baseline than the other.
- Flag low-volume, high-importance transactions explicitly via the UBP flag so Step 2 doesn't silently under-test them just because a blended Load Test wouldn't naturally stress them.
- Treat "no data available" as a valid, explicit output — state it, label the resulting number as an assumption, and move on.
