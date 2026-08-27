# Technique: Open vs. Closed Model Selection

**ISTQB CT-PT reference:** 4.2.5 — Analyzing Throughput and Concurrency.

## What It Is

A workload model describes how new work arrives relative to how quickly the system processes existing work.

**Closed Workload Model:** A fixed number of virtual users repeatedly execute a workflow; a new iteration for any given VU starts only after its previous iteration completes. Total concurrent load is self-limiting.

**Open Workload Model:** New arrivals occur at a rate independent of how quickly the system processes existing work — like real internet traffic. If the system slows down, in-flight requests grow, because arrivals keep coming regardless.

## The Coordinated Omission Problem

In a closed model, when the system under test becomes slow, the virtual user pool's _effective_ arrival rate drops automatically (each VU is busy waiting on a slow response). This silently reduces offered load exactly when the system is struggling — making percentile metrics look better than they would under real, independent-arrival traffic. This is known as **coordinated omission**, a well-documented distortion associated with closed-model load generation under degraded conditions.

## When to Use Closed Model

- Systems where a genuinely fixed population interacts in a look-then-act loop and cannot generate new requests while waiting (call-center agents, internal batch schedulers, ticketing platforms with a hard concurrent-agent cap).
- Regression/benchmark testing where reproducibility matters more than perfect real-world fidelity, and the population genuinely behaves this way.

## When to Use Open Model

- Public-facing web/API systems where users/client systems arrive independently of response speed — the large majority of customer-facing and public-API profiles.
- Any test where accurately capturing degradation under load matters, since only an open model avoids coordinated omission's optimistic bias.

## How to Apply

1. For each transaction (or the whole profile if uniform), determine: does a new instance get triggered by an external, independent event, or only because a specific, capacity-limited pool of actors decided to do it next?
2. Default to open unless there's a specific, defensible reason the population is genuinely capacity-limited.
3. State the choice and justification per transaction — check whether any transaction (e.g., an internal admin task performed by a small, fixed team) behaves differently from the rest.
4. If a downstream tool only supports closed natively, note this as an implementation constraint for Step 4's script blueprint — it doesn't change the model decision made here, but does mean the eventual script must be built carefully (e.g., arrival-rate-based scripting pattern) to avoid coordinated omission.

## Output

A per-transaction (or per-profile, if uniform) workload model decision with justification.

## Example

| Transaction                                                        | Model                                     | Justification                                                                                                                     |
| ------------------------------------------------------------------ | ----------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| Browse Catalog, Search, Add to Cart, Confirm Payment, Submit Order | Open                                      | Public retail traffic arrives independently of response speed; closed model would understate degradation via coordinated omission |
| Fulfillment Dispatch (backend consumer)                            | Closed (bounded by a fixed consumer pool) | Fixed, configured number of worker processes — genuinely capacity-limited, not independently-arriving traffic                     |
