# Technique: Test Type Selection

**ISTQB CT-PT reference:** Module 1 performance test types catalog, applied within 4.2.4.

## What It Is

Performance testing is an umbrella covering several distinct test types, each answering a different question about the system. Choosing the right type(s) per transaction is a design decision — most profiles need more than one type applied to different transactions, not one type applied uniformly.

## The Test Types and When to Use Each

| Test Type                  | Question it answers                                                                                                      | Use when                                                                                                                                                | Don't use when                                                                                |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| **Load Testing**           | Does the system meet its NFRs under expected/peak volume?                                                                | Always — every profile needs at least this, using Step 1's mix and growth-adjusted volume.                                                              | Never skip.                                                                                   |
| **Stress Testing**         | Where is the breaking point, and how does the system fail past it?                                                       | Mission-critical system, or Step 1 flagged a high-UBP/low-margin transaction whose overload failure mode matters.                                       | Goal is only SLA compliance confirmation at known volume.                                     |
| **Spike Testing**          | How does the system handle a sudden, short traffic surge, and how fast does it recover?                                  | Flash sales, marketing campaigns, or a transaction Step 1's Persona Behavioral Detailing flagged as bursty.                                             | Steady, predictable-traffic profiles.                                                         |
| **Soak/Endurance Testing** | Does the system degrade over long-duration sustained use?                                                                | Always-on services, or a predecessor system with a history of long-running degradation.                                                                 | Short-lived batch jobs with no persistent process state.                                      |
| **Scalability Testing**    | How does response time/throughput change as load or data scales?                                                         | Capacity planning is a goal, or architecture uses auto-scaling whose behavior needs validation.                                                         | Fixed-size infrastructure with capacity planning out of scope this cycle.                     |
| **Volume Testing**         | Does the system handle large data volumes correctly and performantly (mostly database-centric)?                          | Batch data processing, large report generation, or a database expected to grow substantially.                                                           | No significant data-volume dimension in this profile.                                         |
| **Baseline Testing**       | What is this transaction's isolated performance, apart from the rest of the mix?                                         | Always useful as a first pass on a new or previously-untested transaction.                                                                              | Not a substitute for a full-mix Load Test — isolated numbers don't reveal contention effects. |
| **Capacity Testing**       | What is the maximum sustainable throughput this profile's transactions can support before violating acceptance criteria? | Business needs a defensible "how much headroom do we have" answer distinct from a pass/fail Load Test — e.g., for infrastructure procurement decisions. | The question is only pass/fail against a known target — that's Load Testing.                  |

## How to Apply

1. Start every profile with **Load Testing** as the default, non-optional selection, applied to the full transaction mix.
2. Walk through Step 1's UBP flags one by one — for each flagged transaction, ask which additional test type specifically addresses that flagged risk, and select it if so.
3. Check the NFR/SLA document for language implying a specific type even if not named as such: "must handle Nx normal traffic during promotional events" implies Spike; "must run unattended for N hours/days" implies Soak; "database must scale to Nx current volume" implies Volume/Scalability; "we need to know our maximum headroom before the next infrastructure budget cycle" implies Capacity.
4. For any newly-introduced or previously-untested transaction, consider a Baseline Test pass before folding it into the full-mix Load Test.
5. Document, for each selected type, which specific transaction(s) it targets — not every type needs to cover every transaction.

## Output

A test type selection table: type, targeted transaction(s), and the specific reason (NFR language or Step 1 UBP flag) that justified selecting it — plus an explicit note on any type considered but not selected.

## Example

| Test Type        | Targets                             | Reason                                                                                |
| ---------------- | ----------------------------------- | ------------------------------------------------------------------------------------- |
| Load Testing     | Full mix (all 6 transactions)       | Default — validates NFR compliance at growth-adjusted peak volume                     |
| Stress Testing   | Confirm Payment, Submit Order       | Step 1 UBP flags: 3rd-party dependency with hard timeout; known DB contention         |
| Spike Testing    | Browse Catalog, Search, Add to Cart | NFR-DOC-03 §6: "must sustain 3x normal traffic during flash-sale events"              |
| Soak Testing     | Full mix                            | Predecessor Order Service had documented memory-leak history                          |
| Baseline Testing | Confirm Payment (isolated)          | First isolation of 3rd-party gateway latency before combining into full-mix Load Test |

Scalability and Volume Testing: considered, **not selected** — infrastructure is fixed-size this release, and no batch/large-data-volume transaction exists in this profile.
