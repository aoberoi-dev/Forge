LLM Cost and Compute Optimisation Notes
1. RouteLLM
GitHub

RouteLLM is a framework for serving and evaluating LLM routers. It compares model performance against cost.

The router examines each query and chooses between:
- A smaller, cheaper model for simpler queries
- A stronger, more expensive model for difficult queries
Cost threshold
Each request has a threshold controlling the cost–quality tradeoff.
- Lower threshold → more strong-model calls → higher cost and usually better quality
- Higher threshold → more weak-model calls → lower cost but potentially lower quality
The threshold must be calibrated using queries similar to the application’s real workload.
Routers
- mf: Matrix-factorization router trained on preference data; recommended.
- sw_ranking: Uses similarity-weighted Elo scores from prompts similar to the current query.
- bert: BERT classifier trained on preference data.
- causal_llm: LLM-based classifier trained to select the appropriate model.
- random: Random routing; used as a baseline.
Benchmarks
- MMLU
- GSM8K
- MT-Bench
Key limitation
RouteLLM makes the routing decision before seeing the generated answer. It normally calls only one model.
2. FrugalGPT
GitHub

FrugalGPT provides strategies for reducing LLM API costs while maintaining acceptable performance.

Its three main techniques are:
1. Prompt adaptation
Reduce unnecessary input tokens while retaining important information.
- Select only the most relevant few-shot examples.
- Combine similar requests when they share the same instructions.
- Use a stronger model once to create an effective prompt that can later run on a cheaper model.
- Prompt compression methods such as LLMLingua are related extensions, not part of the original FrugalGPT implementation.
2. LLM approximation
Avoid unnecessary expensive model calls.
- Exact caching: Reuse the result when the same request appears again.
- Semantic caching: Reuse results for semantically similar requests.
- Fine-tuning: Train a smaller model for a narrow repeated task.
Semantic caching and fine-tuning require careful evaluation to prevent incorrect or stale responses.
3. LLM cascade
Query models sequentially, beginning with the cheapest.
Cheap model
→ score its answer
→ satisfactory? Return it
→ otherwise call a stronger model

Generation scoring function
g(q, a) gives a reliability score between 0 and 1 for answer a to query q.

The score determines whether to:
- Accept the answer
- Query the next model
- Stop because the budget is exhausted
Cascade router
The router learns:
- Which models to use
- In what order
- The acceptance threshold for each model
Key difference from RouteLLM
- RouteLLM: Selects one model before generation.
- FrugalGPT: Evaluates the generated answer and may call additional models.
3. LoopFormer
GitHub

LoopFormer is an elastic-depth looped transformer designed for budget-conditioned latent reasoning.

A standard transformer uses separate layers:
Layer 1 → Layer 2 → Layer 3

A looped transformer repeatedly uses shared transformer blocks:
Shared block → Shared block → Shared block

This is called parameter sharing or a recurrent/looped transformer.
Variable-length trajectories
LoopFormer is trained using different numbers of loops so the same model can work under different compute budgets.
Shortcut-consistency training
Shorter reasoning paths are trained to remain aligned with longer paths:
- Short paths should still produce useful representations.
- Longer paths should continue refining rather than drifting or collapsing.
Time and step-size conditioning
Each loop knows:
- Its current position in the reasoning process
- The size of the current reasoning step
This helps the shared block behave differently during different passes.
Budget-conditioned inference
At inference time, the user or external router selects compute budget M.
Small M → fewer loops, faster and cheaper
Large M → more loops, slower but potentially better

The model does not need to be retrained for each budget.
Elastic depth
The goal is one model that performs reliably across multiple depths without:
- Short-depth degradation
- Internal-state stagnation
- Representation collapse
- Later loops making answers worse
Important limitation
LoopFormer executes the selected budget but does not automatically determine the ideal budget for every query. Forge still needs a router to choose M.
Quick comparison
Method	Main decision
RouteLLM	Which model should receive the query?
FrugalGPT	Is the current answer good enough, or should another model be called?
LoopFormer	How much internal computation should one model use?
