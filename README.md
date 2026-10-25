# AI Systems — Learning Path for Senior Engineers

## Who this is for

You're a senior software engineer, architect, or engineering leader. You know distributed systems, cloud, operational excellence, probably multiple languages, and you've shipped production software for years. You don't need to learn what a function is.

What you *do* need is to become credible across the AI stack — from model-facing software to the GPU infrastructure under it. Not "I read an article about transformers" credible, but "I can build the system, size the platform, and defend the trade-offs in a design review" credible.

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
05-ai-workloads/             (ML foundations, workloads, sizing, and MLOps — 5 chapters)
06-ai-infrastructure/        (GPUs, networks, storage, orchestration, IaC — 6 chapters)
07-enterprise-architecture/  (Solution architecture and production adoption — 3 chapters)
05-workshops/                (Hands-on practice — 17 workshops, multi-hour each)
reference/                   (Glossary, papers, tools, cloud platforms, gotchas, videos)
```

## Part I: Software engineering for AI

This part builds the judgment needed to ship AI software. It starts with model behavior, then moves through agents, production systems, and technical leadership.

### Phase 1: Foundations (~60 min reading)

The conceptual layer. You won't build production systems without these.

- **01. How LLMs Actually Work** — the mental model an engineer needs
- **02. Prompting as Programming** — prompts as contracts, not incantations
- **03. Context Engineering** — the broader discipline that includes RAG, memory, tools
- **04. Evaluation: The Hardest Unsolved Problem** — why ship decisions are hard
- **05. Security and Safety** — two very different things, both critical
- **06. Structured Output and Type Safety** — bridging LLM text to typed code

### Phase 2: Agents (~50 min reading)

The specific systems that will define the next five years of infrastructure.

- **07. What an Agent Actually Is** — the spectrum from tool use to autonomy
- **08. Agent Frameworks Landscape** — LangChain, LangGraph, AutoGen, Claude Agent SDK, CrewAI
- **09. MCP and Tool Interfaces** — the interoperability layer
- **10. Multi-Agent Patterns** — when useful, when snake oil
- **11. State, Memory, and Long-Running Agents** — the durability problem

### Phase 3: Production (~50 min reading)

Running AI systems at scale, with cost and latency and reliability.

- **12. Inference Economics** — tokens, latency, capacity, caching
- **13. Observability for AI** — tracing, metrics, eval-in-production
- **14. Deployment Patterns** — hosted vs self-hosted, managed vs custom
- **15. Local and Edge Inference** — Ollama, llama.cpp, vLLM, TensorRT
- **16. AI Platform Engineering** — the LLMOps layer that doesn't exist yet

### Phase 4: Leadership (~40 min reading)

The architect and Principal Engineer scope.

- **17. Deciding What to Build with AI** — when it's worth it, when it isn't
- **18. AI Product Sense** — what good AI UX looks like
- **19. Team and Org Patterns** — shape of an AI team, career ladders, hiring
- **20. Staying Current** — how to not drown in the firehose

## Part II: Cloud infrastructure for AI

This part follows the bytes below the API. It covers how training and inference use hardware, how GPU clusters are built, and how to move a platform from a benchmark to a reliable production service.

### Phase 5: AI workloads

- **21. ML Algorithms for Architects** — problem families, model choices, metrics, and infrastructure implications
- **22. Distributed Training Systems** — parallelism, memory, communication, and failure
- **23. Self-Hosted Inference** — prefill, decode, KV cache, batching, and serving engines
- **24. Workload Sizing and Economics** — turn an SLO or training target into GPUs, time, and cost
- **25. The MLOps Lifecycle** — data, experiments, pipelines, registries, deployment, and monitoring

### Phase 6: AI infrastructure

- **26. GPU Systems** — memory, bandwidth, interconnects, software stack, and health
- **27. Cluster Networking** — RDMA, InfiniBand, RoCE, NCCL, and topology
- **28. Storage and the Data Path** — datasets, checkpoints, model loading, and caching
- **29. Kubernetes and Slurm** — GPU scheduling, gang scheduling, and platform choice
- **30. Infrastructure as Code** — repeatable environments that can grow past a demo
- **31. Reliability and Performance** — acceptance tests, observability, and structured debugging

### Phase 7: Solution architecture

- **32. AI Solution Architecture in Practice** — knowledge areas, use cases, deliverables, and hands-on depth
- **33. Secure AI Platforms** — identity, isolation, data boundaries, governance, and operations
- **34. From Proof of Concept to Production** — discovery, success criteria, TCO, and handoff

## Workshops (hands-on)

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
- **W11. Python for AI Infrastructure** — parse benchmarks and automate health checks
- **W12. GPU Cluster Acceptance Test** — prove compute, network, and storage before a workload
- **W13. Workload Sizing and TCO** — build a small capacity and cost model
- **W14. Enterprise AI Platform Capstone** — design and defend a production platform
- **W15. End-to-End MLOps** — train, track, validate, deploy, monitor, promote, and roll back a model
- **W16. Kubernetes Operations for AI** — operate networking, storage, scheduling, tenancy, failures, and upgrades
- **W17. Automate an AI Platform** — provision a safe cloud platform with Terraform and validate it with Python

### Reference

- **Glossary** — terms a PE needs in their vocabulary
- **Papers to Know** — the ~20 papers that actually matter
- **Tools and Models** — quick reference, caveat: changes fast
- **Cloud ML Platform Landscape** — compare NVIDIA, AWS, Azure, and Google Cloud by architecture layer
- **Gotchas** — consolidated list across all chapters
- **AI Infrastructure Videos** — a guided watch list paired with the new chapters

## Suggested path

Three ways to approach this depending on your time:

**AI software sprint (2 weekends, ~20 hours).** Read Phase 1 and 2. Do W1, W2, and W4. You'll be conversant in applied AI.

**AI infrastructure bridge (4-6 weeks, part-time).** Read Phase 5 through 7. Do W11 through W17. You'll be able to reason from model choice and lifecycle down to GPU, network, storage, and platform choices.

**Full stack (8-12 weeks, part-time).** Complete both parts and the core workshops. You'll be credible in software, platform, and architecture discussions.

## Prerequisites

- Solid software engineering experience and comfort with Python
- Basic familiarity with LLM APIs (you've called OpenAI or Anthropic at least once)
- Understanding of distributed systems, basic cloud services, HTTP
- Shell and git

No GPU is required for most of the reading or sizing exercises. The cluster workshop can use borrowed, rented, or work-provided hardware.

## What this is not

- **Not an ML research course.** It covers the workload mechanics an engineer needs, not how to invent a new model architecture.
- **Not a coding bootcamp.** I'll give you code when it clarifies a concept.
- **Not a research summary.** I'll point to papers when they matter; I won't re-summarize them.
- **Not exhaustive.** This is the *working knowledge* a Principal Engineer needs. You'll keep learning after.

## How "graduation" works

You've internalized this material when you can:

1. Sit in a design review for a new AI feature and ask the right questions (not "what if the model hallucinates" — that's table stakes — but "what's your eval strategy, who owns the regression harness, how do you handle prompt drift across model versions").
2. Read a new LLM paper from DeepMind or Anthropic and tell a colleague in 5 minutes what matters.
3. Architect an agentic system end-to-end: identify the agent boundaries, tool interfaces, state/memory model, failure modes, cost model, evaluation approach.
4. Explain where an AI workload is limited by memory, compute, network, storage, or scheduling.
5. Size a training or inference platform with explicit assumptions, then describe how you would test those assumptions.
6. Design a secure GPU platform and a path from proof of concept to production.
7. Walk a stakeholder through "should we use AI here" without defaulting to either hype or dismissal.
8. Mentor an engineer who's new to this space.

You don't have to do all of this today. You have to be able to see the path.

---

Start with **`00-how-to-use-this.md`** for the meta, then go to **`01-foundations/01-llm-mental-model.md`**.
