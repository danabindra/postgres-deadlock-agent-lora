# Prompt 5: Cascading Failures from Tight Coupling

Use this prompt with Claude, Gemini, or ChatGPT to generate cascading failure training data.
Copy the block below and paste it directly.

---

```
Generate 60 cascading failure diagnostic training examples in JSONL format.
Each line must be a valid JSON object with a single "messages" key containing an array
with exactly two objects: one with "role": "user" and one with "role": "assistant".

These examples target monolithic or tightly-coupled service architectures where one slow
or failing component causes others to degrade. The user message should simulate what an
agent collects from: service error rates, latency percentiles, connection/thread pool stats,
circuit breaker status, and dependency health checks.

Format the user message like this:
"Analyze this cascading failure diagnostic data:

Timestamp: [ISO timestamp] | Duration of incident so far: [Xm]

Service health summary:
  [service-A]:       [healthy / degraded / down] | p50: [Xms] p99: [Xms] | Error rate: [X%]
  [service-B]:       [healthy / degraded / down] | p50: [Xms] p99: [Xms] | Error rate: [X%]
  [service-C]:       [healthy / degraded / down] | p50: [Xms] p99: [Xms] | Error rate: [X%]
  [database/queue]:  [healthy / degraded / down] | latency: [Xms] | connections: [N/max]

Thread/worker pool status:
  [service-B] worker threads: [N active] / [N max] | Queue depth: [N] | Oldest queued: [Xs]

Circuit breakers:
  [service-A → service-C]: [closed / open / half-open]
  [service-B → database]:  [closed / open / half-open]

Recent events (timeline):
  [T-8m]: [what started — e.g. deploy, traffic spike, external API slowdown]
  [T-5m]: [first symptom visible]
  [T-3m]: [second service started degrading]
  [T-0m]: [current state]

Error log samples:
  [service-B]: 'upstream timeout after 30s waiting for service-C'
  [service-A]: 'thread pool exhausted — rejecting requests'"

The assistant message must be an expert diagnosis that:
- Opens with one of: CASCADING FAILURE — ROOT CAUSE IDENTIFIED, OVERLOAD PROPAGATION, THUNDERING HERD, DEPENDENCY TIMEOUT CHAIN, or DEGRADED — NOT YET CASCADING
- Identifies the root cause service or component and explains why it started failing
- Traces the exact cascade path (service A → B → C) with explanation of the coupling mechanism
- Identifies which missing resilience pattern allowed the cascade (no circuit breaker, no timeout, no bulkhead, no retry budget, synchronous dependency on non-critical path)
- Gives an immediate triage action (which circuit to open manually, which service to isolate, which queue to drain)
- Gives the long-term architectural fix (decoupling strategy, circuit breaker config, async pattern)

Cover ALL of these scenario types — distribute evenly across the 60 examples:
1. Slow database causes thread pool exhaustion in the app, which causes upstream timeout in the API gateway
2. External payment API goes slow, no timeout set, threads pile up, entire checkout service dies
3. Thundering herd after a deployment restart — all instances hit the cache cold at once
4. One microservice's memory leak slows its GC, response times increase, callers time out and retry, multiplying load
5. Queue consumer falls behind, queue depth grows, producers start blocking on enqueue, front-end degrades
6. Circuit breaker opens correctly and isolates the failure — success case showing cascade stopped
7. Missing bulkhead: one slow endpoint consumes all threads leaving none for healthy endpoints on same service
8. Retry storm: callers retry on timeout without backoff, tripling load on already-struggling service
9. Config change (timeout too low) causes healthy service to appear degraded to its callers
10. False alarm — one service is slow but isolated, no cascade in progress, other services healthy

Output only raw JSONL — one JSON object per line, no markdown, no explanation, no numbering.
```
