# Technique: System Document Analysis

**Grounding:** Requirements-analysis practice from business analysis discipline (BABOK® Guide technique 10.30 "Non-Functional Requirements Analysis" and 10.47 "Use Cases and Scenarios"; the "user goal technique" and "event decomposition technique" for actor/use-case identification), combined with ISTQB CT-PT's protocol/transaction concepts (4.2.1–4.2.2) applied at system-discovery scope rather than per-transaction scope.

## What It Is

The systematic first pass through every available system document, done **once**, to extract everything Profile Boundary Definition (the next technique) will need: who interacts with the system, what they do, what constraints apply, and what the system's structural boundaries are. This is a reading-and-extraction technique, not yet a design technique — its job is to surface facts, not to decide how to group them into profiles.

## When to Use

- Always, as the first step of Phase 1, regardless of how much or how little documentation exists.
- Whenever new documentation becomes available for a system already partially profiled (e.g., a new module's FRD is added) — re-run this technique for the new material and check whether it changes the existing profile list.

## When NOT to Use

- Not a substitute for stakeholder interviews when documentation is sparse or contradictory — use it to extract what documents _do_ say, and explicitly list what they don't, rather than inferring facts the documents don't support.

## How to Apply

1. **Actor identification (the "user goal technique"):** For every document, extract every actor — a person, role, or external system that interacts with the system under test. BABOK distinguishes primary actors (gain direct benefit from an interaction) from secondary actors (participate but don't directly benefit, e.g., a notification service) — both matter for performance testing, since a secondary actor's system-to-system traffic still consumes capacity. Classify each actor by functional role and, where relevant, organizational level (operational/management/executive) since this often correlates with usage volume and pattern.
2. **Business-event / flow identification (the "event decomposition technique"):** Ask, for each actor, "what business events occur that require the system to respond?" Each business event traces to a use case or user story — this is the systematic way to avoid missing a flow, rather than relying on whatever flows happen to be top-of-mind. Apply the "coffee-break test" (Cockburn): a candidate flow is at the right level of granularity if, once it's complete, the actor could take a break without leaving the system in an inconsistent state (i.e., it corresponds to one elementary business process, not a sub-step of one, and not a bundle of several).
3. **NFR/constraint extraction:** Separately from functional flows (since NFRs are typically documented apart from use cases per BABOK guidance), extract every non-functional requirement: response-time targets, throughput/concurrency targets, availability windows, data-volume constraints. Tag each with which actor/flow it applies to, where stated; flag NFRs stated only at a system-wide level for later resolution in Phase 2 Step 2.
4. **System boundary extraction:** From architecture/design docs, note the protocols, tiers, and integration points involved (this doesn't need to be as detailed as Phase 2 Step 1's own Protocol & System Analysis — that will go deeper per-profile — but a first-pass system-wide map here helps Profile Boundary Definition recognize when two candidate profiles actually share, or don't share, underlying infrastructure).
5. **Gap logging:** For anything a complete discovery would need but isn't documented anywhere available (e.g., no NFR document exists at all, or use cases exist only for the customer-facing module but not the admin module), log this explicitly as a gap — don't let an absence of documentation silently shrink the scope of what gets discovered.

## Output

A working extraction document (not the final profile list yet) containing: actor list (with primary/secondary classification), business-event/flow list (each traced to its source document), NFR list (tagged by actor/flow where possible), a system boundary summary, and a gap log. This feeds directly into Profile Boundary Definition.

## Example

From a fictional retail platform's FRD, Use Case doc, and NFR doc:

**Actors:** Registered Customer (primary), Guest Customer (primary), Fulfillment Staff (primary), Inventory Sync Service (secondary, external system), Payment Gateway (secondary, external system).

**Business events (sample):** "Customer wants to find a product" → Browse/Search use cases. "Customer wants to purchase" → Cart/Checkout use cases. "Fulfillment staff needs to release a shipment" → Order Approval use case. "Nightly inventory count must reconcile with warehouse system" → Inventory Sync event (no human actor — a scheduled/system-triggered event).

**NFR sample:** "Checkout must complete within acceptable time even at Black Friday volumes" (tagged: Registered/Guest Customer, Checkout flow) — vague on exact numbers, logged as a gap requiring stakeholder follow-up in a later phase, not invented here.

**Gap log sample:** "No NFR document section addresses the Inventory Sync Service's performance requirements — flagged for Phase 2 Step 2 to resolve directly with the systems team, since no document currently states one."
