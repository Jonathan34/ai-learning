---
title: "11 — Inference Economics"
nav_order: 1
parent: "Production"
---
# 11 — Inference Economics

> Status: **outline**. Will expand in full depth.

## The mental model

Every LLM request has three costs: input tokens, output tokens, and time. Your AI system's economics — whether it's viable, how you price it, what features you can afford to build — are determined by how you manage these three dimensions.

## Planned contents

- Input vs output token pricing (output is typically 3-5x input cost)
- How context length maps to cost (linearly-ish for current frontier models at inference)
- Prompt caching (Anthropic, OpenAI, Google) — the biggest single cost optimization when your prompts have a stable prefix
- Batching: when it helps, when it doesn't
- Streaming: UX win, also helps time-to-first-token
- Model routing: small model for easy cases, big model for hard cases
- Capacity and rate limits: how provider TPMs and RPMs translate to system limits
- Latency budget: where the time actually goes (network, queue, prefill, generation)
- Capacity planning: estimating TPM needed, cost envelope, failure modes when capacity runs out
- Multi-provider strategies (fallback, routing, cost arbitrage)

## Key gotchas

- Output tokens are the dominant cost for generative workloads. Prompt caching only helps the input.
- Many cost calculators underestimate by 2-10x because they miss the "model thinks, then model generates" asymmetry
- Provider capacity is not infinite — at scale, you'll hit rate limits and need to queue or spill over
- Tail latency kills UX — p99 matters more than p50 for interactive AI
- Token counts vary by tokenizer. Budget with the model's actual tokenizer, not an estimate.
- Agent loops multiply cost — a 10-step agent costs 10x a single-shot call, at minimum

## What a PE needs to be credible on

- Can estimate the cost of a proposed AI feature before building
- Can identify which levers actually move cost (usually output tokens, model size, cache hit rate)
- Can design capacity plans that survive 10x traffic spikes
- Can decide when a problem is "too expensive for AI" and say so
