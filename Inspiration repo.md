1) RouteLLM: https://github.com/lm-sys/RouteLLM

RouteLLM is a framework for serving and evaluating LLM routers.
Compares Performance with cost
Each routing request has a cost threshold that controls the tradeoff between cost and quality. We should calibrate this based on the types of queries we receive to maximize routing performance
We introduce a router that looks at queries and routes simpler queries to smaller, cheaper models, saving costs while maintaining quality. We focus on routing between 2 models: a stronger, more expensive model and a cheaper but weaker model. Each request is also associated with a cost threshold that determines the cost-quality tradeoff of that request - a higher cost threshold leads to lower cost but may lead to lower-quality responses.

Routers:
The full list of routers:

mf: Uses a matrix factorization model trained on the preference data (recommended).
sw_ranking: Uses a weighted Elo calculation for routing, where each vote is weighted according to how similar it is to the user's prompt.
bert: Uses a BERT classifier trained on the preference data.
causal_llm: Uses a LLM-based classifier tuned on the preference data.
random: Randomly routes to either model.

Benchmarks router: We currently support: mmlu, gsm8k, and mt-bench

The threshold used for routing controls the cost-quality tradeoff. The range of meaningful thresholds varies depending on the type of router and the queries you receive
