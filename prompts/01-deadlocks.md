# Prompt 1: Database Deadlocks (Expanded Coverage)

Use this prompt with Claude, Gemini, or ChatGPT to generate expanded deadlock training data.
Copy the block below and paste it directly.

---

```
Generate 60 PostgreSQL deadlock diagnostic training examples in JSONL format.
Each line must be a valid JSON object with a single "messages" key containing an array
with exactly two objects: one with "role": "user" and one with "role": "assistant".

The user message should simulate the exact output an agent collects from these PostgreSQL
tools: pg_locks, pg_stat_activity, pg_stat_database, and the deadlock history counter.

Format the user message like this:
"Analyze this PostgreSQL deadlock diagnostic data:

Blocked sessions: [N]
Session 1 (PID [X]): [SQL query] — waiting for lock on [table]
Session 2 (PID [Y]): [SQL query] — waiting for lock on [table]
[Add Session 3 only for 3-way deadlocks]
Lock graph: PID [X] holds [LockType] on [table], wants [LockType] on [table]. PID [Y] holds [LockType] on [table], wants [LockType] on [table].
Deadlock history: [N] occurrences in last 24 hours [between which tables, and when pattern started if relevant]."

The assistant message must be an expert DBA diagnosis that:
- Opens with one of: DEADLOCK CONFIRMED, DEADLOCK OCCURRED BUT ALREADY RESOLVED, NOT A DEADLOCK, or ADVISORY LOCK DEADLOCK
- Explains the root cause in plain English (no jargon without explanation)
- Identifies whether it is a one-time race or a recurring application bug
- Gives an immediate fix (which PID to terminate and why)
- Gives a long-term fix (code-level change or schema change)
- Is written at the level of a junior-to-mid engineer who needs to act on it

Cover ALL of these scenario types — distribute evenly across the 60 examples:
1. Classic 2-session lock ordering violation (orders + inventory pattern)
2. Classic 2-session lock ordering violation (accounts + transactions pattern)
3. 3-session deadlock chain (primary deadlock + secondary victim)
4. Deadlock that already self-resolved — no currently blocked sessions but counter incremented
5. False positive — lock wait, NOT a deadlock (one session waiting, other not blocked)
6. Foreign key constraint deadlock (parent/child row locking conflict)
7. Advisory lock deadlock (pg_advisory_lock misuse)
8. Deadlock triggered by trigger or stored procedure
9. Deadlock in a high-frequency pattern (20+ occurrences in 24 hours) indicating a deployment bug
10. First-ever deadlock — 1 occurrence, isolated, no pattern

Output only raw JSONL — one JSON object per line, no markdown, no explanation, no numbering.
```
