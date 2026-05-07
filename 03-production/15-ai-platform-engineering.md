---
title: "15 — AI Platform Engineering"
nav_order: 5
parent: "Production"
---
# 15 — AI Platform Engineering (LLMOps)

> Status: **outline**. Will expand in full depth.

## The mental model

If your organization has more than a handful of AI features, you need a platform. Otherwise every team reinvents prompt versioning, eval harnesses, observability, rate limiting, secret management, and cost attribution. LLMOps — the set of platform services supporting AI in production — is where DevOps was in 2015: the patterns are emerging, the tooling is splintered, and there's a huge gap between leaders and laggards.

## Planned contents

### The platform stack
- Prompt registry (versioning, metadata, environment-aware)
- Model registry and routing
- Eval pipelines (offline and online)
- Observability (traces, metrics, logs)
- Cost attribution (per team, per feature, per user)
- Rate limiting and quota management
- Secret and credential management (API keys for providers)
- Capability governance (who can call which model, with which data)
- Red-teaming and safety review workflows

### Developer experience
- Prompt playground for engineers
- SDKs / wrappers around provider APIs that bake in observability, retries, fallbacks
- Templates for common patterns (chat, RAG, agent)
- Local dev with production parity

### Cross-cutting concerns
- Data governance and PII handling
- Audit logs for compliance
- Incident response for AI failures
- Model upgrade and deprecation playbooks

### Buy-vs-build decisions
- Which platform pieces buy (Braintrust, Langfuse, Helicone, OpenRouter)
- Which to build (often: the integration layer that ties them together)
- Which to defer (most orgs don't need everything immediately)

## Key gotchas

- Starting with the fanciest platform → team can't use it effectively
- No platform at all → every team reinvents and makes different trade-offs
- Building the platform before you know what you need → platforms that solve problems no one has
- Ignoring cost attribution → surprise six-figure bills
- "We'll just let engineers pick a provider" → compliance and cost chaos

## What a PE needs to be credible on

- Can design an LLMOps platform roadmap for an organization
- Can decide what to build vs. buy
- Can identify where a platform investment will and won't pay off
- Can communicate the ROI of platform work to leadership in an era where "AI velocity" is the stated goal
