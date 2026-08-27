# Technique: Transaction Mix Design

**Grounding:** ISTQB CT-PT 4.2.3 (output artifact), extended with the User Business Profile (UBP) concept from practical performance-testing methodology (Scott Barber's workload modeling approach), which explicitly separates raw frequency weighting from business-importance weighting rather than conflating them.

## What It Is

The final synthesis step of Step 1: converting frequencies into a **normalized percentage mix** — the proportion each transaction represents of the profile's total activity. Basic transaction mix design weights purely by measured frequency. The User Business Profile refinement goes further: it explicitly considers _business importance_ as a second, separate weighting dimension, because a purely frequency-weighted mix can under-represent a low-frequency, high-importance transaction in the resulting test (a problem already flagged as a risk in Task Frequency Mapping's growth-adjustment step and Phase 1's Profile Prioritization, but which resurfaces here at the transaction level within a single profile).

## When to Use

- As the last step of Step 1, always.
- Whenever frequencies change (Task Frequency Mapping updated) — recalculate, don't patch.

## When NOT to Use

- Don't compute a mix across multiple Operational Profiles combined — scoped to this profile only (root skill Core Principle 3).

## How to Apply

1. **Compute the raw frequency-weighted mix:** sum all transaction frequencies (growth-adjusted, from Task Frequency Mapping), divide each transaction's frequency by the total.
2. **Apply the User Business Profile check:** for each transaction, ask whether its business importance (from Phase 1's actor/flow context, and any risk flags raised in Protocol & System Analysis or Transaction Identification) is disproportionate to its frequency share. A transaction can be simultaneously low-frequency and high-importance — this doesn't change its _frequency-weighted_ mix percentage (that stays honest to the real data), but it does mean the transaction needs explicit visibility rather than being left to blend anonymously into "the mix."
3. **Sanity-check against Persona Behavioral Detailing's session-shape data** — e.g., if that technique found only ~35% of sessions reaching checkout, the mix's Confirm Payment/Submit Order share should be consistent with that conversion rate applied to the profile's overall session volume, not an independently-guessed number.
4. **Flag every transaction whose mix percentage is small but whose business/technical risk is disproportionately high** — carry this flag forward explicitly into Step 2's test-type selection, since a pure Load Test focused on the dominant mix might under-test a low-volume, high-risk transaction.

## Output

A finalized transaction mix table (percentages summing to 100%), with UBP risk/importance flags, ready to hand to Step 2.

## Example

Continuing the e-commerce "Registered Shopper — Browse & Checkout" profile, using growth-adjusted frequencies (total: 49,600 + 21,900 + 7,340 + 4,610 + 4,440 + 1,300 = 89,190/hr):

| Transaction     | Frequency (growth-adj., peak hr) | Mix %    | UBP Flag                                                                                                                                         |
| --------------- | -------------------------------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| Browse Catalog  | 49,600                           | 55.6%    | —                                                                                                                                                |
| Search          | 21,900                           | 24.6%    | —                                                                                                                                                |
| Add to Cart     | 7,340                            | 8.2%     | —                                                                                                                                                |
| Confirm Payment | 4,610                            | 5.2%     | **High business importance despite low share** — 3rd-party dependency, hard 10s timeout (from Protocol & System Analysis), direct revenue impact |
| Submit Order    | 4,440                            | 5.0%     | Known DB contention point (from Transaction Identification)                                                                                      |
| Apply Coupon    | 1,300                            | 1.5%     | —                                                                                                                                                |
| **Total**       | **89,190**                       | **100%** |                                                                                                                                                  |

Sanity check against Persona Behavioral Detailing: Confirm Payment + Submit Order together (~10.2% of mix) should roughly track the ~35% cart-to-checkout conversion rate applied against the _cart-reaching_ sub-population, not the full session population — since Browse Catalog and Search dominate total traffic precisely because most sessions never reach cart at all. This reconciles consistently and requires no adjustment here.
