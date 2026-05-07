---
layout: default

title: "17 — AI Product Sense"
nav_order: 2
parent: "Leadership"
---
# 17 — AI Product Sense

> Status: **outline**. Will expand in full depth.

## The mental model

A production AI feature is not a model. It's a model wrapped in UX decisions that determine whether users trust it, use it, and benefit from it. Good AI product sense is a specific skill — adjacent to general product sense but with its own traps.

## Planned contents

### What "good" AI UX looks like
- Loading states that match actual latency (streaming beats spinners)
- Uncertainty made visible, not hidden
- Confidence calibration in the UI (when to say "I'm not sure")
- Citations and verification paths for claims
- Graceful degradation when the model fails or refuses
- Edit-ability of AI output (the human should be able to correct and continue)

### Latency budget
- First token in <1s where possible; longer is OK for async
- Streaming creates the perception of speed
- Perceived performance matters more than actual for most tasks

### Trust engineering
- Users trust systems that are calibrated (admit uncertainty, cite sources)
- Users distrust systems that are over-confident
- Consistency builds trust; randomness erodes it
- Disclosure: when to show the user this is AI-generated

### Interaction patterns
- Single-turn (query, response)
- Multi-turn conversation
- Document-grounded chat
- Agent with a visible plan
- AI-assisted editing (the Copilot pattern)
- Invisible AI (the model runs behind a deterministic UI)

### Failure modes and their UX
- Hallucination: citations, "I don't know," verification paths
- Refusal: make it clear why, offer alternatives
- Over-promising: don't let the model commit to things it can't deliver
- Stale memory: let the user correct

## Key gotchas

- Shipping a chatbot as the UX by default → often the wrong choice
- Not designing for the failure case → fragile in production
- Over-indexing on the happy path demo → rude surprise at launch
- Disclosure done poorly → either spammy (every response is "I am an AI") or missing (user doesn't know)
- Inconsistent behavior across sessions → users don't know what to expect

## What a PE needs to be credible on

- Can critique an AI feature design from a UX perspective, not just an engineering one
- Can suggest interaction patterns beyond "chatbot"
- Can advocate for investment in AI-specific UX (streaming, citations, uncertainty) over generic PM polish
- Can work fluently with designers and PMs on AI features
