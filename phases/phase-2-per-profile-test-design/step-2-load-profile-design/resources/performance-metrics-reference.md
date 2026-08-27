# Resource: Performance Metrics Reference

## Response Time Percentiles — Why Not Averages

An average can look healthy while masking a meaningful subset of users experiencing much worse latency. P50 = typical experience. P90/P95 = "worse than typical" tail, most common SLA target. P99 = rare-but-real worst case; often reveals contention effects (lock waits, GC pauses, connection pool queuing) invisible at lower percentiles.

## Throughput

TPS/RPS — the sustained rate the system must handle, not a peak instantaneous burst; applies to the steady-state portion of the load shape.

## Error Rate

Track and threshold separately from response time — a system can meet its response-time target while silently failing (timing out fast) a meaningful fraction of requests. Categorize errors where possible (timeouts vs. 5xx vs. application-level validation failures).

## Resource Utilization

CPU, memory, disk I/O, network — server-side metrics indicating headroom. A transaction meeting its response-time target at 95% CPU has no safety margin for growth.

## Concurrency

The simultaneous active sessions the acceptance criteria assume — ties directly into Step 3's Little's Law calculation.
