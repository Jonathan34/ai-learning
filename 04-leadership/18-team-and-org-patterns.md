---
layout: default

title: "18 — Team and Org Patterns"
nav_order: 3
parent: "Leadership"
---
# 18 — Team and Org Patterns

> Status: **outline**. Will expand in full depth.

## The mental model

How you organize AI work determines how well it scales across the business. The most common pattern (every product team rolls their own) creates chaos. The other common pattern (a central AI team owns everything) creates bottlenecks. Good organizations evolve toward a platform-plus-embedded model.

## Planned contents

### The anti-patterns
- "AI center of excellence" that becomes a bottleneck
- Every product team building their own prompts and evals from scratch
- Research-flavored ML teams building papers-not-products
- "We'll figure out evals later" cultures

### Patterns that work
- Central platform team builds LLMOps; product teams build features on top
- Embedded "AI champions" in product teams, with a central community of practice
- Shared eval harnesses, prompt registry, model routing
- Dedicated safety/red-team function that reviews new features
- Regular AI architecture reviews (like security reviews, but for AI concerns)

### Roles
- AI engineer / Applied AI engineer / Forward-deployed engineer
- AI platform engineer
- ML engineer (traditional + modern)
- Prompt engineer / evals engineer
- AI product manager
- ML research engineer (if you do any real research)

### Hiring
- What to screen for (specific LLM project experience, taste, fast-learning)
- What to avoid (hype chasers, benchmark gamers)
- The ML-engineer-vs-software-engineer confusion
- Generalist engineers are often better than specialists for early AI teams

### Career ladders
- How do you promote an AI engineer vs a software engineer?
- The "deep ML" vs "applied AI" specialization
- Principal-level expectations in an AI-heavy role

### Community of practice
- Internal AI/LLM Slack channel that's actually active
- Monthly demo day for AI features
- Paper reading groups (if you have people who want this)

## Key gotchas

- Over-hiring "AI specialists" when applied software engineers with interest would be fine
- Under-investing in platform work because "just build features"
- Letting research agendas disconnect from product needs
- Creating parallel orgs that duplicate infrastructure
- Ignoring the change management — AI shifts engineering culture, not just tools

## What a PE needs to be credible on

- Can design an org structure for AI work at their company's size
- Can advocate for platform investment that pays off at 2-3x team scale
- Can hire AI engineers; can coach existing engineers into AI work
- Can write the career ladder expectations that work
- Can moderate between ML research types and applied engineers
