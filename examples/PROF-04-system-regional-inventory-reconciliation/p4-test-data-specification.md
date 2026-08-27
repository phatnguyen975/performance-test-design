# Test Data Specification — System: Regional Inventory Reconciliation (PROF-04)

## 1. Data Parameterization Specification

| Field                             | Type/Format                                 | Required Volume                                                                                                                                      | Source                                                                                                                                                                                                   | Reuse Policy                                                                         | Uniqueness Constraint |
| --------------------------------- | ------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------ | --------------------- |
| WMS extract file (per region)     | CSV, matching WMS export schema             | 3 files per test run (one per region), each containing the assumed 15,000–40,000 SKU-level records                                                   | `[ASSUMPTION - synthetic generation matching WMS export schema, since no sample extract was available in the source document; a real engagement should obtain an actual desensitized WMS export sample]` | Fresh-generated per test run (not reused — simulates a new nightly extract)          | No                    |
| `sku_id` (within extract records) | Existing catalog SKU                        | Matches the ≥3,000-SKU catalog sample already established for PROF-01/PROF-02 (shared reference data across profiles, not regenerated independently) | Same source as PROF-01's `product_id` field                                                                                                                                                              | Reused across records within an extract (a SKU can appear once per region's extract) | No                    |
| Discrepancy tolerance threshold   | Configuration value, not per-iteration data | Single value, environment-level                                                                                                                      | `[ASSUMPTION - README §4.4.2 doesn't state the specific tolerance value used to decide what counts as a "discrepancy," only the overall failure-rate NFR; needs confirmation]`                           | N/A — environment config, not parameterized data                                     | No                    |

This profile's data needs are structurally different from the customer-facing profiles: there is no per-virtual-user pool to cycle through, since there's no concurrent human population — there are three fixed, region-scoped input files per test run.

## 2. Correlation Mapping Specification

**Not applicable in the usual multi-step-transaction sense.** This profile's three sub-transactions (Ingestion → Reconciliation → Flagging) do pass data forward, but as a batch data pipeline, not as request/response correlation between HTTP calls:

| Value                  | Source Sub-transaction                                    | Destination Sub-transaction                                                      | Lifetime                                                                                     |
| ---------------------- | --------------------------------------------------------- | -------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| Staged extract dataset | WMS Extract Ingestion (output: parsed, staged records)    | Inventory Reconciliation (input: staged records to reconcile against central DB) | Duration of the batch run                                                                    |
| Discrepancy record set | Inventory Reconciliation (output: computed discrepancies) | Discrepancy Flagging (input: records to write for morning review)                | Duration of the batch run, persisted afterward for the (out-of-scope) morning review process |

## 3. Data Diversity Rules

**Record volume, not value diversity, is the dominant concern for this profile** — unlike the customer-facing profiles where cache-hit-rate distortion from a too-small `product_id` pool was the main risk, this profile's main data-realism risk is testing against too _few_ records and understating how long Reconciliation takes, or how much lock contention it causes, at real production volume.

- **Volume range to test:** the full 15,000–60,000 record-per-region range established in Step 2's Volume Testing shape — not a single fixed size.
- **Discrepancy rate within synthetic data:** `[ASSUMPTION - no real discrepancy-rate data available; synthetic extracts should inject a deliberate discrepancy rate of ~1–2% of records, higher than the §4.4.2 0.5% failure tolerance, specifically to verify Discrepancy Flagging behaves correctly under a realistic (not zero) discrepancy volume, not just a clean-data happy path]`.

## 4. Script Blueprint Specification

### Initialization

Generate or place the three regions' WMS extract files at the test environment's SFTP location, timed according to the load shape being tested (realistic assumed overlap, or Stress Testing's forced full-overlap scenario). No per-VU authentication applies (no human actor).

### Main Flow

Per region, in the schedule determined by Step 2's load shape:

1. WMS Extract Ingestion — trigger occurs when the file becomes available at the SFTP location (or, for Stress Testing's forced-overlap scenario, all three files placed simultaneously)
2. Inventory Reconciliation — begins once ingestion completes for that region; writes to the shared central Inventory DB
3. Discrepancy Flagging — begins once reconciliation completes for that region

### Verification Points

- WMS Extract Ingestion: file successfully parsed, record count matches the file's stated record count (no silent truncation).
- Inventory Reconciliation: all staged records processed (no records left in a pending state at job end); central DB record counts match expected post-reconciliation state.
- Discrepancy Flagging: discrepancy record count in the output matches the count of records that were deliberately seeded as discrepancies in the synthetic extract (validates the flagging logic itself is catching what it should, not just that the job "completed").
- **Window compliance (cross-cutting):** the entire per-region sequence (Ingestion → Reconciliation → Flagging) completes within that region's 3-hour window — this is checked at the sequence level, not per sub-transaction, matching how §4.4.2's NFR is actually stated.

### Error Handling

A failure in Ingestion for one region should not block or corrupt the other two regions' independent runs (regions are logically separate, sharing only the primary DB as a resource, not a shared execution path) — the script must confirm this isolation holds, not just assume it. A failure in Reconciliation should be logged with enough detail (which records failed, at what point) to distinguish a data-quality issue (malformed extract) from a system-capacity issue (DB contention/timeout) — since Step 2's Stress Testing exists specifically to find the latter.

### Cleanup/Termination

Clear synthetic extract files and any test-injected discrepancy records from the environment after each run, to avoid the next run's Ingestion step encountering stale leftover files.

## Step 4 AI Gate Self-Check Summary

This profile's data-spec correctly reflects its batch nature throughout — parameterization is framed around file-level and record-volume concerns rather than per-VU pools; correlation is framed as pipeline data-passing rather than HTTP request/response correlation; diversity rules correctly prioritize volume-range testing over value-distribution weighting, unlike the customer-facing profiles; verification points include the cross-cutting window-compliance check that directly reflects how §4.4.2's NFR is actually worded. No tool-specific syntax present. Proceeding to final Test Case Specification, then Human Review Gate 2.
