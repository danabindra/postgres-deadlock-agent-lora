# Prompt 4: Memory Leaks and Heap Growth

Use this prompt with Claude, Gemini, or ChatGPT to generate memory leak training data.
Copy the block below and paste it directly.

---

```
Generate 60 application memory leak diagnostic training examples in JSONL format.
Each line must be a valid JSON object with a single "messages" key containing an array
with exactly two objects: one with "role": "user" and one with "role": "assistant".

These examples target backend monolith services written in Python, Node.js, Java, or Ruby.
The user message should simulate what an agent collects from: process memory stats over time,
GC logs or gc metrics, heap snapshots or object allocation summaries, and application logs.

Format the user message like this:
"Analyze this memory diagnostic data for [service-name] ([language/runtime]):

Process memory over time:
  T+0h:   RSS [N]MB | Heap used [N]MB | Heap total [N]MB
  T+2h:   RSS [N]MB | Heap used [N]MB | Heap total [N]MB
  T+6h:   RSS [N]MB | Heap used [N]MB | Heap total [N]MB
  T+12h:  RSS [N]MB | Heap used [N]MB | Heap total [N]MB
  T+18h:  RSS [N]MB | [OOM kill / restart / alert triggered]

GC activity:
  GC pause frequency: [N per minute]
  GC pause duration (avg): [Xms] | (max): [Xms]
  Objects surviving GC cycles: increasing / stable / decreasing

Top object allocations (heap snapshot diff T+0 vs T+6):
  [ClassName or type]: [N] instances (+[N] vs baseline)
  [ClassName or type]: [N] instances (+[N] vs baseline)
  [ClassName or type]: [N] instances (+[N] vs baseline)

Application logs (relevant excerpts):
  [Error or warning messages related to memory, e.g. 'heap out of memory', cache size warnings, etc.]

Recent deployments:
  [Timestamp]: [what changed]"

The assistant message must be an expert diagnosis that:
- Opens with one of: MEMORY LEAK CONFIRMED, HEAP FRAGMENTATION, GC PRESSURE — NOT A LEAK, CACHE UNBOUNDED GROWTH, or EXPECTED GROWTH — NOT A LEAK
- Identifies the specific source: which object type, which code pattern (event listener not removed, cache without TTL or size cap, closure holding reference, database cursor not closed, etc.)
- Explains why GC is not reclaiming the memory
- Gives an immediate fix (restart schedule, cache flush, emergency config change)
- Gives the long-term code fix with the specific pattern to change
- Notes whether the recent deployment correlates with the leak start

Cover ALL of these scenario types — distribute evenly across the 60 examples:
1. Node.js event listener added in a loop and never removed — classic leak
2. Python dictionary used as an in-memory cache with no size limit or TTL
3. Java heap fragmentation after long uptime — GC overhead increasing but no real leak
4. Unclosed database cursors or result sets holding references
5. Closure in an async callback capturing a large object that cannot be collected
6. Redis or Memcached client connection pool growing unbounded
7. Log buffer accumulating in memory instead of flushing to disk
8. Third-party library with known memory issue introduced in recent deployment
9. Ruby object retained in a class-level variable that accumulates across requests
10. False positive — memory growth is linear with traffic, not a leak, just needs more RAM

Output only raw JSONL — one JSON object per line, no markdown, no explanation, no numbering.
```
