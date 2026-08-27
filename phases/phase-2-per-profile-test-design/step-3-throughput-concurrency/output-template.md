# Output Template — `p3-workload-numbers.md` (Step 3 Deliverable)

```markdown
# Workload Numbers — [Profile Name] ([Profile ID])

## 1. Open vs. Closed Model Selection

[Table: Transaction | Model | Justification]

## 2. Think Time & Pacing

[Table: Step | Think Time | Source]
[Pacing calculation shown explicitly]

## 3. Little's Law Application

[Method A λ (from Step 1 data) shown with arithmetic]
[Method B λ (from NFR, if available) shown with arithmetic]
[Reconciliation: which was used and why]
[W value with source]
[N = λ × W arithmetic shown]
[Safety margin applied and final rounded VU count]
[Sanity check against known population size]

## 4. Throughput Reconciliation

[Table: Transaction | Target Throughput (Step 1) | Achieved Throughput | Model | Reconciliation]
[Explicit note on any transaction falling short of its Step 2 acceptance-criteria throughput target]
[Explicit note if this VU count represents a specific scenario (e.g. promotional-event scale) rather than typical-day load]

## Step 3 AI Gate Self-Check Summary

[Confirmation output-quality-checklist.md was run, and its result]
```
