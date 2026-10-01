Forge is an AI computation and cost optimizer.

It determines:

“How much reasoning does this problem actually need?”

For example:

Easy problem → 1 model loop → low cost
Medium problem → 3 loops
Hard problem → 6 loops
Still uncertain → call a stronger, expensive model
Budget exhausted → abstain instead of wasting money

Forge then records:

Computation used
GPU time
Response latency
API/token cost
Whether the answer was correct
Cost per correct answer

But it does more than calculate cost: it actively controls and optimizes computation.

Problem
→ estimate difficulty
→ allocate reasoning compute
→ verify answer
→ measure cost and accuracy
→ improve future allocation

So the best simple description is:

Forge learns how to spend the minimum AI computation needed to produce a correct answer.

Its key goal is not simply “use less compute.” It is:

Achieve the highest possible accuracy within a fixed cost or latency budget.


1) Looped transformer )(studies show benefits on specific reasoning tasks, not guaranteed superiority overall than normal trasnformers)

A looped transformer reuses the same block of layers multiple times.

Input → Shared block → Shared block → Shared block → Output
                  same weights reused

Instead of building 12 different layers, Forge could build a 4-layer block and run it three times.

Important distinction

The model does not gain new parameters every time the block repeats. It reuses the same learned weights but spends additional computation.

This is similar to rereading and reconsidering a problem using the same brain.

Limitation

Reusing weights does not guarantee better reasoning. The model could:

Stop improving
Repeat the same error
Become unstable
Change a correct answer into an incorrect one

A conditioning signal can also tell the model:

How much budget remains
How large the current reasoning step is
Whether it should refine or prepare an answer

LoopFormer, for example, conditions its loop steps on internal time and step size so the model can use different computation schedules

Without residual connections, repeated loops may erase useful information or become unstable.

2) Normalization

Normalization keeps numerical values inside the model within manageable ranges.

Repeated blocks can cause values to:

Grow too large
Become too small
Drift over successive passes

Normalization helps keep the loop stable.

3) a) Intermediate prediction head

A prediction head converts the current hidden state into a possible answer. These intermediate answers help measure whether the model is improving.

3 b) Explicit chain of thought

Explicit chain of thought is reasoning generated as visible text before the final answer. It provides a baseline. Without this comparison, Forge cannot know whether looping provides a special benefit.

3 c) Latent reasoning & Explicit reasoning

Producing reasoning tokens one by one is slow. Hidden-state computation may carry more information per step and avoid generating long explanations.

3 d) Intermediate supervision

Supervision means showing the model what it should learn during training. Intermediate supervision can also teach what useful internal states should represent at different passes.

3 e) Representation collapse

Representation collapse occurs when repeated passes stop producing meaningfully different internal states.

Pass 1 state ≈ Pass 2 state ≈ Pass 3 state

The model is technically looping but no longer refining anything. No accuracy/gradient/prediction changes or similar hidden states mean model is stuck.

3 f) Adaptive reasoning

Adaptive reasoning means allocating different amounts of computation to different problems. Always using maximum computation wastes time and money. Always using minimum computation reduces accuracy.

Forge attempts to find a better accuracy–cost tradeoff.

4) Task features

Task features are compact facts used to estimate how much reasoning a problem needs.

Possible features include:

Task type
Input length
Estimated difficulty
Current loop count
Model confidence
Disagreement between intermediate predictions
Remaining time or compute budget

5) Router

The router decides what should happen next.

Its possible decisions are:

Stop: Accept the current answer.
Continue: Give the model another loop.
Escalate: Send the task to a stronger model.
Abstain: Admit that the system cannot answer reliably.

Heuristic router

A heuristic router uses manually written rules.
Why build this first

It is:

Easy to understand
Cheap
Reproducible
A strong baseline against which Jev must compete

6) Jev

Jev is an external decision model from TypeSafe AI. It is designed to receive unstructured state and return typed decisions with probabilities.

The routing task needs:

A small set of valid decisions
Probabilities
Low latency
Consistent schemas

It does not require a long natural-language response.

Important limitation

Typed output prevents malformed output; it does not guarantee a correct decision.




