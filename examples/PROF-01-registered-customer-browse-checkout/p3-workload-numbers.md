# Workload Numbers — Registered Customer: Browse & Checkout (PROF-01)

## 1. Open vs. Closed Model Selection

| Transaction                                                        | Model | Justification                                                                                                                                                             |
| ------------------------------------------------------------------ | ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Browse Catalog, Search, Add to Cart, Confirm Payment, Submit Order | Open  | Public customer traffic arrives independently of system response speed; open model avoids coordinated omission for this revenue-critical, recently-reworked checkout path |

No transaction in this profile is paced by a fixed internal worker pool — all are human-driven, customer-facing actions, so the open model applies uniformly here (unlike a profile with a backend batch component, e.g. PROF-04).

## 2. Think Time & Pacing

| Step                                 | Think Time                                                  | Source                                                                                                                                                                       |
| ------------------------------------ | ----------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Product list → click product         | 3–7s                                                        | `[ASSUMPTION - Persona Behavioral Detailing in Step 1 flagged session analytics as unavailable in the source document; estimated from typical grocery-app browsing cadence]` |
| Product detail → Add to Cart         | 4–10s                                                       | Same basis                                                                                                                                                                   |
| Cart → Select Delivery Slot          | 5–12s (slot selection involves reviewing available windows) | Same basis                                                                                                                                                                   |
| Delivery Slot → Apply Payment/Promo  | 3–8s                                                        | Same basis                                                                                                                                                                   |
| Payment form → Confirm Payment click | 6–15s                                                       | Same basis                                                                                                                                                                   |

**Pacing calculation:** Execution time (P50, from Step 2's acceptance criteria used as a planning proxy since actual measured P50s don't exist pre-test): Browse ~300ms + Search ~300ms + Add to Cart ~200ms + Confirm Payment ~1500ms + Submit Order ~1500ms ≈ 3.8s. Think time (midpoints): 5 + 7 + 8.5 + 5.5 + 10.5 = 36.5s. **Total pacing ≈ 40.3s per full iteration.**

## 3. Little's Law Application

**Method A (from Step 1's estimated frequency data):** λ ≈ Submit Order growth-adjusted frequency = 1,414/hr = 0.393/sec. **Flagged: this inherits Step 1's weak sourcing** (back-calculated, not directly measured).

**Method B (from NFR):** §4.1.4 states ≥1,800 concurrent sessions system-wide at peak; Step 1 allocated 70% to Registered Customer = 1,260 concurrent sessions. This is a **directly-stated NFR figure**, not derived — Method B is the stronger source here, the reverse of the earlier e-commerce example in this skill's technique files where Method A happened to be stronger. This asymmetry is itself worth noting: which method is stronger depends on what the source documents actually state, not a fixed preference for one method over the other.

**Reconciliation:** Method B (1,260, directly NFR-sourced) is used as the design target over Method A's weaker, back-calculated figure. This also means Step 1's Task Frequency Mapping table should, in a real engagement, be revisited once real APM data is available — Method B's strength here doesn't retroactively fix Step 1's data quality, it just means this specific number (N) doesn't depend on Step 1's weaker figures.

**N = 1,260 concurrent sessions (taken directly from Method B, already expressed as a concurrency figure, not re-derived through N = λ × W since the NFR states N directly rather than stating λ).**

Cross-check via N = λ × W: implied λ = N / W = 1,260 / 40.3s ≈ 31.3 iterations/sec ≈ 112,500 iterations/hr — this is far higher than Step 1's Submit-Order-based estimate (1,414/hr), which makes sense: not every session that browses reaches Submit Order (cart abandonment, per Step 1's unquantified "behavioral variability" gap), so a session-level concurrency figure (1,260) is expected to imply a much higher _iteration-start_ rate than the _order-completion_ rate alone — these measure different things and the large gap between them is not an error, but it does mean pacing (W) as calculated above represents a _full completed checkout_ iteration, while many real sessions won't reach that point. **This distinction is flagged for Step 4** since it affects how "iteration" is defined in the script blueprint (does an iteration always attempt checkout, or does it sometimes stop at Browse/Search, matching real abandonment behavior more closely?).

**Adding a 10% safety margin: 1,260 × 1.10 ≈ 1,386, rounded to 1,390 concurrent virtual users (final).**

**Sanity check:** 1,390 concurrent sessions against a 210,000-registered-account base (§1) is a plausible peak-hour concurrency fraction (~0.66%), consistent with typical grocery-delivery engagement patterns during a defined evening peak window.

## 4. Throughput Reconciliation

| Transaction  | Target (Step 1, growth-adj., weak source) | Achieved (1,390 VU @ 40.3s, full-checkout iteration assumption) | Model | Reconciliation                                                                                                                                                                                                                                                                                     |
| ------------ | ----------------------------------------- | --------------------------------------------------------------- | ----- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Submit Order | 1,414/hr                                  | ≈124,100/hr (if every iteration reaches Submit Order)           | Open  | **Large mismatch** — as explained above, this VU count represents total concurrent _sessions_, not all of which complete checkout; the achieved figure above is only valid if the script blueprint models 100% of iterations reaching Submit Order, which does not match real abandonment behavior |

**Resolution flagged for Step 4:** the Script Blueprint must model a conditional flow — not every virtual user's iteration proceeds all the way to Submit Order. A `[ASSUMPTION]` conversion rate (e.g., an estimated 8–12% of sessions reaching Submit Order, consistent with typical grocery e-commerce conversion, pending real data) should be applied so that the _achieved_ Submit Order throughput reconciles closer to Step 1's ~1,414/hr figure, while the full 1,390 VUs still generates realistic Browse/Search/Cart-level traffic matching §4.1.4's session-concurrency target. This is exactly the kind of cross-step reconciliation issue this skill's process is designed to surface before it reaches an implementer.

## Step 3 AI Gate Self-Check Summary

Every transaction's model is stated (open, uniformly, with justification). Think time is a range, never zero. Pacing arithmetic is shown. Both Little's Law methods are shown; Method B's superior sourcing is explicitly identified (correctly reversing the usual pattern from this skill's technique documentation, since here the NFR document happened to state concurrency directly). The large discrepancy between session-concurrency and order-completion-rate is surfaced explicitly, not silently reconciled, and is carried forward as an explicit modeling requirement for Step 4 rather than left implicit. Proceeding to Step 4 with this flag carried forward.
