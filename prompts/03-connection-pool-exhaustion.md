# Prompt 3: Connection Pool Exhaustion

Use this prompt with Claude, Gemini, or ChatGPT to generate connection pool exhaustion training data.
Copy the block below and paste it directly.

---

```
Generate 60 PostgreSQL connection pool exhaustion diagnostic training examples in JSONL format.
Each line must be a valid JSON object with a single "messages" key containing an array
with exactly two objects: one with "role": "user" and one with "role": "assistant".

The user message should simulate the exact output an agent collects from these tools:
pg_stat_activity (connection counts by state and application), pg_settings for max_connections,
PgBouncer stats (if present), and application error logs showing connection timeout errors.

Format the user message like this:
"Analyze this PostgreSQL connection pool diagnostic data:

pg_settings:
  max_connections: [N]

pg_stat_activity summary:
  Total connections: [N] / [max_connections]
  Active (running queries): [N]
  Idle: [N]
  Idle in transaction: [N]
  Idle in transaction (aborted): [N]
  Waiting for lock: [N]
  Top applications by connection count:
    [app_name]: [N] connections
    [app_name]: [N] connections

Longest running idle-in-transaction:
  PID [X] | App: [name] | Duration: [Xm Xs] | Last query: [SQL snippet]

Application error log (last 60 seconds):
  [N] x 'remaining connection slots are reserved for non-replication superuser connections'
  [or: 'FATAL: sorry, too many clients already']
  [or: 'connection timeout after 30000ms']

PgBouncer stats (if applicable):
  Pool mode: [transaction/session]
  Pool size: [N] | Active: [N] | Waiting: [N] | Max wait: [Xs]"

The assistant message must be an expert diagnosis that:
- Opens with one of: CONNECTION POOL EXHAUSTED, IDLE IN TRANSACTION LEAK, POOL MISCONFIGURATION, CONNECTION SPIKE — TRAFFIC EVENT, or APPROACHING LIMIT — NOT YET EXHAUSTED
- Identifies the root cause: which application is leaking, whether it is idle-in-transaction sessions, missing connection limits per role, PgBouncer misconfiguration, or a traffic spike
- Gives an immediate fix (which sessions to terminate, which PIDs, what SQL to run)
- Gives a long-term fix (pool sizing, PgBouncer config change, application code change)
- Includes the exact SQL for immediate remediation where applicable

Cover ALL of these scenario types — distribute evenly across the 60 examples:
1. Application not closing connections after use — connection count grows until exhaustion
2. Idle-in-transaction sessions holding connections open for minutes (forgotten BEGIN with no COMMIT)
3. PgBouncer pool too small for the actual application concurrency
4. PgBouncer in session mode instead of transaction mode for a stateless app
5. Traffic spike (marketing event, batch job) causing legitimate but sudden pool pressure
6. max_connections set too low for the number of application instances deployed
7. One application instance leaked connections after a crash without cleanup
8. Long-running report/analytics query holding a connection during execution
9. Missing connection_limit on a role allowing one app to starve others
10. False alarm — pool at 80% but healthy, just needs monitoring threshold adjustment

Output only raw JSONL — one JSON object per line, no markdown, no explanation, no numbering.
```
