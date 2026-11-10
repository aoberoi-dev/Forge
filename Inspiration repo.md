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


2) FrugalGPT: https://github.com/stanford-futuredata/FrugalGPT

FrugalGPT offers a comprehensive set of strategies for optimizing LLM API usage while reducing costs and maintaining high performance. By implementing techniques such as prompt adaptation, LLM approximation, and LLM cascade, developers and businesses can significantly reduce their LLM operating expenses without compromising on the quality of their AI-powered applications.
<img width="2028" height="1654" alt="image" src="https://github.com/user-attachments/assets/b0efeb55-4f86-4eb5-9bcc-54f57ce235ea" />

The core of FrugalGPT revolves around three key techniques for reducing LLM inference costs:

1) Prompt Adaptation - Using concise, optimized prompts to minimize prompt processing costs
2) LLM Approximation - Utilizing caches and model fine-tuning to avoid repeated queries to expensive models
3) LLM Cascade - Dynamically selecting the optimal set of LLMs to query based on the input

1.1 Decrease the prompt size: FrugalGPT proposes that instead of sending a lot of examples in a few-shot prompt, we could pick and choose the best ones, and thus reduce the prompt size.
1.2 Combine similar requests together: LLMs have been found to retain context for multiple tasks together and FrugalGPT proposes to use this to group multiple requests together thus decreasing the redundant prompt examples in each request.
Bonus 1.3: Better utilize a smaller model with a more optimized prompt: use Claude Opus to convert a zero-shot prompt into a few-shot prompt which could then be run on a much smaller model without a significant decrease in accuracy.
Bonus 1.4: Compress the prompt: reduce LLM costs called prompt compression. Prompt compression aims to shorten the input prompts fed into LLMs while preserving the key information. LLMLingua is a task-agnostic prompt compression method

2.1 Cache LLM Requests: The age-old technique to reduce costs applies to LLM requests as well. When the prompt is exactly the same, we can save the inference time and cost by serving the request from the cache.The generative AI twist to this is an emerging technique called Semantic Caching.
2.2 Fine-tune a smaller model in parallel. Fine-tuning is a very effective middle ground, where you can train a smaller model on a specific task and have it perform as well or even better than the larger model.
<img width="2012" height="1236" alt="image" src="https://github.com/user-attachments/assets/3b0d2c95-8c5b-4987-8d8a-3b7296d30b83" />

3)LLM Cascade: he key idea is to sequentially query different LLMs based on the confidence of the previous LLM's response. If a cheaper LLM can provide a satisfactory answer, there's no need to query the more expensive models, thus saving costs.
Generation Scoring Function: This function, denoted as g(q, a), assigns a reliability score between 0 and 1 to an LLM's response a for a given query q. It helps determine if the response is satisfactory or if we need to query the next LLM in the cascade.
LLM Router: The router is responsible for selecting the optimal sequence of m LLMs to query for a given task. It learns this sequence by optimizing for a combination of cost and performance on a validation set.


3) loopformer: https://github.com/armenjeddi/loopformer
Elastic-Depth Looped Transformers for Latent Reasoning via Shortcut Modulation





