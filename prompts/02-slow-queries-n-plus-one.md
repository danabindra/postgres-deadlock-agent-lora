# Prompt 2: Slow Queries and N+1 Problems

Use this prompt with Claude, Gemini, or ChatGPT to generate slow query / N+1 training data.
Copy the block below and paste it directly.

---

```
Generate 60 PostgreSQL slow query diagnostic training examples in JSONL format.
Each line must be a valid JSON object with a single "messages" key containing an array
with exactly two objects: one with "role": "user" and one with "role": "assistant".

The user message should simulate the exact output an agent collects from these tools:
pg_stat_statements (for top slow queries), EXPLAIN ANALYZE output, pg_stat_user_tables
(for sequential scans), and application-level query logs showing repeated queries.

Format the user message like this:
"Analyze this PostgreSQL slow query diagnostic data:

Top slow queries (pg_stat_statements):
  Query: [SQL, truncated to key structure]
  Calls: [N] | Mean time: [Xms] | Total time: [Xms] | Rows: [N]
  [Repeat for 2-3 queries if relevant]

EXPLAIN ANALYZE output:
  [Key lines from explain plan: Seq Scan vs Index Scan, actual rows vs estimated, cost]

Sequential scan alerts:
  Table: [name] | Seq scans: [N] in last hour | Live rows: [N]

Application query log sample (last 60 seconds):
  [Show repeated queries that indicate N+1, e.g. SELECT * FROM users WHERE id=X repeated 47 times]"

The assistant message must be an expert diagnosis that:
- Opens with one of: N+1 QUERY DETECTED, MISSING INDEX, SEQUENTIAL SCAN ON LARGE TABLE, SLOW QUERY — PLAN REGRESSION, or QUERY PERFORMING AS EXPECTED (false positive case)
- Explains exactly what is happening and why it is slow
- Identifies whether it is an ORM issue, missing index, bad query plan, or data volume problem
- Gives the immediate mitigation (EXPLAIN options, index hint, query rewrite)
- Gives the long-term fix with the exact SQL or ORM change needed
- Estimates the performance improvement expected

Cover ALL of these scenario types — distribute evenly across the 60 examples:
1. Classic N+1: ORM loads a list then queries each record individually (Rails/Django/Hibernate pattern)
2. Missing index on a foreign key column causing full table scan
3. Index exists but planner chooses sequential scan due to stale statistics (needs ANALYZE)
4. Query plan regression after a data volume increase (plan was fine at 10k rows, breaks at 1M)
5. SELECT * in a hot loop pulling columns that are never used
6. Unindexed ORDER BY on a large table causing sort spill to disk
7. LIKE '%keyword%' query that cannot use an index (needs full text search)
8. JOIN on mismatched data types causing implicit cast and index skip
9. Subquery running once per row instead of as a single set operation
10. False positive — query is slow but correct, data volume is the only fix

Output only raw JSONL — one JSON object per line, no markdown, no explanation, no numbering.
```
