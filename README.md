<div align="center">
  <h1>Performance Test Design Skill</h1>
  <small>
    <strong>Author:</strong> Nguyễn Tấn Phát
  </small> <br />
  <sub>July 12, 2026</sub>
</div>

An Agent skill that takes a system's requirements documentation and turns it into a complete set of **implementer-ready Performance Test Case Specifications** — one per Operational Profile the system needs tested.

The skill covers the full design process: discovering which profiles exist, analyzing each profile's transactions and load behavior, calculating virtual users and throughput with documented reasoning, and specifying test data requirements. It stops at the specification. It never writes JMeter, k6, Gatling, or LoadRunner code — that remains a separate implementation step.

## What it does

Given system documentation (BRD, FRD, use cases, NFR/SLA docs, architecture docs, and optionally production APM data), the skill runs a two-phase process:

**Phase 1 — Operational Profile Discovery:** Reads all available system documents once, identifies user roles and business flows, and produces a prioritized list of Operational Profiles the system needs performance-tested. Pauses for human review before any design begins.

**Phase 2 — Per-Profile Test Case Design:** For each approved profile, in priority order, runs four dependent steps and pauses for human review after each profile completes before moving to the next:

| Step                     | Output file                                           | What it produces                                                                                          |
| ------------------------ | ----------------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| Profile Analysis         | `p1-profile-analysis.md`                              | Transaction boundaries, protocols, persona behavior, frequency data, transaction mix                      |
| Load Profile Design      | `p2-load-profile.md`                                  | Test type selection (Load/Stress/Spike/Soak/Scalability/Volume/Baseline), load shape, acceptance criteria |
| Throughput & Concurrency | `p3-workload-numbers.md`                              | Open/closed model choice, think time, Little's Law VU calculation, throughput reconciliation              |
| Test Case Specification  | `p4-test-data-specification.md` + `test-case-spec.md` | Data parameterization, correlation mapping, data diversity rules, script blueprint, final condensed spec  |

Every number in every output is either sourced (with a citation) or explicitly labeled `[ASSUMPTION]` — the skill never invents a figure without flagging it.

## What it is not

- It does not write test scripts. Output is tool-agnostic design documentation.
- It does not execute tests or analyze post-execution results.
- It does not do functional test design (equivalence partitioning, BVA, etc.).
- It does not do infrastructure sizing or capacity procurement planning.

## Setup

1. Clone or download this repository.
2. Copy the entire `performance-test-design/` into your AI's `skills/` directory.

## Usage

### Default mode — output to conversation

Type the invoke command and attach (or paste) your system documentation:

```
/performance-test-design
```

The skill will:

1. Run Phase 1 — print `operational-profiles.md` content to the conversation, then **pause and ask you to confirm the profile list**.
2. On your approval, begin Phase 2 for Profile #1 — print each step's output as it's completed, running an AI self-review gate between steps.
3. After all four steps for Profile #1 complete, **pause and ask you to confirm** that profile's design. You can request corrections at this point; the skill will fix the specific step(s) that need it and re-run only the steps downstream of the fix.
4. On your approval, move to Profile #2 and repeat until all profiles are done.

### File mode — write output to disk

```
/performance-test-design --file=docs/perf-test-profiles/
```

The path after `--file=` is a **directory**. The skill writes all outputs there:

```
docs/perf-test-profiles/
├── operational-profiles.md           ← Phase 1: approved profile list
├── PROF-01-{short-name}/
│   ├── p1-profile-analysis.md
│   ├── p2-load-profile.md
│   ├── p3-workload-numbers.md
│   ├── p4-test-data-specification.md
│   └── test-case-spec.md             ← final condensed spec (implementer-facing)
├── PROF-02-{short-name}/
│   └── [same 5 files]
└── ...
```

In file mode, a short progress summary is still printed to the conversation after each step so you can follow along without opening files. Review gates work the same way — the skill still pauses and asks for your approval at Phase 1 and after each completed profile.

