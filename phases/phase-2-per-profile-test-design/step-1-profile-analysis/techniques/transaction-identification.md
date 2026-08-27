# Technique: Transaction Identification

**ISTQB CT-PT reference:** 4.2.2 — Transactions.

## What It Is

A transaction, per ISTQB CT-PT, is a unit of work with a clear beginning and end — the basic unit against which response time is measured. Transactions can be **nested**: a coarse-grained business transaction (e.g., "Complete Checkout") can contain several fine-grained transactions (e.g., "Apply Coupon," "Confirm Payment," "Submit Order"), each potentially needing its own response-time target.

This technique defines, explicitly, where each transaction of interest starts and ends. Everything downstream — response-time targets, correlation points, script structure — is built on top of this boundary decision.

## When to Use

- For every business flow within this profile, before assigning frequency or mix numbers.
- Whenever a flow has natural sub-steps that might individually violate an NFR even if the overall flow's average looks fine.
- For asynchronous hops identified in Protocol & System Analysis — these need a transaction definition based on completion detection, not a simple request/response pair.

## When NOT to Use

- Don't create a transaction for every technical HTTP call with no independent business or performance significance — nest it inside its parent transaction instead.
- Don't skip nested-transaction definition just because the parent flow "usually passes" — the parent average can hide a failing sub-step.

## How to Apply

1. List the candidate business flows within this profile (from Phase 1's assigned business events, refined with this profile's own detail).
2. For each flow, decide the transaction start event and end event (for async flows, the observable completion signal, not a single request/response pair).
3. Decide whether the flow needs nested sub-transactions. Rule of thumb: nest a sub-transaction when it (a) has its own NFR/SLA target, (b) involves a distinct external dependency whose latency should be isolated, or (c) is a known common failure point worth measuring independently.
4. Name each transaction clearly and consistently — used verbatim through every later step and the final Test Case Specification.
5. Flag any transaction spanning an asynchronous hop with an explicit note on how completion is detected (polling interval, callback, event subscription).

## Output

A transaction list for this profile: name, boundary (start → end event), parent/child relationship, and sync/async completion-detection flag.

## Example

**Parent transaction: "Complete Checkout"** — Start: click "Proceed to Checkout." End: order confirmation page fully rendered.

| Sub-transaction      | Boundary                                   | Sync/Async                     | Why nested                                 |
| -------------------- | ------------------------------------------ | ------------------------------ | ------------------------------------------ |
| Apply Coupon         | Click "Apply" → discount total re-rendered | Sync                           | Own NFR (≤500ms)                           |
| Confirm Payment      | Click "Pay" → gateway response received    | Sync, 3rd-party                | Isolate external latency; hard 10s timeout |
| Submit Order         | Payment confirmed → order record persisted | Sync                           | Known DB contention point                  |
| Fulfillment Dispatch | Order persisted → warehouse ack            | Async, polled 5s, max wait 60s | Different completion semantics             |

## Anti-Pattern to Avoid Here

Do not average "Complete Checkout" response time across its whole span and call it done — if "Confirm Payment" alone frequently exceeds a threshold due to the external gateway, that must be visible as its own measured sub-transaction, not buried inside a passing parent average.
