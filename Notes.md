Milestone 1: Evaluation foundation
- Repository and Python package
- Task data structure
- One task generator
- Independent answer verifier
- Train/validation/hidden-test splits
- Evaluation runner
- Machine-readable results
- Human-readable scorecard
- Reproducibility using seeds and versions
Milestone 2: Fixed model baseline
- Model interface
- Small existing model
- Direct-answer baseline
- Fixed reasoning-budget baseline
- Accuracy, compute, latency and cost measurements
Milestone 3: Looped transformer
- Shared transformer block
- Multiple internal passes
- Intermediate prediction heads
- Accuracy after each pass
- Parameter-matched and compute-matched comparisons
Milestone 4: Adaptive router
- Fixed-depth policy
- Rule-based heuristic
- Stop/continue/escalate/abstain decisions
- Confidence calibration
- Jev integration
- Cost-versus-accuracy comparison
Milestone 5: Bounded improvement loop
- Failure analyzer
- Experiment proposal agent
- Restricted configuration changes
- Frozen evaluation gate
- Human approval
- Immutable experiment ledger
- Random-proposal control
Milestone 6: Product demonstration
- FastAPI inference service
- Minimal UI
- Experiment dashboard
- Reproduction instructions
- Ablation report
- Technical paper