## Inputs

Provide any combination of the following — the more that's available, the fewer assumptions the skill needs to make:

| Document                                     | Used for                                                                                   |
| -------------------------------------------- | ------------------------------------------------------------------------------------------ |
| Business Requirements (BRD)                  | Identifying business-critical flows and their relative priority                            |
| Functional Requirements (FRD) / User Stories | Enumerating transactions and user roles                                                    |
| Use Case documents                           | Mapping actors to business goals (feeds directly into Operational Profile discovery)       |
| NFR / SLA documents                          | Response time targets, throughput/concurrency targets, error rate limits                   |
| System architecture / design docs            | Protocols, tiers, sync/async boundaries, known constraints                                 |
| Production logs / APM data                   | Real transaction frequencies and session behavior (strongest source for workload modeling) |

Missing document types don't block the skill — they result in `[ASSUMPTION]` labels on the affected figures, which are consolidated in each Test Case Specification's "Open Questions" section for stakeholder follow-up.

## Outputs explained

Each profile produces five files:

- **`p1-profile-analysis.md`** — the full reasoning trail for the profile's transaction list, frequency data, and transaction mix. Verbose by design so a reviewer can catch a wrong assumption at its source.
- **`p2-load-profile.md`** — which test types were selected and why (each traces to a specific NFR or risk flag), the load shape for each type, and per-transaction acceptance criteria.
- **`p3-workload-numbers.md`** — the workload model choice (open vs. closed), think time per step, Little's Law calculation showing all inputs and arithmetic, and a throughput reconciliation cross-check.
- **`p4-test-data-specification.md`** — data parameterization requirements (fields, volumes, sources, reuse policy), correlation mapping (dynamic values extracted from one step and injected into later steps), data diversity rules (pool sizing, distribution shape), and a tool-agnostic script blueprint.
- **`test-case-spec.md`** — the condensed, implementer-facing document. Contains only what's needed to configure a script and its data. References the four files above for anyone who wants the full reasoning trail. Includes a consolidated list of all assumptions requiring confirmation before execution.

## Techniques covered

### Phase 1

- System Document Analysis (actor identification, business-event decomposition, NFR extraction)
- Profile Boundary Definition (coherence testing, actor/task partitioning)
- Profile Prioritization (business criticality, risk exposure, volume, dependency ordering)

### Phase 2 — Step 1

- Protocol & System Analysis
- Transaction Identification (including nested sub-transactions and async completion boundaries)
- Persona Behavioral Detailing
- Task Frequency Mapping (APM data, HAR captures, session-replay tooling, growth factor application)
- Transaction Mix Design (User Business Profile weighting for low-volume, high-importance transactions)

### Phase 2 — Step 2

- Test Type Selection (Load, Stress, Spike, Soak/Endurance, Scalability, Volume, Baseline, Capacity)
- Load Shape Design (ramp-up pacing, steady-state duration, spike patterns, stress stop conditions)
- NFR to Acceptance Criteria Mapping (percentile-based targets, hard constraint cross-referencing)

### Phase 2 — Step 3

- Open vs. Closed Workload Model Selection (coordinated omission risk explanation)
- Think Time & Pacing (randomized ranges, HAR/session-replay derivation, zero-think-time prohibition)
- Little's Law Application (dual-method derivation: historical data vs. stated NFR target, reconciliation)
- Throughput Reconciliation (cross-check against source frequency data)

### Phase 2 — Step 4

- Data Parameterization Specification (pool sizing, reuse policy, uniqueness constraints)
- Correlation Mapping Specification (dynamic value extraction, lifetime, single-use flags)
- Data Diversity Rules (Zipfian/power-law distribution, two-tier weighted pool approximation, cache-skew prevention)
- Script Blueprint Specification (initialization, main flow, verification points, error handling, cleanup — no tool syntax)

## Example walkthrough

