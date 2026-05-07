---
title: "12 — Observability for AI"
nav_order: 2
parent: "Production"
---
# 12 — Observability for AI Systems

> Status: **outline**. Will expand in full depth.

## The mental model

In traditional distributed systems, observability is logs, metrics, traces. In AI systems, you need all three plus the ability to reconstruct *exactly what the model saw and produced*. Without that, debugging is guesswork.

## Planned contents

### What to instrument
- Every prompt: system prompt, user input, full assembled context, model, parameters
- Every response: raw output, parsed output, token counts, finish reason
- Every tool call: name, arguments, result, duration, errors
- Agent loops: each step, with full state snapshot
- User feedback: thumbs up/down, corrections, escalations

### Tracing AI applications
- Distributed tracing adapted for LLM calls
- OpenTelemetry for AI (LLM semantic conventions)
- Span structure for agent loops and tool calls
- Correlating user sessions with model calls

### Metrics that matter
- Latency: time-to-first-token, time-to-last-token, total
- Quality: eval scores in production (running eval continuously on a sample)
- Cost: tokens in, tokens out, $ per request
- Error rates: API errors, parse errors, refusals, tool failures
- User signals: feedback rates, escalations, session length

### Tools
- Braintrust, Langfuse, LangSmith, Helicone, Arize — the specialized vendors
- OpenTelemetry + your existing observability stack
- DIY with structured logs

### Online evaluation
- Sampling production calls for human review
- LLM-as-judge on a sampled subset, tracked over time
- Drift detection: is the model behaving differently this week?

## Key gotchas

- Logging prompts means logging user data — respect privacy, redact PII, encrypt at rest
- Full-context logging is expensive at scale; sample intelligently
- Without correlation IDs, debugging a multi-step agent is impossible
- Prompt caching makes latency metrics misleading if you don't account for cache state
- "The model did X" is not reproducible without the full context and model parameters
- Dashboards without alert thresholds are decorative

## What a PE needs to be credible on

- Can design an observability plan for an AI system from scratch
- Can answer "what did the model see when it produced this output?" — always
- Can detect quality regressions in production, not just in CI
- Can balance instrumentation cost with debugging value
