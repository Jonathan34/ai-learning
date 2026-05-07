---
title: "09 — Multi-Agent Patterns"
nav_order: 4
parent: "Agents"
---
# 09 — Multi-Agent Patterns (When Useful, When Snake Oil)

> Status: **outline**. Will expand in full depth.

## The mental model

Multi-agent systems have multiple LLM-driven components coordinating to accomplish a task. They're often more impressive in demos than in production. The research literature shows real gains on some problems, but a lot of "multi-agent" in the wild is just complexity without benefit.

A useful heuristic: **multi-agent is worth it when different subtasks genuinely need different contexts, different tools, or different prompts. It's not worth it to make architecture diagrams look more sophisticated.**

## Planned contents

### Patterns that work
- **Router + specialists** — classifier agent directs to specialist agents with narrower tools/prompts
- **Orchestrator + workers** — a planning agent decomposes tasks and delegates
- **Critic / reviewer** — one agent produces, another critiques, they iterate
- **Committee / ensemble** — multiple agents answer, aggregation picks the best
- **Human-in-the-loop as an agent** — the human is a first-class participant

### Anti-patterns
- Role-playing theater ("you are a product manager, you are an engineer, you are a designer") — looks good, rarely produces better output
- Deep agent hierarchies — each level adds latency, cost, and failure modes
- "Let's let them negotiate" — agents don't converge, they loop
- Multi-agent as a proxy for not knowing how to specify the task

### When to use what
- Decision framework for workflow vs. agent vs. multi-agent
- Cost analysis (multi-agent multiplies costs; sometimes that's fine)
- Observability in multi-agent systems (harder by an order of magnitude)

## Key gotchas

- Multi-agent systems fail in ways single agents don't: deadlocks, infinite loops, conflicting goals
- Cost can be 5-10x a single-agent approach; make sure the benefit is commensurate
- Debugging is hard; you need strong tracing from the start
- "Autonomous" multi-agent systems make non-reversible actions easier to reach — audit trails become critical
- Communication overhead: every inter-agent message is tokens, which means cost and context budget

## What a PE needs to be credible on

- Can identify when a problem genuinely needs multi-agent and when single-agent is simpler
- Can design coordination protocols (shared state, message queues, control flow)
- Can predict the failure modes before building
- Can argue against a multi-agent design when a simpler one works
