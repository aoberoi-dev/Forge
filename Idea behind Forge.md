Pipeline: 

Query
  ↓
1. Cache lookup
  ↓ cache miss
2. Query analysis
  ↓
3. Select model + compute budget
  ↓
4. Adapt prompt for selected model
  ↓
5. Run model/LoopFormer
  ↓
6. Verify and score answer
  ↓
Accept OR increase compute OR cascade to stronger model OR abstain
  ↓
Record quality, cost and latency

1) For cache lookup: Exact cache: Identical request
Semantic cache: Meaningfully similar request

The cache key should include the prompt/template version, model version and relevant context. Avoid semantic caching for dynamic, personalized or time-sensitive questions.
Technology: Redis + RedisVL, which supports similarity thresholds, TTLs and metadata filters for semantic caching.

2) Query analysis
Extract features before selecting the model:
- Task type
- Input length
- Estimated difficulty
- Whether an objective verifier exists
- Required output format
- Time and cost limits
- Similar historical tasks

{
  "task_type": "coding",
  "difficulty": "medium",
  "verifier": "unit_tests",
  "latency_limit_ms": 3000,
  "maximum_cost": 0.02
}

3. Joint model and compute-budget selection
This should eventually become Forge’s main contribution.

Instead of deciding only the model, select a combined action:
Model + reasoning budget

Example actions:
Small LoopFormer + 2 loops
Small LoopFormer + 6 loops
Medium model + normal reasoning
Strong API model + high reasoning
Abstain

Initially, use clear rules:
Easy → small model, 2 loops
Medium → small model, 6 loops
Hard → strong model

Once we have enough experimental data, train a router to make this decision.
4. Prompt adaptation
Prompt adaptation should happen after model selection, because different models may need different prompting styles.

Possible adaptations:
- Select only relevant few-shot examples.
- Remove redundant instructions.
- Use a strict structured-output schema.
- Batch compatible requests.
- Use a reusable prompt optimized offline by a stronger model.
Do not compress prompts blindly. Removing important constraints may save tokens while reducing correctness.
5. LoopFormer execution
LoopFormer runs with the budget selected for that query:
Budget M=2 → fewer internal passes
Budget M=6 → more internal passes

Important correction:
LoopFormer can only be applied to a model whose architecture and inference code we control.

We cannot add LoopFormer loops internally to closed APIs such as ChatGPT or Claude.

For external models, we can only control:
- Which model is called
- Prompt size
- Output-token limit
- Available reasoning-effort setting
- Whether another call is made
6. Answer verifier/scorer
This is mandatory.

Use the strongest available verification method:
1. Deterministic verifier
2. Unit tests or executable checks
3. Known reference answer
4. Trained scoring model
5. LLM judge
6. Model’s own confidence—weakest option
Examples:
Math → calculate answer independently
Code → run unit tests
Classification → compare with label
SQL → execute query against test database
Open-ended answer → rubric-based judge

Self-reported confidence should never be the only reason to accept an answer.
7. LLM cascade
If the answer is not verified:
Current answer fails
→ use additional compute or stronger model
→ verify again

Possible cascade:
Small LoopFormer at M=2
→ fail
→ Small LoopFormer at M=6
→ fail
→ Strong external model
→ fail
→ abstain

Important efficiency issue
If LoopFormer cannot continue from the existing hidden state, rerunning at M=6 after completing M=2 repeats computation.

We must test two options:
- Preserve the intermediate hidden state and continue.
- Select the full budget initially and escalate directly to a stronger model after failure.
8. Record every outcome
For every request, save:
- Query features
- Cache hit/miss
- Prompt strategy
- Selected model
- Loop budget
- Tokens
- Cost
- Latency
- Verifier result
- Cascade steps
- Final correctness
- Reason for stopping
Technology: MLflow can track experiments, evaluations, traces, latency and token usage. MLflow AI Platform
Recommended improvement: Joint Budget Policy
The most interesting Forge idea is a router that chooses both:
\[
\text{Action} = (\text{model},\ \text{compute budget})
\]
It optimises a reward such as:
\[
\text{Reward}
=
\text{Correctness}
-
\lambda(\text{Cost})
-
\mu(\text{Latency})
\]
Where:
- \(\lambda\) controls how strongly we penalize cost.
- \(\mu\) controls how strongly we penalize latency.
Example:
Action	Correct	Cost	Latency	Result
Small model, 2 loops	No	Low	Low	Insufficient
Small model, 6 loops	Yes	Medium	Medium	Best
Strong model	Yes	High	High	Correct but wasteful


After collecting data, we could train this as a contextual-bandit routing problem:
Context: query features
Actions: model + budget choices
Reward: verified quality − cost − latency

This would improve on RouteLLM because it selects more than just “weak versus strong.” It also controls internal compute.
Recommended technology stack
Purpose	Technology
Main implementation	Python
Model training	PyTorch + LoopFormer
Initial router	Rules, then scikit-learn/XGBoost
Model API gateway and cost tracking	LiteLLM
Exact/semantic cache	Redis + RedisVL
Verification	Custom deterministic verifiers + pytest
Experiment tracking	MLflow
API	FastAPI
Data/config validation	Pydantic
Experiment database	SQLite initially, PostgreSQL later


LiteLLM can normalize model-provider calls and track model spend, while MLflow handles the research experiments and complete traces. liteLLM
Build order
1. Build the evaluation harness and verifier.
2. Add fixed-model and fixed-budget baselines.
3. Add exact caching.
4. Add prompt adaptation.
5. Add RouteLLM-style model selection.
6. Add FrugalGPT-style verification and cascading.
7. Integrate LoopFormer with fixed budgets.
8. Learn the joint model-and-budget policy.
9. Add conservative semantic caching.
10. Compare everything at equal quality, cost and latency.

Use a fixed benchmark dataset with known answers.

For every query, record:
- Answer correctness/quality
- Model used
- Loop count
- Cache hit/miss
- Total tokens and API cost
- Latency
- Escalations
- Abstention
Compare these strategies on the same queries:
1. Strong model only
2. Cheap model only
3. RouteLLM routing
4. FrugalGPT cascade
5. Fixed LoopFormer depth
6. Complete Forge optimizer
Main result:
Verified accuracy at equal cost, plus cost per correct answer.

Also report:
- p50/p95 latency
- Total cost savings
- Accuracy by difficulty
- Router/escalation accuracy
- Multiple-run confidence intervals
Start with GSM8K for math, HumanEval for code and MMLU for knowledge, then add a small custom hidden dataset to test generalization.
