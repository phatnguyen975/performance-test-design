# Step 1 — Profile Analysis

**ISTQB CT-PT reference:** 4.2.1 (Typical Communication Protocols), 4.2.2 (Transactions), 4.2.3 (Identifying Operational Profiles — per-profile depth).

## Purpose

Take the one Operational Profile approved in Phase 1 and analyze it to the depth Phase 2's later steps need: exact transaction boundaries, the protocols carrying them, a fully detailed behavioral picture of the profile's actor(s), precise frequency figures, and a finalized transaction mix. Phase 1 established _that_ this profile exists and roughly what it covers; this step establishes _exactly_ what's in it.

## Prerequisite

One profile ID, name, and scope statement from Phase 1's approved `operational-profiles.md`.

## Steps

1. **Protocol & System Analysis** — `techniques/protocol-and-system-analysis.md` → Map the protocols and system tiers this specific profile's transactions travel over — deeper than Phase 1's system-wide boundary summary.
2. **Transaction Identification** — `techniques/transaction-identification.md` → Define each transaction's precise start/end boundary, including nested sub-transactions.
3. **Persona Behavioral Detailing** — `techniques/persona-behavioral-detailing.md` → Go deeper on the actor(s) Phase 1 already assigned to this profile — session characteristics, device/channel mix, behavioral notes needed for later think-time and load-shape decisions. This step does not re-decide _which_ actor the profile represents; that was already settled in Phase 1.
4. **Task Frequency Mapping** — `techniques/task-frequency-mapping.md` → Attach a precise frequency (measured or estimated) to every transaction identified in Step 2.
5. **Transaction Mix Design** — `techniques/transaction-mix-design.md` → Convert frequencies into a normalized percentage mix, ready for Step 2 and Step 3.

## Supporting Material

- `resources/data-gathering-sources.md`
- `resources/common-failure-modes-reference.md`
- `output-template.md`
- `output-quality-checklist.md` — **run this before moving to Step 2**
- `best-practices.md`
- `anti-patterns.md`

## Output

Fill in `output-template.md` to produce `p1-profile-analysis.md`.

## Handing off to Step 2

Step 2 needs: the finalized transaction list with boundaries, the transaction mix (%), and any NFR-relevant risk flags raised here (e.g., a transaction with a hard external timeout, a known contention point).
