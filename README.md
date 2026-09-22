# Forge
Adaptive Reasoning Lab

An experimental system that learns how much computation a small model should spend on each problem, then uses measured results to improve that decision policy. It combines looped transformers, latent reasoning, a structured decision model (Jev), and bounded recursive self-improvement (RSI) in one reproducible research project.

Research question: At a fixed compute or latency budget, can adaptive internal reasoning plus evaluation-guided policy improvement solve more problems than fixed-depth inference?

Deliverable: An open-source repository with a working inference service, trained small looped model, evaluation harness, experiment dashboard, documented ablations, and a short technical report. Success is a measured improvement on held-out tasks, even if the hypothesis is rejected; it is not a claim of AGI.

1. How the system works

Problem → task features → depth/router decision → model inference → answer
                                  ↓                  ↓
                         more loops / escalate   verifier + telemetry
                                  ↓                  ↓
                           final result       experiment dataset
                                                       ↓
                                          propose policy change → offline evaluation → promote or reject

Looped transformer: A small transformer reuses a block of layers for several passes. The number of passes can vary per problem. Each pass changes hidden states; it is not an extra chat message.

Latent reasoning: The repeated computation occurs in hidden states without requiring a written chain of thought. Whether this helps is an experimental question, assessed against parameter- and compute-matched baselines.

Jev: A separate decision service receives compact task features and permitted choices such as stop, continue, escalate, or abstain. It returns a typed probabilistic judgment. Do not assume its confidence is calibrated on this benchmark; measure it. Jev is an optional dependency with a rule-based fallback.

Bounded RSI: An agent proposes edits to routing thresholds, budgets, data mixes, or training configurations. A fixed verifier and held-out evaluation gate accept or reject changes. The agent cannot change hidden labels, evaluator code, or promotion criteria.

Important distinction: A model reusing layers at inference time is not recursive self-improvement. RSI here refers to the research workflow accumulating experimentally validated changes across iterations.

2. Scope and evaluation contract

Start with two objectively gradable domains:

Algorithmic reasoning: arithmetic, symbolic manipulation, graph paths, and program-trace questions with deterministic answers.

Small coding tasks: functions with unit tests and bounded runtime. Keep these separate from the classification tasks; code execution and scoring require a different verifier.

Prefer generated training tasks with controlled difficulty. Use independently written, frozen held-out sets for evaluation; ensure no template or near duplicate appears in training. Keep public benchmark results as a secondary, disclosed measure because of contamination risk.

For each run record: task ID and difficulty, split and data version, model checkpoint, router version, loop count, input/output length, correctness, latency distribution, GPU seconds or FLOPs estimate, API cost, tool calls, and error category. Fix a maximum loop count, time budget, and escalation budget before comparing methods.

Primary metric: correctness at equal average compute, with a latency ceiling. Secondary metrics: accuracy by difficulty, calibration, cost per correct answer, p50/p95 latency, abstention quality, and improvement on a frozen holdout after each promoted revision. Publish uncertainty intervals over multiple seeds.

3. Phases

Phase 0 — Reproducible harness and data

Build

Repository, typed configuration, experiment IDs, deterministic seeds, containerized runners, task generators, verifiers, data split manifest, and results store.

A command that evaluates a model and emits a machine-readable report and a human-readable scorecard.

At least three difficulty levels and a small manually audited sample of generated tasks.

Exit gate: Running the same checkpoint and seed reproduces the same task scores; incorrect answers are rejected by independent verifiers. Establish hardware, time, and API cost budgets. Do this before training anything.

Phase 1 — Fixed-compute baseline

Build

One existing small open model or a tiny transformer trained on the task family.

Baselines: direct answer, fixed token-based reasoning budget, and a simple difficulty heuristic for escalation.

Evaluation across three seeds, measured on the same held-out examples and budgets.

Exit gate: Publish a baseline table of quality, compute, and latency. Save per-task predictions so later variants can be compared pairwise.

Phase 2 — Looped transformer

Build

Repeated application of shared transformer blocks with a loop index or conditioning signal, stable residual/normalization design, and a configurable maximum pass count.

Intermediate probes or heads for answer prediction and confidence after each pass; choose the most appropriate supervision scheme in pilot experiments.

Training and inference scripts, checkpointing, and an explicit stop rule for maximum loops.

Experiments: one versus multiple passes; trained depth versus unseen depth; shared-loop model versus parameter-matched and FLOP-matched ordinary transformers. Measure representation collapse and whether more passes actually help difficult tasks.

Exit gate: Reproduce results across seeds and document both wins and regressions. If looping provides no benefit, retain that negative result and proceed with an honest comparison.

Phase 3 — Latent reasoning study

Build

Let successive hidden states carry refinement while emitting only the final answer. Optionally use intermediate auxiliary supervision during training; remove it at inference.

An explicit-text baseline with an equalized inference budget.

Diagnostics: accuracy by pass, change in predictions across passes, early-correct-then-wrong cases, and hidden-state probes that do not themselves become the production reasoning path.

Exit gate: Show whether internal passes add useful computation beyond simply using more parameters, more generated tokens, or more training data. Do not label hidden activations as interpretable thoughts without evidence.

Phase 4 — Adaptive routing with Jev

Build

A router using only features available at decision time: task type, length, difficulty estimates, intermediate confidence or disagreement, remaining budget, and loop count.

A typed decision schema with stop, continue, escalate, and abstain; apply hard budgets in ordinary code.

Jev adapter, local heuristic fallback, and a stronger-model escalation adapter. Keep provider keys server-side.

Experiments: fixed depth, heuristic router, Jev router, and (optionally) learned local router on identical task sets. Compare confidence calibration, decision errors, added latency, costs, and correctness at equal budget. Do not claim Jev is superior unless this comparison supports it.

