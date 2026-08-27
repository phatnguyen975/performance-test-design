# Anti-Patterns — Step 1: Profile Analysis

- Guessing a transaction mix without stating the source — indistinguishable from a fabricated number to anyone reviewing later.
- Re-litigating Phase 1's persona/actor assignment inside Persona Behavioral Detailing instead of raising it as a change request against `operational-profiles.md`.
- Treating a business flow as one atomic transaction when it contains a high-risk, independently-measurable sub-step — this hides exactly the defect performance testing exists to catch.
- Using a single unrepresentative day of log data, or a HAR sample too small to be meaningful, without flagging the resulting confidence level.
- Applying a growth factor invented without a citable source, or skipping growth adjustment when the NFR document explicitly specifies one.
- Skipping Protocol & System Analysis for a profile with async/queued hops — Transaction Identification will likely try to measure an asynchronous flow as a simple request/response, producing an unscriptable boundary.
- Computing Transaction Mix Design without sanity-checking against Persona Behavioral Detailing's session-shape/conversion data.
- Rounding an assumption into a suspiciously clean number without showing its derivation.
