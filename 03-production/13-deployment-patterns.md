---
layout: default

title: "13 — Deployment Patterns"
nav_order: 3
parent: "Production"
---
# 13 — Deployment Patterns

> Status: **outline**. Will expand in full depth.

## The mental model

How you deploy an AI system determines its latency, cost, compliance posture, and reliability. The decision tree is richer than most teams think. "We'll just use OpenAI" is a deployment decision with real consequences.

## Planned contents

### The main options
- **Hosted API**: OpenAI, Anthropic, Google (Gemini). Fastest to start, least control, data leaves your perimeter.
- **Hosted through cloud provider**: Amazon Bedrock, Google Vertex AI, Azure OpenAI. Middle ground; easier compliance, sometimes cheaper at scale.
- **Self-hosted open-weights model**: Llama, Mistral, Qwen, Nemotron, etc., on your infrastructure. Most control, most operational burden.
- **Hybrid**: different models for different use cases, by policy or economics

### Decision factors
- Data sensitivity (regulated, PII, IP)
- Latency requirements (network hops matter)
- Scale (hosted can be cheaper at low volume, self-hosted at high volume)
- Compliance (data residency, certifications)
- Model availability (newest frontier models are only hosted)
- Offline / air-gapped requirements

### Operational patterns
- Multi-region for latency and availability
- Fallback across providers when one has an outage
- Request queueing when capacity is constrained
- Caching (prompt cache, response cache, semantic cache)
- Circuit breakers and graceful degradation

### Model lifecycle
- Pinning versions in production
- Canary and staged rollouts for model upgrades
- Regression testing before cutover
- Rollback plans

## Key gotchas

- "Just use OpenAI" is often fine, until a data governance or outage conversation starts
- Self-hosting open-weights models is harder than it looks: ops, capacity, versioning, GPU procurement
- Fallback across providers sounds great; in practice, behavior differs enough that fallback is quality risk
- Bedrock/Vertex/Azure are not identical to calling the underlying model directly (different rate limits, slightly different features)
- Regional availability matters — not every model is in every region

## What a PE needs to be credible on

- Can design the deployment architecture for an AI system given business constraints
- Can make the "hosted vs self-hosted" decision and defend it
- Can design failure modes and fallbacks
- Knows the compliance landscape (at least the categories — GDPR, HIPAA, SOC 2 context)
