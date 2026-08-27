# Anti-Patterns — Step 2: Load Profile Design

- Applying every test type to every transaction regardless of risk — bloats the suite with no clear rationale.
- Instantaneous ramp-up ("all VUs start at t=0") creating an artificial thundering-herd effect.
- Setting a Stress/Capacity Test's stop condition to "run until it breaks" with no defined threshold.
- Expressing an acceptance criterion only as an average response time.
- Setting an acceptance criterion looser than a known hard downstream constraint.
- Silently inventing an NFR value when the document doesn't specify one.
- Reusing a fixed-infrastructure ramp-up pace for a system that auto-scales without checking the scaler's reaction interval.
