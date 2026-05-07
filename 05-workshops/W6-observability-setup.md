---
layout: default

title: "W6 — Observability Setup"
nav_order: 6
parent: "Workshops"
---
# Workshop W6 — Observability Setup

> Status: **outline**. Will expand in full depth.

**Goal:** instrument an agent end-to-end so you can answer "what did the model see when it produced that output?" for any past call.

**Time:** 2-3 hours

**Planned contents:**

1. Take the agent from W4 or W5 and strip out any existing logging
2. Add structured logging: every prompt (system + user + assembled context), every response, every tool call, every result
3. Add OpenTelemetry traces with semantic conventions for LLMs
4. Set up Langfuse (free tier) or Braintrust (free tier) or both
5. Replay a failed session from logs — can you reconstruct what happened?
6. Add metrics: latency p50/p95/p99, tokens in/out, cost per request, tool call count
7. Set up a simple dashboard
8. Simulate a production incident (bad response, timeout, tool failure) — can you debug from traces alone?

**Key gotchas demonstrated:**
- Context truncation issues only visible from full logs
- The difference between logging the "prompt template" and logging the "actual prompt the model saw"
- Sampling trade-offs
- Cost of full context logging at scale
