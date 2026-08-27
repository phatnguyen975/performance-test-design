# Anti-Patterns — Phase 1: Operational Profile Discovery

- **Starting Profile Boundary Definition before System Document Analysis is complete.** Partial extraction produces boundaries that a later-read document would have changed.
- **Defining one profile per use case instead of per coherent population.** This over-fragments the list into dozens of trivial profiles that don't reflect how real usage actually clusters together.
- **Merging two actors into one profile because their tasks overlap on paper**, without checking whether their actual behavioral pattern, volume, or criticality genuinely coincide — see Profile Boundary Definition's Registered-vs-Guest example.
- **Silently excluding administrative, batch, or system-to-system profiles** because the FRD emphasized customer-facing flows — check `resources/profile-types-reference.md` explicitly rather than trusting document emphasis to reflect true scope.
- **Prioritizing purely by usage volume.** A low-volume, high-criticality or high-risk profile can matter more than a high-volume, low-impact one — see Profile Prioritization's fulfillment-approval example.
- **Treating an absent NFR document as "no NFR constraint exists"** rather than as a gap requiring stakeholder follow-up before Phase 2 proceeds too far to easily incorporate it.
- **Skipping the coverage cross-check** ("does every business event map to exactly one profile?"). An event mapped to zero profiles is a missed test; an event mapped to two is a boundary error that will corrupt both profiles' transaction mix later.
- **Proceeding into Phase 2 without Human Review Gate 1 approval** because the list "looks obviously right" — the review exists specifically to catch what isn't obvious to whoever produced the list.
