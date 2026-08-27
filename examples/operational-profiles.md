# Operational Profiles — GreenCart

## Source Documents

- `examples/README.md` — GreenCart System Requirements Documentation v3.2, 2026-05-18 (combined BRD/FRD/NFR/Architecture doc). All section references (§) below refer to this document.
- **Not available:** A dedicated use-case document (the FRD's requirement-level detail was used instead — sufficient for this discovery, but a formal use-case doc would have made actor/flow extraction more precise; noted as a gap, not a blocker). No analytics exist yet for the Inventory Reconciliation batch job (§6) beyond job-duration/success logging.

## 1. System Document Analysis Summary

### Actors

| Actor                           | Primary/Secondary          | Functional Role                        | Notes                                                                                                                                    |
| ------------------------------- | -------------------------- | -------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| Registered Customer             | Primary                    | Customer-facing, transactional         | Has account, saved payment methods, order history (§2)                                                                                   |
| Guest Customer                  | Primary                    | Customer-facing, transactional         | Web only, no saved payment methods (§2, FR-2.4)                                                                                          |
| Warehouse Picker/Packer         | Primary                    | Operational staff                      | Scans items, packs orders (§2, FR-3.2–3.3)                                                                                               |
| Delivery Dispatcher             | Primary                    | Operational staff                      | Assigns/confirms dispatch; explicitly noted as low headcount/low volume (§2)                                                             |
| Regional Inventory Sync Service | Secondary, external system | Scheduled/system-triggered             | No human actor; nightly batch (§2, FR-4.1)                                                                                               |
| Payment Gateway                 | Secondary, external system | Downstream dependency                  | Called BY GreenCart, does not generate inbound load — not a profile itself, but a hard constraint on the Checkout flow (§4.3, NFR-4.3.1) |
| SMS/Push Notification Provider  | Secondary, external system | Downstream dependency, fire-and-forget | Same treatment as Payment Gateway — not a profile                                                                                        |

### Business Events / Flows

| Flow                                                               | Source | Associated Actor(s)                                                                                     |
| ------------------------------------------------------------------ | ------ | ------------------------------------------------------------------------------------------------------- |
| Browse by category                                                 | FR-1.1 | Registered Customer, Guest Customer                                                                     |
| Search with autocomplete                                           | FR-1.2 | Registered Customer, Guest Customer                                                                     |
| View real-time stock availability                                  | FR-1.3 | Registered Customer, Guest Customer                                                                     |
| Cart management (add/remove/adjust)                                | FR-2.1 | Registered Customer, Guest Customer                                                                     |
| Delivery slot selection                                            | FR-2.2 | Registered Customer, Guest Customer                                                                     |
| Checkout with saved payment/promo                                  | FR-2.3 | Registered Customer only                                                                                |
| Guest manual payment entry                                         | FR-2.4 | Guest Customer only                                                                                     |
| Order completion (inventory reserve, payment auth, order creation) | FR-2.5 | Registered Customer, Guest Customer                                                                     |
| Picking list retrieval                                             | FR-3.1 | Warehouse Picker/Packer                                                                                 |
| Item scan confirmation                                             | FR-3.2 | Warehouse Picker/Packer                                                                                 |
| Pack confirmation                                                  | FR-3.3 | Warehouse Picker/Packer                                                                                 |
| Dispatch assignment/confirmation                                   | FR-3.4 | Delivery Dispatcher                                                                                     |
| Nightly stock reconciliation                                       | FR-4.1 | Regional Inventory Sync Service                                                                         |
| Discrepancy flagging                                               | FR-4.2 | Regional Inventory Sync Service (note: the human morning-review step is explicitly out of scope per §7) |

### NFR/Constraint Extraction

| NFR                                                                  | Applies To                     | Source               | Gap flag                                                                                                                             |
| -------------------------------------------------------------------- | ------------------------------ | -------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| Browse/Search P95 ≤600ms                                             | Browse, Search flows           | §4.1.1               | —                                                                                                                                    |
| Add to Cart P95 ≤400ms                                               | Cart flow                      | §4.1.2               | —                                                                                                                                    |
| Checkout P95 ≤3s, P99 ≤6s                                            | Checkout/Order completion      | §4.1.3               | —                                                                                                                                    |
| ≥1,800 concurrent sessions, evening peak                             | Customer-facing, system-wide   | §4.1.4               | —                                                                                                                                    |
| +40% session/throughput growth within 6mo of Q3 expansion            | Customer-facing, system-wide   | §4.1.5, tied to BG-2 | Growth rate has a source; applies system-wide, not yet broken down per-transaction                                                   |
| Item scan P95 ≤300ms                                                 | Item scan confirmation         | §4.2.1               | —                                                                                                                                    |
| Picking-list generation P95 ≤2s                                      | Picking list retrieval         | §4.2.2               | —                                                                                                                                    |
| Payment Gateway: 8s/99%, hard 12s timeout                            | Checkout (external constraint) | §4.3.1               | Not a GreenCart-set target — a constraint to design against                                                                          |
| 99.9% availability, customer-facing, business hours                  | All customer-facing flows      | §4.4.1               | —                                                                                                                                    |
| Inventory reconciliation: complete within window, ≤0.5% failure rate | Nightly reconciliation         | §4.4.2               | —                                                                                                                                    |
| 99.5% availability, warehouse tooling                                | Warehouse flows                | §4.4.3               | —                                                                                                                                    |
| Checkout-path error rate ≤0.1%                                       | Cart, Checkout, Payment        | §4.5.1               | —                                                                                                                                    |
| Browse/Search error rate ≤0.5%                                       | Browse, Search                 | §4.5.2               | —                                                                                                                                    |
| Warehouse scan error rate ≤0.2%                                      | Item scan                      | §4.5.3               | —                                                                                                                                    |
| Dispatch assignment                                                  | Delivery Dispatcher flow       | —                    | **No explicit NFR found for dispatch assignment response time/error rate** — flagged for Phase 2 Step 2 to resolve with stakeholders |
| Nightly reconciliation response time/throughput                      | Inventory Sync                 | —                    | **No explicit per-record or per-batch throughput NFR** beyond the overall window and failure-rate constraints (§4.4.2) — flagged     |

### System Boundary Summary

Customer-facing web/mobile → API Gateway (HTTPS/REST) → Product Catalog / Cart / Order / Inventory microservices (HTTPS/REST internal) → Order Service → Payment Gateway (HTTPS/REST, external, hard 12s timeout) and → order-status event queue (AMQP) → Notification Service (→ external SMS/Push, fire-and-forget) and → WMS integration layer. Warehouse handheld scanners → separate Warehouse API (HTTPS/REST) → shared Inventory Service. Regional Inventory Sync Service → SFTP CSV extracts from WMS → direct JDBC writes to central Inventory DB (legacy pattern, §5.1).

### Gap Log

- No dedicated use-case document was available; actor/flow extraction relied on the FRD's requirement-level detail.
- No analytics exist for the Inventory Reconciliation batch job beyond job-duration/success logging (§6) — Task Frequency Mapping for PROF-04 in Phase 2 will need to rely more heavily on stakeholder estimation than the customer-facing profiles.
- No explicit NFR exists for Delivery Dispatcher's dispatch-assignment flow, nor for the Inventory Sync job's per-record/per-batch throughput.

## 2. Operational Profile List

| ID      | Name                                       | Actor(s)                                     | Scope                                                                                                                                                                                                                                                                                                                                                                                             |
| ------- | ------------------------------------------ | -------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| PROF-01 | Registered Customer — Browse & Checkout    | Registered Customer                          | Includes: Browse (FR-1.1–1.3), Cart (FR-2.1–2.2), Checkout with saved payment/promo (FR-2.3, FR-2.5). Excludes: guest-specific manual payment entry (PROF-02); loyalty/rewards (out of scope per §7)                                                                                                                                                                                              |
| PROF-02 | Guest Customer — Quick Checkout            | Guest Customer                               | Includes: Browse (FR-1.1–1.3), Cart (FR-2.1–2.2), guest manual payment checkout (FR-2.4, FR-2.5). Excludes: any account-bound flow, saved payment methods                                                                                                                                                                                                                                         |
| PROF-03 | Warehouse Staff — Order Fulfillment        | Warehouse Picker/Packer, Delivery Dispatcher | Includes: picking list retrieval, item scan, pack confirmation (FR-3.1–3.3), dispatch assignment/confirmation (FR-3.4). Merged into one profile per Profile Boundary Definition's coherence test — same general population (warehouse staff), sequential operational context (single fulfillment pipeline), similar criticality; Dispatcher's explicitly lower volume noted but not disqualifying |
| PROF-04 | System — Regional Inventory Reconciliation | Regional Inventory Sync Service              | Includes: nightly stock reconciliation and discrepancy flagging (FR-4.1–4.2). Excludes: the human morning-review step (out of scope per §7)                                                                                                                                                                                                                                                       |

### Coverage Cross-Check

Every business event listed in System Document Analysis maps to exactly one profile above:

- Browse/Search/Cart/Delivery-slot flows → both PROF-01 and PROF-02 (these are genuinely shared flows performed by both actors — this is a many-to-one _transaction_-to-_profile_ relationship within each profile, not an overlap between profiles; each profile independently includes these flows in its own transaction list, since Step 1 in Phase 2 analyzes each profile's transactions separately even where the underlying endpoint is shared code)
- Checkout-with-saved-payment (FR-2.3) → PROF-01 only
- Guest manual payment (FR-2.4) → PROF-02 only
- Picking/packing/dispatch (FR-3.1–3.4) → PROF-03 only
- Reconciliation/discrepancy-flagging (FR-4.1–4.2) → PROF-04 only

No business event mapped to zero profiles or ambiguously to more than one distinct profile.

## 3. Prioritization

| Order | ID      | Name                                       | Business Criticality                                                                                                                                                     | Risk Exposure                                                                                                                                                                                  | Volume                               | Priority Justification                                                                                                                                                                                         |
| ----- | ------- | ------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1     | PROF-01 | Registered Customer — Browse & Checkout    | High — primary revenue path, majority of registered base                                                                                                                 | **High** — Order Service's checkout path underwent a major rework 3 months ago (optimistic→pessimistic inventory locking) and has not been load tested since (§5.2); directly affects Checkout | High                                 | Highest combined score across all four factors; recent untested change on the critical path makes this urgent, not just important                                                                              |
| 2     | PROF-04 | System — Regional Inventory Reconciliation | Medium — affects data accuracy and next-day fulfillment, not directly customer-facing revenue                                                                            | **High** — documented lock-contention history when regional reconciliation windows overlap on the shared primary DB (§5.2); directly tied to BG-4                                              | N/A (scheduled, not volume-driven)   | A known, documented risk with a specific architectural cause outranks its non-customer-facing nature                                                                                                           |
| 3     | PROF-02 | Guest Customer — Quick Checkout            | Medium — smaller share of order volume than PROF-01, but still direct revenue                                                                                            | Medium — shares the same recently-reworked Order Service checkout path as PROF-01, so carries similar technical risk, but lower business share moderates overall priority                      | Medium                               | Same technical risk as PROF-01 but lower volume/criticality share justifies testing after PROF-01, not before                                                                                                  |
| 4     | PROF-03 | Warehouse Staff — Order Fulfillment        | Medium-High — directly tied to BG-3 (reducing picking errors) and BG-1 (reducing cart-to-delivery time), but is an internal-tooling profile, not directly revenue-facing | Low-Medium — no documented recent change or incident history for warehouse tooling specifically                                                                                                | Low (bounded by warehouse headcount) | Business-important (two of four release goals touch this profile) but lacks the urgency signal of a known technical risk or an untested recent change — ranked last among the four, not deprioritized entirely |

## Phase 1 AI Gate Self-Check Summary

Verified against `output-quality-checklist.md`: Every actor classified primary/secondary; every business event traced to a source FR/NFR section; NFR gaps (dispatch-assignment response time, reconciliation throughput) explicitly flagged rather than invented; all four profiles pass the three-part coherence test (including the explicit PROF-03 merge justification); scope statements state both inclusions and exclusions; coverage cross-check confirms no orphaned or ambiguously-shared business events; the profile list checked against `resources/profile-types-reference.md` confirms coverage of customer-facing (PROF-01, PROF-02), backoffice/internal-tooling (PROF-03), and batch/scheduled (PROF-04) categories — no system-to-system/integration-partner profile exists in this system beyond the downstream-dependency actors already excluded above, so that category is confirmed absent rather than overlooked; prioritization shown against all four factors, not volume alone. Ready for Human Review Gate 1.
