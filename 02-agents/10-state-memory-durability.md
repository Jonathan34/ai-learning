---
title: "10 — State, Memory, Durability"
nav_order: 5
parent: "Agents"
---
# 10 — State, Memory, and Long-Running Agents

> Status: **outline**. Will expand in full depth.

## The mental model

An LLM has no memory. Any "memory" in an agent system is a state system you build around the model. For long-running agents (hours, days, persistent assistants), state management becomes the central engineering problem — bigger than the model itself.

## Planned contents

### Kinds of state
- Conversation history (short-term, usually in-memory per session)
- Working memory (current task, plan, scratchpad)
- Long-term memory (facts, preferences, prior interactions)
- Shared state across agents (when multi-agent)
- Tool state (what tools have been called, their results)
- World model (agent's belief about external state)

### Persistence patterns
- Session store (Redis, in-memory, database)
- Event log (every action the agent took, replayable)
- Checkpointing (save the full agent state at key points, resume later)
- Vector store for semantic memory retrieval

### The durability problem
- Agents that run for hours/days need crash recovery
- Idempotent tool calls so retries don't double-charge, double-send, etc.
- State migrations when your schema changes
- Garbage collection: stale facts, expired sessions

### Memory quality
- Memory consolidation (pruning, summarization, de-duplication)
- Handling contradictions in memory
- Privacy and retention (what can you keep, for how long, user's right to delete)
- Testing memory systems (hard — like testing distributed systems)

### Frameworks
- LangGraph checkpointing
- Mem0, Zep for managed memory
- DIY with Postgres / Redis / S3
- Claude Agent SDK memory patterns

## Key gotchas

- Storing the full context of every interaction → unbounded storage cost
- Storing too little → agent "forgets" important context
- PII in memory and the retention policies that implies
- Memory poisoning: user plants a false fact in early session, agent trusts it forever
- "Chat history" as memory → stops being useful above a few turns; needs summarization
- Non-deterministic memory → same query retrieves different facts on different runs
- Cold-start cost: rehydrating long memory at session start takes time

## What a PE needs to be credible on

- Can design a memory architecture for a given agent (what's stored, where, for how long)
- Can reason about cost, latency, and privacy implications of memory choices
- Can design crash recovery for long-running agents
- Can design memory pruning and conflict resolution
- Knows when "memory" is actually solving the user's problem and when it's cargo-culted