Exit gate: A deployable router that obeys budgets and degrades gracefully if an external API fails.

Phase 5 — Bounded self-improvement research agent

Build

Agent reads aggregate failures and sanitized examples, writes a hypothesis and a proposed change in a constrained config or experiment branch, then runs a validation set.

Promotion gate checks frozen holdout results, regression slices, budgets, statistical uncertainty, and experiment reproducibility. Human review controls promotion initially.

Immutable experiment ledger: parent revision, proposal, changed files/config, run IDs, scores, cost, and acceptance reason.

Experiments: static policy versus self-improving policy across multiple rounds, equalizing total experiment compute. Include a placebo agent that proposes random allowable changes. Retain failures in the report to avoid selective reporting.

Exit gate: Demonstrate retained gains on a holdout the proposing agent has not inspected, or report clearly that the improvement loop did not outperform a static policy. Calling repeated prompt tweaks “RSI” alone does not meet this gate.

Phase 6 — System demonstration and paper-style report

Build

Minimal UI: enter a problem; view answer, chosen loop budget, router decision, actual latency, and whether an independent verifier accepted the answer.

Dashboard: Pareto plot of accuracy versus compute/latency, accuracy by difficulty, per-pass outcomes, router confusion matrix, and improvement history.

Technical report: hypothesis, dataset construction, experimental controls, baselines, all ablations, confidence intervals, failure analysis, compute cost, and limitations.

Reproduction instructions with pinned versions, data/checkpoint availability, and one command per main figure.

Exit gate: Another engineer can rerun the main claims with the released artifacts and identify which outcomes depended on external APIs.

4. Proposed repository layout

adaptive-reasoning-lab/
  README.md
  configs/                 # budgets, model and experiment configurations
  data/                    # generators and split manifests; no secrets
  models/                  # baseline and shared-block looped model
  routers/                 # heuristic, Jev and optional local router
  verifiers/               # isolated exact-answer and code-test verifiers
  research_agent/          # hypothesis and bounded proposal loop
  eval/                    # metrics, paired comparisons and report generation
  service/                 # inference API and optional UI
  experiments/             # immutable configs and result summaries
  tests/                   # verifier and budget-enforcement tests
  reports/                 # figures and technical report

5. Recommended implementation choices

Python + PyTorch for model research; small FastAPI service for the demo; SQLite or PostgreSQL for experiment records; lightweight plotting for the report.

Use a hosted strong model only for escalation and research proposals. Run training and the primary benchmark on controllable models so cost and reproducibility remain measurable.

Start with a small looped model and short synthetic tasks. Scale only after baseline and harness results justify it. A modest local GPU or rented single GPU can support the pilot; exact memory needs depend on model size, sequence length, batch size, and optimizer.

Track checkpoints, datasets, configurations, seeds, and code commits together. Profile real GPU time and memory rather than inferring speed from parameter count.

6. Core ablation matrix

Question

Controlled comparison

Does looping help?

Shared-loop model vs parameter-matched and compute-matched standard model

Is compute allocation useful?

Fixed loop count vs heuristic vs Jev at matched average cost

Is the benefit latent computation?

Internal passes vs explicit generated steps at matched budget

Does the improvement loop work?

Frozen initial policy vs accepted revisions vs random-proposal control

Does routing generalize?

In-domain tasks vs unseen difficulty and task templates

Does escalation pay off?

Quality and total cost with vs without stronger-model calls

7. Risks and controls

Evaluation leakage: freeze the holdout before agent proposals; do not expose individual hidden labels to the proposal agent.

Reward hacking: independent verifiers and review of samples; track regressions and cost rather than optimizing one score alone.

Compute confounds: compare under both matched parameters and matched computation; report all training and inference costs.

Overthinking: monitor cases where extra passes change a correct answer to a wrong one; cap loops.

Confidence misuse: calibrate on a dedicated validation split and measure selective accuracy; a probability is not a correctness guarantee.

External dependency drift: pin Jev model/API versions where supported, log responses and costs, and maintain the local router baseline.

Code execution: sandbox coding tasks with time, memory, process, and network limits; use vetted fixtures.

8. Practical milestones

Milestone

Demoable result

M1: Harness

Reproducible dataset, verifier, baseline scorecard

M2: Looped model

Accuracy-by-pass curve and controlled standard-model comparison

M3: Adaptive router

Quality/compute frontier comparing fixed, heuristic, and Jev

M4: Research loop

Auditable proposal → experiment → acceptance history

M5: Release

Working app, ablation report, reproducibility instructions

9. First implementation sprint

Define one exact-answer task family and a generator with training, validation, and hidden test splits.

Implement an independent answer verifier and budget-aware evaluation runner.

Run a small existing model with fixed budgets; record latency, correctness, and cost.

Plot correctness by difficulty and an accuracy-versus-compute baseline.

Freeze the data split and evaluation contract before designing the looped model.

First milestone definition of done: evaluate creates a reproducible scorecard and task-level results from a pinned model, dataset version, and seed. That is the foundation for every later claim.

10. Research references

TypeSafe AI, Introducing System One Models & Jev: https://typesafe.ai/blog/introducing-system-one-models-and-jev

LoopFormer, Elastic-Depth Looped Transformers for Latent Reasoning: https://loopformer.github.io/

Microsoft Research, Bridging the Gap Between Latent and Explicit Reasoning with Looped Transformers: https://www.microsoft.com/en-us/research/publication/bridging-the-gap-between-latent-and-explicit-reasoning-with-looped-transformers/

Recursive Self-Improvement in AI: From Bounded Self-Refinement to Autonomous Research Loops: https://arxiv.org/abs/2607.07663
