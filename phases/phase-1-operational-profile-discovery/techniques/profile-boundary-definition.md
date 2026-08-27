# Technique: Profile Boundary Definition

**Grounding:** ISTQB CT-PT 4.2.3 (Identifying Operational Profiles) combined with software reliability engineering's Operational Profile theory (Musa) — specifically its guidance that an operational profile represents a distinct set of tasks with their probability of occurrence for a _coherent_ population, which is the basis for deciding what does and doesn't belong in the same profile.

## What It Is

Taking the actor list, business-event list, and NFR list produced by System Document Analysis, and partitioning them into a set of discrete, non-overlapping **Operational Profiles** — each one scoped to a single, coherent user/system population performing a related set of tasks. This is a design decision, not a mechanical extraction: two technically-different actors might belong in the same profile if their behavior is genuinely similar, and one actor might need to be split across two profiles if their behavior is genuinely bimodal.

## When to Use

- Immediately after System Document Analysis, using its output as input.
- Whenever a new actor or business event is discovered later (e.g., a new module) and needs to be placed into an existing profile or spawn a new one.

## When NOT to Use

- Don't apply this technique to produce a single profile per individual use case — that over-fragments the design and produces many trivially small profiles that don't reflect how real usage actually clusters (e.g., "Browse" and "Search" are usually the same population behaving continuously, not two populations).
- Don't force-fit every actor into one of a fixed number of pre-decided categories — let the actual data (from System Document Analysis) drive how many profiles emerge.

## How to Apply

1. **Group by actor first, task second.** Start from the actor list. For each actor, list the business events/flows they're involved in. An actor with a materially different behavioral pattern for different subsets of their tasks (e.g., a Registered Customer who both browses/shops _and_ occasionally uses a loyalty-points redemption flow with very different volume and criticality) is a signal to consider splitting into two profiles rather than one.
2. **Apply the coherence test.** A single profile should satisfy: (a) one actor population (or a small number of actors whose combined behavior is genuinely similar), (b) tasks that a real instance of that population would plausibly perform in the same usage session or operational context, and (c) a shared order of magnitude in usage frequency and criticality. If any of these three break down, split into separate profiles.
3. **Handle system-to-system and scheduled/batch actors explicitly.** Not every profile has a human actor — secondary actors (external systems, integration partners) and scheduled/triggered events (nightly batch jobs, cron-triggered syncs) form their own profiles, scoped the same way, since they have their own distinct usage pattern (often closed-model, fixed-schedule, rather than open-model human-driven).
4. **Name and number each profile.** Assign a stable ID (`PROF-01`, `PROF-02`, ...) and a short, descriptive name following the pattern `{Actor} — {Primary Activity}` (e.g., "Registered Customer — Browse & Checkout"). This name is used verbatim through the rest of the design process.
5. **Write a one-line scope statement per profile**, explicitly stating what's included and — just as importantly — what's deliberately excluded (e.g., "Includes browsing, search, cart, and checkout. Excludes loyalty-points redemption, which is PROF-05.").
6. **Cross-check for coverage and overlap.** Every business event from System Document Analysis should map to exactly one profile — not zero (a missed flow) and not more than one (an ambiguous boundary that will corrupt Phase 2's transaction mix calculations for both profiles it was accidentally assigned to).

## Output

A profile-by-profile breakdown: ID, name, scope statement (included/excluded), the actor(s) it represents, and the business events/flows assigned to it — plus a coverage cross-check confirming every discovered business event lands in exactly one profile.

## Example

From the retail platform example in System Document Analysis:

| ID      | Name                                    | Actor(s)                                            | Scope                                                                                                                   |
| ------- | --------------------------------------- | --------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| PROF-01 | Registered Customer — Browse & Checkout | Registered Customer                                 | Browse, Search, Cart, Checkout. Excludes loyalty-points redemption (separate profile if that flow exists)               |
| PROF-02 | Guest Customer — Quick Checkout         | Guest Customer                                      | Browse, Cart, Checkout without account. Excludes any account-bound flow                                                 |
| PROF-03 | Fulfillment Staff — Order Approval      | Fulfillment Staff                                   | Review, approve, release shipments. Excludes inventory management (if that's a distinct staff role/task set)            |
| PROF-04 | System — Nightly Inventory Sync         | Inventory Sync Service (secondary actor, scheduled) | Scheduled reconciliation between warehouse system and platform inventory. No human actor; closed/fixed-schedule pattern |

**Why Registered and Guest Customer are separate profiles despite overlapping tasks:** their coherence differs on criterion (c) — Guest Customer sessions are shorter, have no account-bound history, and (per the NFR document) carry a different SLA tier than registered customers in this fictional platform's business rules. This is exactly the kind of judgment call this technique exists to make explicit and reviewable, rather than leaving it as an implicit assumption.
