# Resource: Data Gathering Sources for Transaction Frequencies

Reference for Task Frequency Mapping. Ranked roughly by reliability — prefer higher items when available.

## 1. Production APM / Access Logs

Strongest source when the system (or a predecessor) is live. Web/app server access logs, APM tools (New Relic, Datadog, Dynatrace, AppDynamics), API gateway metrics, database query logs for backend-heavy profiles. Check the time window for representativeness.

## 2. HAR (HTTP Archive) Captures and Session-Replay Tooling

Useful when full APM transaction-level analytics aren't accessible but browser-level captures are. HAR files record every network request with timing, usable for both frequency sampling and execution-time sourcing. Session-replay tools (FullStory, Hotjar, Microsoft Clarity) provide interaction-sequence data even without raw network logs. Always label the confidence level of a HAR/session-replay-derived figure lower than a full APM count, since it's typically sampled from a smaller session set.

## 3. Business/Product Analytics Platforms

Google Analytics, Mixpanel, Amplitude, or internal BI dashboards — often already aggregate funnel-step volumes. Useful for cross-checking APM-derived numbers and for session-shape data (feeds Persona Behavioral Detailing as much as this technique).

## 4. Predecessor System Data

Legitimate starting point when testing a replacement or major rework — adjust explicitly for known behavioral differences.

## 5. Stakeholder Interviews

Used when no usage data exists. Cross-check a single stakeholder's estimate against at least one other source where possible.

## 6. Business Case / Market-Sizing Documents

For new features, the original business case's adoption estimates are a legitimate, citable assumption source.

## Applying a Growth Factor — Where to Find the Rate

Check the NFR/SLA document first (growth targets are often stated there), then the business case/strategy document, then ask the stakeholder directly if neither states one. Never invent a growth rate without a cited basis — if none exists, test against current measured volume only and state explicitly that no growth projection was available.

## What to Do When No Source Is Available At All

State this explicitly rather than silently picking a round number: `"Frequency for [transaction] could not be sourced — flagging as an open question before Step 2 proceeds."`
