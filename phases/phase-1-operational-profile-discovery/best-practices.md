# Best Practices — Phase 1: Operational Profile Discovery

- **Read every available document before defining a single profile boundary.** Defining profiles from a partial reading risks a boundary decision that a later document would have contradicted.
- **Apply the "coffee-break test" when unsure if a flow is at the right granularity** — a flow the actor can walk away from cleanly once it's done is a good candidate for a distinct business event; a mid-flow interruption point usually isn't.
- **Log gaps explicitly rather than letting missing documentation silently shrink scope.** A missing NFR document should produce a visible gap entry, not a quietly-skipped profile.
- **Default to splitting, not merging, when a coherence criterion is ambiguous** — Phase 2 can still note a cross-reference between two closely related profiles, but merging two genuinely different populations corrupts every downstream calculation for both.
- **Check the discovered profile list against `resources/profile-types-reference.md` before finalizing** — customer-facing bias is a common, easy-to-miss gap.
- **Make the priority order's justification visible**, especially wherever it diverges from what raw usage volume alone would suggest — a reviewer should be able to see _why_ a low-volume profile was ranked highly, not just that it was.
- **Treat Human Review Gate 1 as a real checkpoint, not a formality.** The person reviewing the list often has context (an upcoming feature, a known pain point) that no document captured — build the review into the workflow with the expectation that changes are likely, not just possible.
