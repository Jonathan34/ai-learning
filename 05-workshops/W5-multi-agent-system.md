---
title: "W5 — Multi-Agent System"
nav_order: 5
parent: "Workshops"
---
# Workshop W5 — Multi-Agent System

> Status: **outline**. Will expand in full depth.

**Goal:** build an orchestrator + specialists pattern and experience the failure modes firsthand.

**Time:** 3-4 hours

**Planned contents:**

1. Scenario: e.g., a "research assistant" with an orchestrator, a web-fetch agent, and a summarizer agent
2. Define coordination protocol (shared state vs. message-passing)
3. Build with LangGraph or AutoGen (pick one, make a choice)
4. Instrument heavily — you'll need it
5. Run 10 queries, watch what happens
6. Deliberately trigger failure cases (orchestrator loop, agent disagreement, stuck negotiation)
7. Add guardrails: budget, max steps, human checkpoint
8. Compare cost and latency vs. the W4 single-agent version

**Key gotchas demonstrated:**
- Multi-agent multiplies cost
- Coordination overhead
- Debugging across multiple LLM calls
- When the orchestrator is wrong, everything downstream is wrong
- Cases where a workflow or single agent would have been simpler
