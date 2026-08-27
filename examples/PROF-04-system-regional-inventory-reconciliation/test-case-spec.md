# Performance Test Case Specification — System: Regional Inventory Reconciliation (PROF-04)

**Status:** Approved (example)
**Related step documents:** p1-profile-analysis.md · p2-load-profile.md · p3-workload-numbers.md · p4-test-data-specification.md

## 1. Test Case Header

- **Test Case ID:** PROF-04-TC
- **Objective:** Validate that nightly regional inventory reconciliation completes within its 3-hour window with an acceptable failure rate, under the real regional-overlap pattern and under a deliberately-stressed full-overlap scenario, given the documented shared-primary-DB lock-contention risk.
- **Test Type(s):** Volume (Inventory Reconciliation) · Load (realistic regional overlap) · Stress (forced full regional overlap)
- **Preconditions:**
  - Three synthetic WMS extract files (one per region), 15,000–60,000 SKU-level records each depending on the sub-test, with a deliberate ~1–2% injected discrepancy rate
  - Central Inventory DB test environment matching production's shared-primary-with-regional-read-replicas topology
  - Regional trigger schedule configured to match assumed real time zones (flagged as needing confirmation — see Section 9)

## 2. Scenario Flow Summary

Per region, in sequence: WMS Extract Ingestion → Inventory Reconciliation → Discrepancy Flagging. All three regions run on their own schedule; overlap between regions (not think time) is the shape-defining factor for this profile.

## 3. Load Profile Summary

| Test Type                | Shape                                                                                                                                          |
| ------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| Load (realistic overlap) | Simulate all 3 regions' actual local windows converted to UTC with real partial overlap; full 3-hour window observed per region                |
| Volume                   | Vary record count per region (15,000 / 40,000 / 60,000) as the independent variable; observe duration and failure rate                         |
| Stress (forced overlap)  | Force all 3 regions' windows to fully coincide (beyond today's real pattern). Stop: any region misses its window, or failure rate exceeds 0.5% |

## 4. Workload Numbers

- **Workload model:** Closed/scheduled (fixed, deterministic trigger — not an arrival-rate model)
- **Concurrency:** at most 2 regions' Reconciliation sub-transactions overlap concurrently under the assumed real time-zone configuration (never all 3) — see Section 9 for the time-zone assumption this depends on
- **Record volume:** 15,000–40,000 SKU-level records per region per night (estimated, unconfirmed)
- Think time / Little's Law: **not applicable** — no human actor, fixed schedule

## 5. Test Data Requirements

| Field                         | Required Volume                                    | Source                         | Notes                                                                      |
| ----------------------------- | -------------------------------------------------- | ------------------------------ | -------------------------------------------------------------------------- |
| WMS extract file (per region) | 3 files/run, 15,000–60,000 records each            | Synthetic, matching WMS schema | `[ASSUMPTION - no real sample extract available]`                          |
| `sku_id`                      | Shared with PROF-01/02's ≥3,000-SKU catalog sample | Same source                    | —                                                                          |
| Injected discrepancy rate     | ~1–2% of records                                   | Synthetic                      | Validates Discrepancy Flagging under realistic non-zero discrepancy volume |

## 6. Correlation Points

| Value                  | Extracted From           | Used In                  |
| ---------------------- | ------------------------ | ------------------------ |
| Staged extract dataset | WMS Extract Ingestion    | Inventory Reconciliation |
| Discrepancy record set | Inventory Reconciliation | Discrepancy Flagging     |

## 7. Acceptance Criteria

| Sub-transaction          | Criterion                                                               |
| ------------------------ | ----------------------------------------------------------------------- |
| Inventory Reconciliation | Complete within region's 3-hour window; ≤0.5% record-level failure rate |
| WMS Extract Ingestion    | No explicit NFR — flagged, see Section 9                                |
| Discrepancy Flagging     | Covered by the parent window/failure-rate constraint above              |

No response-time percentile targets apply to this profile (structural, not an omission).

## 8. Verification Points & Error Handling

- WMS Extract Ingestion: parsed record count matches file's stated count.
- Inventory Reconciliation: no records left pending at job end; DB record counts match expected post-state.
- Discrepancy Flagging: flagged-discrepancy count matches the deliberately-seeded discrepancy count.
- **Window compliance checked at the full per-region sequence level**, matching how the NFR is actually stated.
- A failure in one region's Ingestion must not block or corrupt the other regions' independent runs — verify this isolation explicitly.

## 9. Open Questions / Assumptions Requiring Confirmation

1. **Regional time zones are assumed** (Eastern/Central/Pacific) — README only states "three metropolitan regions"; the entire overlap analysis in Section 4 depends on this assumption and must be confirmed against GreenCart's actual regions before this design is finalized.
2. **Record volume per region (15,000–40,000) is estimated** — README §6 explicitly flags no dedicated analytics exist for this job; obtain real WMS export volumes before final execution.
3. No explicit NFR exists for WMS Extract Ingestion's acceptable latency once a file arrives.
4. Whether Discrepancy Flagging writes one batched record per region-night or one record per discrepancy is assumed (batched), not confirmed against FR-4.2's exact intent.
5. The specific discrepancy-tolerance threshold value (what counts as a "discrepancy" vs. normal variance) is not stated in the source NFR and needs confirmation.
