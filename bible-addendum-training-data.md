## TRAINING DATA GENERATION (added during Demo 2)

### How to Generate JSONL Training Data

Use a large LLM (Claude, GPT-5, Opus) as the teacher. One prompt generates
50-100 training pairs. The quality depends entirely on prompt specificity.

**The prompt template:**

"Generate [N] [domain] diagnostic training examples in JSONL format. Each line
should have a messages array with a user role and an assistant role. The user
message should simulate the output of [your agent's tools]: [list exact fields
and format]. The assistant message should be an expert diagnosis that [list
exactly what the diagnosis should contain]. Cover these scenarios: [list every
scenario type the model needs to handle]."

**Key principle:** The domain expert decides what scenarios to include. The LLM
generates the bulk content. The human grades the quality. This is teacher-student
distillation — the same pattern Mike uses in his training harness.

**Three sources of training data:**
1. Large LLM generates it (fast, scalable, needs review)
2. Real production data captured from your agent (highest quality, slowest)
3. Domain expert writes by hand (most accurate, most expensive)

Best approach: combine all three. LLM generates 80%, production data fills gaps,
domain expert reviews everything.

**If you don't know what scenarios to ask for, your model has blind spots.**
This is why the domain expert (you) is the moat. Claude writes the data.
You decide what data to write.
