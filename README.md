# postgres-deadlock-agent-lora

## POV
So this repo is basically me figuring out how to fine-tune a the model the dadlock agent uses.

The deadlock agent works. It collects real data from pg_locks, ships it to the LLM (Mistral), gets back a diagnosis a junior engineer could act on. 

Follow-up: The reasoning needs to more specific.  In this case, specific means better diagnostics around the dealock but could drill down in other areas; based on the input dataset.

Right now the agent calls Mistral like it's paging an oncall senior DBA...it is still expensive in context, slow on inference, and overkill when 80% of the deadlocks in monolith apps are the same three or four patterns. 
LoRA fine-tuning is current solution implemented here. Take a small base model, freeze most of the weights, and train a lightweight adapter on domain-specific examples: deadlock scenarios, lock graphs, etc. The model now does reason for the specific problem. If the deadlock agent is the observation layer, this is the specialization layer.
The training data comes from the same pattern the agent already runs: deterministic collection, LLM reasoning,  gradient output. Using large models (Claude, GPT, Gemini) to generate training pairs. Here it requires manual (human SME) to decides if the diagnosis is right or needs adjusment. The small model learns from the SME graded output. Teacher, student distillation, nothing exotic.

This runs on MLX on Apple Silicon MacBook

## Background

If you want to see the actual app this is connected to, it is at  [postgres-deadlock-agent](https://github.com/danabindra/postgres-deadlock-agent)  it's a live agent that watches a PostgreSQL database, detects deadlocks, and asks to approve the fix before it runs. The LoRA repo is the next layer on top of that: instead of just sending raw evidence to a generic LLM, 

---

## Contents

| File / Folder | Summary |
|---|---|
| `train.jsonl` | 350 labelled instruction pairs, deadlock scenarios as input, expert diagnosis as output |
| `valid.jsonl` | examples to check if the adapter is actually learning |
| `postgres_deadlock_training_examples_50.jsonl` | raw 50 postgres examples the training data is built from |
| `bible-addendum-training-data.md` | Notes on how the training data ws generated and the prompt template  |
| `adapters/` | LoRA weights (tracked via Git LFS) |
| `deadlock-agent-animated.svg` | architecture diagram showing how everything connects |

---

## Training

Model: `mlx-community/Mistral-7B-Instruct-v0.3-4bit` runs locally on M4 via MLX.

Config is pretty minimal, rank 8, 100 iterations, learning rate 1e-5, batch size 1. No benchmarks yet, just learning the mechanics of what changes when you fine-tune vs when you don't.

The training data was generated using Claude (Opus) as the "teacher", one prompt, 50–100 pairs at a time. The key thing (which is in `bible-addendum-training-data.md`) is that the LLM writes the bulk content I can decide what scenarios to cover. Edge cases can cause blind spots.

---

## Future: LoRA and RAG refinement (separate repo will link here when done)

**LoRA and RAG together.** Right now the adapter just bakes deadlock knowledge into the weights. But I want to test combining it with retrieval, pull in live pg_locks / pg_stat_activity data at inference time, let the fine-tuned model reason over it. 

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
