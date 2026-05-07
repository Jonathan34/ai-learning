---
title: "16 — Deciding What to Build"
nav_order: 1
parent: "Leadership"
---
# 16 — Deciding What to Build with AI

> Status: **outline**. Will expand in full depth.

## The mental model

Most organizations are currently making two opposite mistakes: shipping AI features where AI isn't the right tool, and dismissing AI where it is. The Principal Engineer's job is to help the organization make better decisions about *where* AI creates real value.

## Planned contents

### The viability filter
- Is there a genuine task being automated or augmented?
- Is the task one LLMs are actually good at (language tasks, summarization, classification, extraction, generation), or bad at (arithmetic, reasoning with strict constraints, up-to-the-minute facts without retrieval)?
- What's the cost of being wrong, and how often will the system be wrong?
- Is the user better off with AI or with a simpler tool?

### The cost-benefit model
- Engineering cost (building, evaluating, maintaining)
- Inference cost (ongoing)
- Quality cost (errors reaching users)
- Opportunity cost (what you're not building)
- vs. Value (revenue, efficiency, new capability)

### The "good fit" archetypes
- Summarization of user-generated content
- Classification and triage where patterns are loose
- Draft-generation that humans review
- Conversational interface to deterministic backend
- Extraction from unstructured text
- Translation and adaptation
- Pattern matching where examples are available but rules are hard to write

### The "bad fit" archetypes
- Arithmetic and precise calculation
- Systems with legal/regulatory compliance on exact output
- Low-latency, high-volume lookup
- When a simple rule or traditional ML model would work
- Workflows where the cost of one error is catastrophic

### Helping the org say no
- "Can we add AI to X?" — the right first response is questions, not a yes or no
- Pushing back on AI theater
- Recommending non-AI solutions when they fit better
- The political reality that "AI feature" sells better than "rule engine," even when the latter is right

## Key gotchas

- The demo effect: a great demo doesn't mean a shippable product
- The reverse: dismissing a capability because the first prototype fails
- Over-indexing on cost savings vs. capability gains (or vice versa)
- Not accounting for maintenance: AI features rot as models change
- "Let's just try it" → scope creep into "we have to support it forever"

## What a PE needs to be credible on

- Can evaluate an AI feature proposal and give a grounded yes/no/maybe
- Can articulate why a non-AI solution is better when it is, without being dismissive
- Can push back on executives who want "AI" because it's in the budget slide
- Can advocate for AI where it's genuinely the right tool, including for work that doesn't look flashy
