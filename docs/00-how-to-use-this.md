# How to Use This Material

## Format

Most chapters are written to be read in 8-12 minutes. The infrastructure chapters are a little denser. Workshops are longer — plan for a half day or a weekend.

Chapters are not all structured the same way. Some start with a concept, some with a problem, some jump straight into the details. The shape of each chapter matches what its content needs.

Most chapters include:

- The mental model you need

- Why it matters in production

- What experienced people know that doesn't show up in tutorials

- Common traps and failure modes

- Links to deeper reading

Workshops are hands-on. Every workshop has you build something real and compare it to what you expected.

## Two parts, one stack

Part I is about software engineering for AI: model behavior, agents, evaluation, production systems, and technical leadership.

Part II starts where an API abstraction stops. It covers distributed training, self-hosted inference, GPU systems, cluster networks, storage, scheduling, IaC, and enterprise platform design.

If you already build production AI applications, start at Phase 5. Go back to Part I when a topic exposes a gap.

## What "credible" means here

The goal is to be credible in architecture reviews with people who've been doing this longer than you.

That looks like:

- You can design a system that works and explain *why* it will work, not just that it will

- You can predict the failure modes before you ship

- You know what you don't know, and you say so without performing humility

- Buzzwords don't impress or intimidate you — you translate them into concrete components

- You can work with ML engineers as peers

You're not here to become an ML researcher. You're here to become someone ML engineers want in the room when architecture gets decided.

## Pacing

**Sprint path (2 full days):** Read Phase 1 and 2. Do W1, W2, W4. You'll be conversant.

**Depth path (4-6 weeks, part-time):** All chapters, all workshops. You'll be ready for any architecture review.

**Principled path (6-12 months):** Add the papers. Contribute to an open-source project. Ship something production-grade at work.

**Infrastructure bridge (4-6 weeks, part-time):** Phases 5-7 plus W11-W17. Use a real workload and real measurements whenever you can.

## One piece of advice

The hardest part of learning this is not the concepts. It's resisting the temptation to skim.

A lot of AI content is fluff. Your filter from other domains is good — "is this person actually doing the work, or just talking about it?" Apply that filter here too. I've tried to keep fluff out. When a chapter seems obvious, trust that the 20% you're missing is the part that matters in production.

## When you get stuck

Three moves that work:

1. **Sleep on it.** Concepts often click overnight.

2. **Build the concrete version.** Jump to the relevant workshop.

3. **Read real production code.** Anthropic's docs, LangChain's source, the Claude Agent SDK — reading real prompts and eval harnesses is one of the fastest ways to build taste.

Don't get stuck perfecting one chapter. Keep moving. The chapters reinforce each other.
