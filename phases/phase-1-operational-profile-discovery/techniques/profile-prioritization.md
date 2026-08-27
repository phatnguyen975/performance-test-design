# Technique: Profile Prioritization

**Grounding:** Risk-based testing principles (ISTQB Foundation Level's risk-based test prioritization, applied here to performance-specific risk factors) combined with BABOK's Prioritization technique (10.33) for weighing business value against implementation/testing effort and urgency.

## What It Is

Once Profile Boundary Definition produces a set of discrete profiles, this technique orders them — deciding which profile Phase 2 tackles first, second, and so on. Order matters in practice: teams rarely have unlimited time to design and execute every profile before a release, and a defensible, explicit priority order means the most important testing happens first even if time runs out before the list is exhausted.

## When to Use

- Always, as the final step of Phase 1, applied to the complete profile list from Profile Boundary Definition.
- Whenever the profile list changes (a profile added, split, or merged) — re-prioritize rather than just appending the new profile to the end by default.

## When NOT to Use

- Don't use raw transaction volume alone as the sole prioritization factor — a low-volume, high-business-impact profile (e.g., a payment-adjacent admin flow) can matter more than a high-volume, low-impact one (e.g., a static content page).

## How to Apply

Score each profile against these factors, then combine into an overall priority order (a simple high/medium/low rating per factor, combined qualitatively, is usually sufficient — a numeric weighted score is optional for larger profile sets where ties need breaking):

1. **Business criticality.** Does this profile touch revenue, legal/compliance obligations, or core value proposition? A checkout flow outranks a "view my order history" flow even if both are well-documented.
2. **Risk exposure.** Does this profile involve a known-fragile component, a third-party dependency, a recent major change, or a documented history of past performance incidents (in this system or a predecessor)? Higher risk exposure raises priority even if business criticality is moderate.
3. **Usage volume/frequency.** How often does this profile's activity actually occur, relative to the others? (This is often only roughly knowable at this stage — Phase 2 Step 1's Task Frequency Mapping will get the precise figures; Phase 1 only needs enough to rank profiles relative to each other.)
4. **Dependency ordering.** Does designing one profile logically require understanding gained from another? For example, if a "System — Nightly Batch Sync" profile's behavior materially affects what "Registered Customer" transactions see (e.g., inventory availability), understanding the batch profile first can inform assumptions in the customer profile's design — note such dependencies explicitly, even if the final priority order doesn't strictly follow them.
5. **Release/deadline urgency.** Is this profile tied to a specific upcoming release or contractual deadline? This can override the other factors for practical scheduling reasons — note it as a distinct factor rather than silently inflating "business criticality" to justify a deadline-driven reordering.

## Output

The complete profile list from Profile Boundary Definition, now ordered, with each profile's priority rating shown against the factors above and a one-line justification for its position — especially for any profile whose priority isn't obviously implied by its raw usage volume.

## Example

| Order | ID      | Name                                    | Business Criticality                                 | Risk Exposure                                                | Volume                             | Priority Justification                                      |
| ----- | ------- | --------------------------------------- | ---------------------------------------------------- | ------------------------------------------------------------ | ---------------------------------- | ----------------------------------------------------------- |
| 1     | PROF-01 | Registered Customer — Browse & Checkout | High (primary revenue path)                          | Medium                                                       | High                               | Highest combined score; default first                       |
| 2     | PROF-03 | Fulfillment Staff — Order Approval      | High (blocks revenue realization if slow)            | High (recent rework of approval service, no prior perf test) | Low                                | Recent change + high criticality outrank its low volume     |
| 3     | PROF-02 | Guest Customer — Quick Checkout         | Medium (smaller share of revenue than PROF-01)       | Medium                                                       | Medium                             | Similar transactions to PROF-01 but lower business share    |
| 4     | PROF-04 | System — Nightly Inventory Sync         | Medium (affects data accuracy, not directly revenue) | Low (stable, unchanged component)                            | N/A (scheduled, not volume-driven) | Lowest urgency; scheduled job with no recent change history |

Note that PROF-03 outranks PROF-02 despite lower usage volume — its recent rework and lack of any prior performance testing raise its risk exposure enough to justify testing it earlier, illustrating why volume alone should never be the sole ordering factor.
