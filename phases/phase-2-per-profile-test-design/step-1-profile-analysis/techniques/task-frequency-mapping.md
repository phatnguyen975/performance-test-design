# Technique: Task Frequency Mapping

**Grounding:** ISTQB CT-PT 4.2.3 (frequency/probability component), extended with practical data-sourcing methods (HTTP Archive/HAR analysis, session-replay tooling, APM transaction analytics) and capacity-planning practice for applying a growth factor to historical data.

## What It Is

Attaching a frequency value (count per unit time) to every transaction identified in this profile. Usage frequency data can come from copying an existing, similar system's profile, from logging/monitoring actual usage, or from interviewing users when no data exists — these are realistic estimates, not claims of perfectly accurate measurement, and should be presented as such.

## When to Use

- After Transaction Identification and Persona Behavioral Detailing, for every transaction now defined for this profile.
- Whenever the system (or a predecessor) already has production traffic — the strongest source of truth.

## When NOT to Use

- Don't apply this technique using a single day of log data as representative — check for known peak/off-peak cycles (daily, weekly, seasonal) first, consistent with the temporal pattern already captured in Persona Behavioral Detailing.
- Don't skip this step for a new feature with no data — an estimate, clearly labeled, is still required.

## How to Apply, by Data Availability

**If production APM/log data exists:**

1. Query transaction counts per task over a representative time window (avoid atypical days — promotions, outages, holidays).
2. Normalize to a rate (transactions/hour, or per-minute at peak) matching the unit later steps will need.
3. Cite the exact data source, window, and any normalization applied.

**If HAR (HTTP Archive) captures or session-replay tooling (e.g., FullStory, Hotjar, Clarity) are available but full APM transaction counts aren't:**

1. HAR captures record every network request in a browser session with full timing — useful for deriving both frequency (how many times a request type appears across a sample of captured sessions) and the execution-time component needed later for pacing (Step 3).
2. Session-replay tools provide session-level interaction sequences even without exposing raw network logs — usable to count how often a given user-facing action (e.g., "Apply Coupon" click) occurs per session, which can be converted to a system-wide rate once combined with known total session counts for the time window.
3. Always state which method produced the figure (HAR sample vs. session-replay-derived vs. direct APM count) since they carry different confidence levels — a HAR sample from a handful of captured sessions is a weaker source than a full APM transaction count and should be labeled accordingly, e.g. `[ESTIMATED FROM HAR SAMPLE, n=40 sessions]`.

**If a comparable predecessor system exists:**

1. Use its logged frequencies as a starting estimate.
2. Adjust explicitly for known differences (a redesigned flow removing a step, a new user segment).

**If neither exists (greenfield system/feature):**

1. Interview product owners/business stakeholders for expected volumes.
2. Cross-check against market-sizing or business-case documents.
3. Label every resulting number `[ASSUMPTION - stakeholder estimate, not measured]`.

## Applying a Growth Factor

Historical data reflects the past, not the target testing horizon. Where the NFR document or business context specifies a growth expectation (e.g., "must handle 20% year-over-year growth" or a specific future date the system must be ready for), apply that growth factor explicitly to the sourced frequency rather than testing only against today's historical volume: `projected_frequency = measured_frequency × (1 + growth_rate)^years_ahead`. State the growth rate's source (business plan, NFR document) — never apply a growth factor invented without a cited basis.

## Output

A frequency table per transaction: raw count/rate, time window, data source (with confidence level where estimated), and — where applicable — the growth-factor-adjusted projected figure alongside the raw measured one.

## Example

| Transaction                                  | Measured Frequency (peak hr) | Growth-Adjusted (18mo @ 15%/yr per NFR-DOC-03 §2)                                                                | Source                                           |
| -------------------------------------------- | ---------------------------- | ---------------------------------------------------------------------------------------------------------------- | ------------------------------------------------ |
| Browse Catalog                               | 42,000/hr                    | ~49,600/hr                                                                                                       | Production APM, last 90 days, weekday peak 7–9pm |
| Search                                       | 18,500/hr                    | ~21,900/hr                                                                                                       | Production APM, same window                      |
| Add to Cart                                  | 6,200/hr                     | ~7,340/hr                                                                                                        | Production APM, same window                      |
| Apply Coupon                                 | 1,100/hr                     | ~1,300/hr                                                                                                        | Production APM, same window                      |
| Confirm Payment                              | 3,900/hr                     | ~4,610/hr                                                                                                        | Production APM, same window                      |
| Submit Order                                 | 3,750/hr                     | ~4,440/hr                                                                                                        | Production APM, same window                      |
| "Buy Now Pay Later" option (new, unlaunched) | —                            | 400/hr `[ASSUMPTION - stakeholder estimate, 10% adoption of Submit Order volume per product team business case]` | Product team interview, 2026-06                  |

Both the raw measured figure and the growth-adjusted figure are retained in the output — Step 2 and Step 3 use the growth-adjusted figure for target throughput, but keeping the raw figure visible lets a reviewer verify the growth calculation independently.