The `examples/` directory contains a complete worked example of the skill applied to **GreenCart**, a fictional online grocery delivery platform. It was generated by actually running the skill against the system documentation — not hand-authored to illustrate the skill.

```
examples/
├── README.md                             ← GreenCart system documentation (BRD + FRD + NFR + Architecture)
├── operational-profiles.md               ← Phase 1 output: 4 discovered profiles
├── PROF-01-registered-customer-browse-checkout/
├── PROF-02-guest-customer-quick-checkout/
├── PROF-03-warehouse-staff-order-fulfillment/
└── PROF-04-system-regional-inventory-reconciliation/
```

The four profiles were selected to illustrate distinct scenarios:

- **PROF-01 / PROF-02** — customer-facing, open workload model, checkout flow with a recently-reworked code path and a third-party payment gateway constraint
- **PROF-03** — internal staff tooling, **closed workload model** (staffing-bounded, not arrival-driven), two concurrent actor tracks (Pickers/Packers + Dispatchers)
- **PROF-04** — scheduled batch job, no human actor, **Volume and Stress testing** instead of Load/Spike, regional scheduling overlap as the key risk factor rather than concurrent user traffic

Reading the `examples/` directory alongside the `phases/` guidance shows how each technique decision was made — including cases where a technique was explicitly _not_ applied and the reason is stated, assumptions that were flagged rather than invented, and cross-step inconsistencies that were caught during the process.

## Repository structure

```
performance-test-design/
├── SKILL.md                                         ← skill entry point
├── phases/
│   ├── phase-1-operational-profile-discovery/
│   │   ├── README.md
│   │   ├── techniques/
│   │   │   ├── system-document-analysis.md
│   │   │   ├── profile-boundary-definition.md
│   │   │   └── profile-prioritization.md
│   │   ├── resources/
│   │   │   └── profile-types-reference.md
│   │   ├── output-template.md
│   │   ├── output-quality-checklist.md
│   │   ├── best-practices.md
│   │   └── anti-patterns.md
│   │
│   └── phase-2-per-profile-test-design/
│       ├── README.md
│       ├── step-1-profile-analysis/
│       │   ├── techniques/  (5 files)
│       │   ├── resources/   (2 files)
│       │   ├── output-template.md
│       │   ├── output-quality-checklist.md
│       │   ├── best-practices.md
│       │   └── anti-patterns.md
│       ├── step-2-load-profile-design/
│       │   └── [same structure, 3 techniques]
│       ├── step-3-throughput-concurrency/
│       │   └── [same structure, 4 techniques]
│       └── step-4-test-case-specification/
│           └── [same structure, 4 techniques, 2 output templates]
│
└── examples/
    ├── README.md                   ← GreenCart system documentation
    ├── operational-profiles.md
    └── PROF-0{1-4}-*/              ← 4 profile folders, 5 files each
```

## Standards alignment

The skill's process and terminology follow the **ISTQB Certified Tester — Performance Testing (CT-PT) syllabus** (2018), specifically:

- ISTQB CT-PT 4.2.1 — Typical Communication Protocols → Step 1 Protocol & System Analysis
- ISTQB CT-PT 4.2.2 — Transactions → Step 1 Transaction Identification
- ISTQB CT-PT 4.2.3 — Identifying Operational Profiles → Phase 1 + Step 1
- ISTQB CT-PT 4.2.4 — Creating Load Profiles → Step 2
- ISTQB CT-PT 4.2.5 — Analyzing Throughput and Concurrency → Step 3
- ISTQB CT-PT 4.2.6 — Basic Structure of a Performance Test Script → Step 4 Script Blueprint (specification scope only)
- ISTQB CT-PT 4.2.7 — Implementing Performance Test Scripts → Step 4 Parameterization & Correlation (specification scope only)

Several techniques extend beyond the syllabus's summary level with real-world practice: HAR-based think-time derivation, dual-method Little's Law reconciliation, Zipfian distribution for data diversity, growth factor application, and the User Business Profile concept for transaction-mix weighting.
