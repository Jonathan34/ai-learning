# AI & Agentic Systems — Learning Path for Senior Engineers

## Who this is for

You're a senior software engineer, architect, or engineering leader. You know distributed systems, cloud, operational excellence, probably multiple languages, and you've shipped production software for years. You don't need to learn what a function is.

What you *do* need is to become credible in the AI/agentic systems space — not "I read an article about transformers" credible, but "I can architect a production agent system, mentor an AI team, and stand up in a design review with ML engineers" credible.

This curriculum is designed for that gap.

## Philosophy

- **Assume competence.** No "what is a variable" energy. If you've built distributed systems, you can skip our training-wheels equivalent.
- **Mental models over trivia.** Knowing the 2023 MMLU leaderboard is useless. Knowing *why* evaluation is the hardest unsolved problem is the difference between a Principal Engineer and an enthusiast.
- **Grounded in production.** Every section connects to what breaks in real systems, not demos.
- **Gotchas included.** What experienced AI engineers have learned the hard way, in writing.
- **Hands-on.** You can't BS-detect a field you haven't built in. The workshops exist for that reason.
- **Honest about the field.** Some of this is solid; some is still a mess. I'll say which is which.

## Curriculum map

```
01-foundations/              (Core mental models — 6 chapters, ~60 min)
02-agents/                   (Agentic systems in depth — 5 chapters, ~50 min)
03-production/               (Running AI in real systems — 5 chapters, ~50 min)
04-leadership/               (Architect and PE-level concerns — 4 chapters, ~40 min)
05-workshops/                (Hands-on practice — 10 workshops, multi-hour each)
reference/                   (Glossary, papers, tools, gotchas)
```

### Phase 1: Foundations (~50 min reading)

The conceptual layer. You won't build production systems without these.

- **01. How LLMs Actually Work** — the mental model an engineer needs
- **02. Prompting as Programming** — prompts as contracts, not incantations
- **03. Context Engineering** — the broader discipline that includes RAG, memory, tools
- **04. Evaluation: The Hardest Unsolved Problem** — why ship decisions are hard
- **05. Security and Safety** — two very different things, both critical
- **06. Structured Output and Type Safety** — bridging LLM text to typed code

### Phase 2: Agents (~50 min reading)

The specific systems that will define the next five years of infrastructure.

- **06. What an Agent Actually Is** — the spectrum from tool use to autonomy
- **07. Agent Frameworks Landscape** — LangChain, LangGraph, AutoGen, Claude Agent SDK, CrewAI
- **08. MCP and Tool Interfaces** — the interoperability layer
- **09. Multi-Agent Patterns** — when useful, when snake oil
- **10. State, Memory, and Long-Running Agents** — the durability problem

### Phase 3: Production (~50 min reading)

Running AI systems at scale, with cost and latency and reliability.

- **11. Inference Economics** — tokens, latency, capacity, caching
- **12. Observability for AI** — tracing, metrics, eval-in-production
- **13. Deployment Patterns** — hosted vs self-hosted, managed vs custom
- **14. Local and Edge Inference** — Ollama, llama.cpp, vLLM, TensorRT
- **15. AI Platform Engineering** — the LLMOps layer that doesn't exist yet

### Phase 4: Leadership (~40 min reading)

The architect and Principal Engineer scope.

- **16. Deciding What to Build with AI** — when it's worth it, when it isn't
- **17. AI Product Sense** — what good AI UX looks like
- **18. Team and Org Patterns** — shape of an AI team, career ladders, hiring
- **19. Staying Current** — how to not drown in the firehose

### Phase 5: Workshops (hands-on)

You can't reason about a field you haven't built in.

- **W1. Local LLM Setup** — Ollama, quantization, comparing models
- **W2. First RAG Pipeline** — chunking, embeddings, retrieval, re-ranking
- **W3. Real Evaluation** — test set + LLM-as-judge + regression
- **W4. Tool-Using Agent** — from scratch, with proper guardrails
- **W5. Multi-Agent System** — orchestrator + specialists
- **W6. Observability Setup** — tracing an agent end-to-end
- **W7. Red-Team Exercise** — break your own system
- **W8. Cost Optimization** — measure, route, cache, budget
- **W9. Agent Efficiency KPIs** — instrument and benchmark agent performance
- **W10. Structured Output in Practice** — extraction, validation, retry, eval

### Reference

- **Glossary** — terms a PE needs in their vocabulary
- **Papers to Know** — the ~20 papers that actually matter
- **Tools and Models** — quick reference, caveat: changes fast
- **Gotchas** — consolidated list across all chapters

## Suggested path

Three ways to approach this depending on your time:

**The sprint (2 weekends, ~20 hours).** Read all of Phase 1 and 2. Do W1, W2, W4. You'll be conversant.

**The depth (4-6 weeks, part-time).** All chapters. All workshops. You'll be credible in any architecture review or PE interview.

**The principled (6-12 months).** Add the papers. Contribute to an open-source agent framework. Ship something production-grade at work. You'll be a real voice in the space.

## Prerequisites

- Comfort with Python (not expertise — just "can read and write it")
- Basic familiarity with LLM APIs (you've called OpenAI or Anthropic at least once)
- Understanding of distributed systems, basic cloud services, HTTP
- Shell and git

If you've built LLM pipelines on Bedrock with retrieval and evals (the work you're already doing), you're already past the starting line.

## What this is not

- **Not an ML course.** I'll barely mention gradient descent. You don't need it to architect systems.
- **Not a coding bootcamp.** I'll give you code when it clarifies a concept.
- **Not a research summary.** I'll point to papers when they matter; I won't re-summarize them.
- **Not exhaustive.** This is the *working knowledge* a Principal Engineer needs. You'll keep learning after.

## How "graduation" works

You've internalized this material when you can:

1. Sit in a design review for a new AI feature and ask the right questions (not "what if the model hallucinates" — that's table stakes — but "what's your eval strategy, who owns the regression harness, how do you handle prompt drift across model versions").
2. Read a new LLM paper from DeepMind or Anthropic and tell a colleague in 5 minutes what matters.
3. Architect an agentic system end-to-end: identify the agent boundaries, tool interfaces, state/memory model, failure modes, cost model, evaluation approach.
4. Walk a customer or stakeholder through "should we use AI here" without defaulting to either hype or dismissal.
5. Mentor an engineer who's new to this space.

You don't have to do all of this today. You have to be able to see the path.

---

Start with **`00-how-to-use-this.md`** for the meta, then go to **`01-foundations/01-llm-mental-model.md`**.
