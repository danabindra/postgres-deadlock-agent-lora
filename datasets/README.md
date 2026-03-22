# datasets/

Generated JSONL training data for each of the 5 monolith issue types.
Each subfolder maps to the prompt in `../prompts/` with the same number.

Drop the raw LLM output into the matching folder.
Convention: `[source]-[version].jsonl` e.g. `claude-v1.jsonl`, `gemini-v1.jsonl`, `gpt-v1.jsonl`

| Folder | Issue type | Prompt |
|---|---|---|
| `deadlocks/` | PostgreSQL deadlocks (expanded) | `prompts/01-deadlocks.md` |
| `slow-queries/` | Slow queries and N+1 problems | `prompts/02-slow-queries-n-plus-one.md` |
| `connection-pool/` | Connection pool exhaustion | `prompts/03-connection-pool-exhaustion.md` |
| `memory-leaks/` | Memory leaks and heap growth | `prompts/04-memory-leaks-heap-growth.md` |
| `cascading-failures/` | Cascading failures from tight coupling | `prompts/05-cascading-failures.md` |

Once you have files from multiple sources, combine them into `train.jsonl` and `valid.jsonl`
at the root level for the actual fine-tuning run.
