# Technique: NFR to Acceptance Criteria Mapping

**ISTQB CT-PT reference:** Module 1, Chapter 2.1 — Performance Metrics, applied within 4.2.4.

## What It Is

Converts NFR/SLA prose into specific, measurable, per-transaction thresholds using the standard performance metric catalog.

## The Standard Metric Catalog

| Metric                                          | What it measures                                                                                              | Typical threshold form    |
| ----------------------------------------------- | ------------------------------------------------------------------------------------------------------------- | ------------------------- |
| **Response Time Percentiles** (P50/P90/P95/P99) | Latency distribution. Percentiles beat averages because an average can hide outliers that percentiles reveal. | "P95 ≤ 800ms"             |
| **Throughput** (TPS/RPS)                        | Transactions/requests processed per second/minute/hour.                                                       | "≥ 1,200 TPS sustained"   |
| **Error Rate**                                  | % of failed transactions (5xx, timeouts, application-level failures).                                         | "≤ 0.5%"                  |
| **Resource Utilization**                        | CPU, memory, disk I/O, network — server-side.                                                                 | "CPU ≤ 75% sustained"     |
| **Concurrency**                                 | Simultaneous active sessions supported.                                                                       | "500 concurrent sessions" |

## When to Use

For every transaction with an explicit or implicit NFR — not just the system as a whole.

## When NOT to Use

Don't invent a threshold for a transaction the NFR document simply doesn't mention — flag the gap explicitly.

## How to Apply

1. For each transaction, search the NFR document for any explicit threshold tied to it or its parent flow.
2. Where the NFR is stated at flow/system level but not per-transaction, decide whether to apply it uniformly or differentiate based on Step 1's UBP flags.
3. Where no NFR exists for a transaction, state this explicitly rather than substituting an "industry typical" number silently.
4. Always express response-time targets as percentiles (P95 minimum), never a bare average — flag it if the source document only states an average.
5. Cross-reference any hard technical constraint from Step 1's Protocol & System Analysis — an acceptance criterion cannot be looser than a hard downstream constraint, and should usually leave margin.

## Output

A per-transaction acceptance criteria table: P95, P99, error rate, throughput/concurrency where relevant, and the NFR source or flagged gap.

## Example

| Transaction                  | P95                   | P99                                               | Error Rate              | Source                                                                                                |
| ---------------------------- | --------------------- | ------------------------------------------------- | ----------------------- | ----------------------------------------------------------------------------------------------------- |
| Browse Catalog               | ≤500ms                | ≤1200ms                                           | ≤0.5%                   | NFR-DOC-03 §4.1                                                                                       |
| Confirm Payment              | ≤3000ms               | ≤8000ms (margin under gateway's hard 10s timeout) | ≤0.1%                   | NFR-DOC-03 §4.3, cross-ref Step 1 gateway constraint                                                  |
| Fulfillment Dispatch (async) | N/A — completion ≤60s | —                                                 | ≤1.0% dispatch failures | `[ASSUMPTION - no explicit NFR, derived from Step 1's polling max-wait, needs business confirmation]` |
