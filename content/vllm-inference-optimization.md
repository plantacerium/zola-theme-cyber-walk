+++
title = "Optimizing Inference: The Rise of vLLM"
date = 2023-11-02
description = "Python is too slow for token generation. How PagedAttention and vLLM are saturating GPU memory bandwidth."
[taxonomies]
categories = ["SYS/ADMIN"]
tags = ["inference", "cuda", "optimization", "gpu"]
+++

In the world of Large Language Models, **throughput** is money. The bottleneck in serving LLMs is rarely compute; it is memory bandwidth.

## The Memory Fragmentation Problem

Standard attention mechanisms require contiguous memory blocks for Key-Value (KV) caches. As sequences grow, memory fragmentation occurs, leading to wasted VRAM (Video RAM). If you have 20% wasted VRAM, that's 20% fewer requests you can batch concurrently.

## Enter PagedAttention (vLLM)

Inspired by OS virtual memory pagination, vLLM allows the KV cache to be split into non-contiguous blocks. This allows for near-zero memory waste.

### Implementation Benchmark

Deploying a Llama-3-8B model using standard HuggingFace pipelines vs vLLM shows drastic differences.

```python
# Standard HuggingFace Pipeline (Slow)
from transformers import pipeline
pipe = pipeline("text-generation", model="meta-llama/Llama-3-8b")
# ~ 30 tokens/sec on A100

# vLLM Engine (Fast)
from vllm import LLM, SamplingParams
llm = LLM(model="meta-llama/Llama-3-8b")
# ~ 85 tokens/sec on A100 (due to better batching)
Continuous Batching
Traditional batching waits for all requests in a batch to finish before sending the response. If one request asks for 500 tokens and another for 5 tokens, the short request is blocked by the long one.

vLLM implements Continuous Batching (iteration-level scheduling). As soon as a sequence finishes, a new one is inserted into the batch immediately.

The Infrastructure Stack
For a production-grade Python AI backend in 2025, the standard stack is becoming:

FastAPI for the web layer.

vLLM or TGI (Text Generation Inference) for the engine.

Prometheus for monitoring token/sec latency.

Stop serving models with raw torch.