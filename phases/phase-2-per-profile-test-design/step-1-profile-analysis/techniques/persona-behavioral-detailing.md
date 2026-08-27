# Technique: Persona Behavioral Detailing

**ISTQB CT-PT reference:** 4.2.3 — Identifying Operational Profiles (behavioral depth component).

## What It Is

Phase 1 already decided _which_ actor(s) this profile represents (that decision belongs to Profile Boundary Definition and must not be re-litigated here). This technique goes one level deeper: it characterizes _how_ that actor actually behaves in a session, in enough behavioral detail to support Step 3's think-time and pacing calculations later. Skipping this and going straight to frequency numbers tends to produce a workload model with the right transaction list but an unrealistic session shape.

## When to Use

- After Transaction Identification, for the persona(s) already assigned to this profile in Phase 1.
- Whenever behavioral data (session analytics, device/channel splits) is available and hasn't yet been captured for this profile.

## When NOT to Use

- Do not use this technique to reconsider whether this profile's actor assignment is correct — if that seems wrong, it's a Phase 1 issue (raise it as a proposed change to `operational-profiles.md`, don't silently redefine it here).
- Not needed in the same depth for scheduled/batch profiles with no human actor (e.g., "System — Nightly Inventory Sync") — behavioral detailing here is largely replaced by the job's scheduling and data-volume characteristics instead.

## How to Apply

1. **Session shape:** typical session length, typical number of transactions per session, and whether sessions cluster (e.g., a burst of activity right after login) or spread evenly.
2. **Channel/device mix:** what proportion of this persona's traffic arrives via which channel (mobile web, native app, desktop web, partner API) — this affects both realistic think-time ranges (mobile users often pause differently than desktop users) and, if relevant, which load-generation approach makes sense downstream.
3. **Temporal pattern:** when this persona is actually active — time-of-day peaks, day-of-week patterns, seasonal effects — sourced from the same data used in Task Frequency Mapping where possible, so the two stay consistent with each other.
4. **Behavioral variability:** does this persona split into meaningfully different sub-behaviors worth noting even though they don't warrant a separate profile (e.g., "most Registered Customers browse then leave, a smaller fraction complete checkout") — this becomes directly relevant to Transaction Mix Design's conditional-transaction handling (see that technique).

## Output

A behavioral notes block for this profile's persona(s): session shape, channel/device mix, temporal pattern, and any noteworthy behavioral variability — each figure sourced or explicitly marked `[ASSUMPTION]`.

## Example

**Registered Shopper (PROF-01):**

- Session shape: average session ~12 minutes, 4–6 transactions per session (source: product analytics, 90-day sample)
- Channel/device mix: 68% mobile web, 32% desktop (same source)
- Temporal pattern: weekday peak 7–9pm local time; ~40% higher volume on the first weekend of each month (source: same analytics, cross-checked against marketing calendar for promotional cadence)
- Behavioral variability: of all sessions that reach the cart page, only ~35% proceed to "Confirm Payment" (cart abandonment) — this matters directly for Transaction Mix Design, since it means Checkout-adjacent transactions should not be assumed to occur as often as Browse-adjacent ones just because they're in the same profile
