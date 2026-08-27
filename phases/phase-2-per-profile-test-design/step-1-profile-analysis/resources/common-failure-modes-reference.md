# Resource: Common Performance Failure Modes (Risk Awareness Reference)

Use while doing Transaction Identification and Transaction Mix Design so risk-relevant transactions aren't under-scoped just because they're low-volume. Performance testing exists specifically to catch failure modes that only appear under concurrent load or over time — not from a single-user functional pass.

## Failure Modes to Keep in Mind While Scoping Transactions

- **Resource exhaustion under sustained load** (memory leaks, connection pool exhaustion, thread starvation) — surfaces only after sustained concurrent usage. Relevant to flag for Step 2's Soak/Endurance test-type selection, not just a Load test.
- **Contention on shared resources** (database row/table locks, shared caches, rate-limited third-party APIs) — surfaces only under concurrency; exactly why a transaction like "Confirm Payment" deserves its own transaction boundary rather than folding into a parent average.
- **Cascading failure across tiers** — a slow downstream dependency causing timeouts/retries upstream, which increases upstream load further. Flag during Protocol & System Analysis when a hop has a hard timeout or retry policy.
- **Degradation that only appears at scale-out boundaries** — a service performing fine at 2 instances but exhibiting contention once auto-scaled to 10. Relevant to Step 2's Scalability Testing selection if the architecture uses auto-scaling.
- **Silent data corruption or partial failure under concurrent writes** — worth flagging for any transaction involving concurrent writes to the same record (e.g., inventory decrement during checkout).

## Why This Belongs in Step 1, Not Later

Identifying which transactions are exposed to these failure modes changes how they're scoped as transactions (own nested transaction so it can be measured independently?) and flags them as candidates for specific test types in Step 2 rather than being tested only as part of a generic Load Test. Missing this here means these risks are invisible by the time Step 2 selects test types.
