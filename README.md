# postgres-deadlock-agent-lora

So this repo is basically me figuring out how to fine-tune a model.

If you want to see the actual app this is connected to, it is at  [postgres-deadlock-agent](https://github.com/danabindra/postgres-deadlock-agent)  it's a live agent that watches a PostgreSQL database, detects deadlocks, and asks to approve the fix before it runs. The LoRA repo is the next layer on top of that: instead of just sending raw evidence to a generic LLM, want to see what happens when you fine-tune a smaller model specifically for this problem.

---

## Contents

| File / Folder | What it is |
|---|---|
| `train.jsonl` | 350 labelled instruction pairs, deadlock scenarios as input, expert diagnosis as output |
| `valid.jsonl` | examples to check if the adapter is actually learning |
| `postgres_deadlock_training_examples_50.jsonl` | The raw 50 postgres examples the training data is built from |
| `bible-addendum-training-data.md` | Notes on how the training data ws generated and the prompt template  |
| `adapters/` | The actual LoRA weights (tracked via Git LFS) |
| `deadlock-agent-animated.svg` | architecture diagram showing how everything connects |

---

## Training

Model: `mlx-community/Mistral-7B-Instruct-v0.3-4bit` runs locally on M4 via MLX.

Config is pretty minimal, rank 8, 100 iterations, learning rate 1e-5, batch size 1. Not trying to win any benchmarks here, just learning the mechanics of what changes when you fine-tune vs when you don't.

The training data was generated using Claude (Opus) as the "teacher", one prompt, 50–100 pairs at a time. The key thing (which is in `bible-addendum-training-data.md`) is that the LLM writes the bulk content I can decide what scenarios to cover. Edge cases can cause blind spots.

---

## Up next

**LoRA + RAG together.** Right now the adapter just bakes deadlock knowledge into the weights. But I want to test combining it with retrieval, pull in live pg_locks / pg_stat_activity data at inference time, let the fine-tuned model reason over it. 

 RAG handles the "right now" part and the LoRA is "diagnose and explain it" part. TBD

---

## Training data update coming

I'm also going to redo the training data generation. Claude did the first pass, but I want to try Gemini or ChatGPT next to see if there's a noticeable quality difference in examples. Probably run all three and compare.

Also, going to expand the problem scope. Right now everything is deadlock-specific. The next dataset is going to cover the **top five issues that monolithic apps run into**:

1. **Database deadlocks** —
2. **Slow queries and N+1 problems**
3. **Connection pool exhaustion**
4. **Memory leaks and heap growth**
5. **Cascading failures tight coupling** 

Goal is a single fine-tuned model that can look at diagnostics from any of these and give an answer, 



## Running the adapter locally

 [MLX](https://github.com/ml-explore/mlx) and [mlx-lm](https://github.com/ml-explore/mlx-lm):

```bash
pip install mlx-lm
```

Then test the adapter:

```bash
python -m mlx_lm.generate \
  --model mlx-community/Mistral-7B-Instruct-v0.3-4bit \
  --adapter-path adapters \
  --prompt "Analyze this PostgreSQL deadlock diagnostic data: ..."
```

---

Still very much a work in progress.
