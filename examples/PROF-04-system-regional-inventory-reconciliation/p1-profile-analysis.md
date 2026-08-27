# Profile Analysis — System: Regional Inventory Reconciliation (PROF-04)

## Context

- Profile scope (from Phase 1): Nightly stock reconciliation and discrepancy flagging (FR-4.1–4.2). Excludes the human morning-review step (out of scope per README §7).
- Actor(s) (from Phase 1): Regional Inventory Sync Service (secondary, scheduled/system-triggered — no human actor).

## 1. Protocol & System Analysis

| Hop                                                    | Protocol                                      | Sync/Async                                                                        |
| ------------------------------------------------------ | --------------------------------------------- | --------------------------------------------------------------------------------- |
| WMS (per region) → Regional Inventory Sync Service     | SFTP (CSV extract)                            | Async, batch file transfer — not a request/response pattern                       |
| Regional Inventory Sync Service → Central Inventory DB | JDBC (direct, legacy pattern per README §5.1) | Sync, but executed as part of a scheduled batch job, not a user-triggered request |

This is the first profile in this example set where no HTTPS/REST hop exists at all — the entire profile runs over file transfer and direct database access. This has a direct consequence for Step 3: no human think-time model applies here (see Persona Behavioral Detailing below).

## 2. Transaction Identification

**Parent transaction: "Nightly Reconciliation Run"** — Start: scheduled job trigger (01:00 local warehouse time). End: reconciliation complete and any discrepancies flagged, or job failure recorded.

| Sub-transaction                       | Boundary                                                                   | Sync/Async                                                                            | Why nested                                                                                                                                                                                |
| ------------------------------------- | -------------------------------------------------------------------------- | ------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| WMS Extract Ingestion                 | SFTP file becomes available → file parsed and staged                       | Async (file arrival is not on GreenCart's schedule, though the _processing_ of it is) | Distinct failure mode from the DB reconciliation step (a malformed or late-arriving extract vs. a DB-side contention issue)                                                               |
| Inventory Reconciliation (per region) | Staged extract data → central Inventory DB updated, discrepancies computed | Sync (within the batch job's execution)                                               | **Elevated risk** — README §5.2 documents historical lock contention on the shared primary DB when regional windows overlap in UTC; this is the step where that contention would manifest |
| Discrepancy Flagging                  | Reconciliation complete → discrepancy records written for morning review   | Sync                                                                                  | Distinct data-write step; low risk on its own, but depends on Reconciliation completing correctly first                                                                                   |

Three regions run this parent transaction on their own local 01:00–04:00 schedules — since the regions are in different time zones, their **UTC-time windows only partially overlap** (per §5.2), which is the direct cause of the documented lock-contention risk. This partial overlap is itself a key input to Step 2's load shape design (the shape isn't three identical, independent windows — it's three windows that partially stack).

## 3. Persona Behavioral Detailing

Not applicable in the human-behavioral sense — there is no human actor. In place of session shape/think-time/channel-mix, the equivalent structural facts for a scheduled/system profile are:

- **Trigger pattern:** Fixed schedule, once nightly per region, 01:00–04:00 local window (README §4.4.2, FR-4.1) — not think-time-paced, but window-bound.
- **"Volume" equivalent:** Number of SKU-level records reconciled per run, not a session count. `[ASSUMPTION - exact SKU/record count per region not stated in source document; README §6 explicitly flags this as a data gap — "no dedicated analytics currently exist for the Inventory Reconciliation batch job beyond basic job-duration and success/failure logging"]`.
- **Overlap pattern:** The three regions' local 01:00–04:00 windows, converted to UTC, partially overlap — this is a _structural_ fact from §5.2, not a behavioral estimate, and is the single most important input to this profile's Step 2 load shape.

## 4. Task Frequency Mapping

Given README §6's explicit gap flag (no dedicated analytics beyond job-duration/success logging), this section relies more heavily on stakeholder-estimation-style reasoning than any other profile in this example set — consistent with this technique's guidance for greenfield/undocumented cases.

| Sub-transaction          | Frequency                                                             | Basis                                                                                                                                                                                                                                                    |
| ------------------------ | --------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| WMS Extract Ingestion    | 1 per region per night = 3/night total                                | Directly implied by FR-4.1's "each night... for all three regions" — this is a structural fact, not an estimate                                                                                                                                          |
| Inventory Reconciliation | 1 per region per night = 3/night total                                | Same                                                                                                                                                                                                                                                     |
| Discrepancy Flagging     | 1 per region per night (batched, not per-discrepancy) = 3/night total | `[ASSUMPTION - FR-4.2 doesn't specify whether flagging is one batched write per run or one write per discrepancy record; assumed batched based on "flagged for morning review" reading as a single review artifact per region per night, not confirmed]` |

**Record-level volume** (the figure that actually drives Step 3's data-volume-sensitive design, distinct from the "3/night" run-count above): `[ASSUMPTION - no source data; estimated at 15,000–40,000 SKU-level records per region per night, based on a typical grocery regional-warehouse SKU count, needs confirmation against actual WMS export volumes]`.

## 5. Transaction Mix Design

Unlike the customer-facing profiles, this profile's "mix" is not a proportion of a shared traffic stream — all three sub-transactions occur exactly once per region per night, in a fixed sequence, not as independently-arriving traffic. A percentage-mix table isn't the right representation here; instead:

| Sub-transaction          | Occurrence                        | UBP Flag                                                                                                                   |
| ------------------------ | --------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| WMS Extract Ingestion    | 1x per region per night (3 total) | Low risk in isolation, but a late/malformed extract from one region can delay that region's downstream Reconciliation step |
| Inventory Reconciliation | 1x per region per night (3 total) | **Highest risk in this profile** — documented lock-contention history (§5.2), directly tied to release goal BG-4           |
| Discrepancy Flagging     | 1x per region per night (3 total) | Low risk, dependent on Reconciliation completing                                                                           |

**Why this deviates from the percentage-mix pattern used in customer-facing profiles:** Transaction Mix Design's own guidance notes it applies "whenever frequencies change" and is meant to represent proportional traffic share — a fixed, once-nightly, per-region sequence has no meaningful proportional-share interpretation, so this step's output is an occurrence table instead. This deviation is itself worth flagging explicitly rather than forcing an artificial percentage onto data that doesn't fit that shape.

## Step 1 AI Gate Self-Check Summary

Every sub-transaction has an explicit boundary; the async file-transfer nature of WMS Extract Ingestion is correctly distinguished from the synchronous-within-batch Reconciliation step; the lack of human actor is handled by substituting the structurally-equivalent facts (trigger pattern, record volume, overlap pattern) rather than forcing a human-behavioral template onto a system profile; the record-level volume gap from README §6 is carried forward explicitly; the deviation from percentage-mix representation is explained rather than silently forced. Proceeding to Step 2 with the record-volume assumption flagged prominently.
