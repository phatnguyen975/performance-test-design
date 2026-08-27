# Technique: Protocol & System Analysis

**ISTQB CT-PT reference:** 4.2.1 — Typical Communication Protocols.

## What It Is

Before any transaction can be identified or measured, you need to know what communication protocol(s) carry it and which system tiers it passes through. ISTQB CT-PT lists the protocol families a performance tester routinely encounters: HTTP/HTTPS and REST/SOAP web services, database access protocols (JDBC/ODBC), messaging protocols (JMS/MQ/AMQP), directory protocols (LDAP), and terminal/legacy protocols (e.g. mainframe 3270, Citrix ICA) for legacy enterprise systems.

Why this matters specifically for test design: the protocol determines **where a transaction boundary can be observed and measured**. An HTTP request/response has an obvious start and end. A multi-step workflow spanning a browser session, an API call, and an asynchronous backend job does not — you have to decide, deliberately, where the "transaction" starts and stops before a response-time target means anything.

This step goes deeper than Phase 1's system-wide boundary summary — it maps protocols at the granularity of _this specific profile's_ transactions, not the whole system.

## When to Use

- At the start of Step 1, before attempting to list this profile's transactions in detail.
- Whenever this profile involves multiple tiers/protocols (e.g., web front end → REST API → message queue → database) — every hop needs clarity.
- When this profile's actor can reach the same business action via more than one channel (web UI, mobile app, partner API) — each channel may need separate transaction identity even if the business action is nominally "the same."

## When NOT to Use

- Not needed as a repeated step for every single transaction — do it once per profile, then reuse the understanding for every transaction identified in the next technique.
- Not a substitute for full architecture review — this extracts only the performance-relevant protocol facts, at the depth this profile needs.

## How to Apply

1. List every system tier this profile's actor interacts with, directly or indirectly (client, load balancer, application server, cache, database, external/third-party services, message queues, batch jobs) — using Phase 1's system-wide boundary summary as a starting point, narrowed to what's actually relevant to this profile.
2. For each tier-to-tier hop relevant to this profile, name the protocol and note sync vs. async behavior.
3. Note protocol-specific performance-relevant behavior: connection pooling limits, keep-alive/session behavior, TLS handshake overhead, protocol-level retries or timeouts already configured.
4. Carry forward, explicitly, any hard technical constraint (a fixed timeout, a rate limit) — this becomes an input to Step 2's acceptance-criteria mapping.

## Output

A protocol map scoped to this profile — a table listing each relevant hop, its protocol, and sync/async nature — referenced (not repeated) throughout the rest of Step 1 and later steps.

## Example

| Hop                                         | Protocol              | Sync/Async                            |
| ------------------------------------------- | --------------------- | ------------------------------------- |
| Browser → Web App                           | HTTPS/REST            | Sync                                  |
| Web App → Product Catalog Service           | HTTPS/REST (internal) | Sync                                  |
| Web App → Order Service                     | HTTPS/REST (internal) | Sync                                  |
| Order Service → Payment Gateway (3rd party) | HTTPS/REST            | Sync, 10s hard timeout                |
| Order Service → Fulfillment Queue           | AMQP                  | Async, consumer polls, ~3s processing |
| Order Service → Order DB                    | JDBC                  | Sync                                  |

This immediately flags two facts relevant later: the Payment Gateway hop has a hard 10s timeout (an input to Step 2's acceptance criteria), and the Fulfillment Queue hop is asynchronous (it needs its own transaction boundary with a different completion definition, addressed in the next technique).
