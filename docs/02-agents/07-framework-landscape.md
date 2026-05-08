# Agent Frameworks Landscape

Agent frameworks are middleware. They abstract common patterns — tool use, loops, memory, multi-agent coordination — so you don't build from scratch. They also introduce their own bugs, opinions, and lock-in.

Think of them the way you think about web frameworks. Rails and Django let you ship faster on common patterns, at the cost of working inside someone else's abstractions. When your needs align, it's a win. When they don't, you spend more time fighting the framework than you'd have spent building from scratch.

The current landscape is in flux. No framework has won. Some are two years old and still reshaping their APIs. Treat framework choice as a reversible decision: build with clean boundaries so you can swap or drop the framework when it stops serving you.

Frameworks don't solve the hard problems. They solve the easy ones. The hard problems — good prompts, good evals, good tool design, good observability, good context engineering — are yours regardless of what framework you pick.

## Framework regret

Three patterns I've seen repeatedly:

**"We picked LangChain early; now we can't untangle it."** Business logic and framework glue became intertwined. Upgrading LangChain breaks things. Replacing it means rewriting the whole system.

**"We picked a framework for a feature we never used."** Team chose CrewAI because multi-agent looked cool. They're using it for single-agent tool calling. They inherited the complexity for nothing.

**"The framework is a black box when things fail."** Something goes wrong in production. The actual prompt sent to the model is 3 levels of framework abstraction away. Debugging requires reading framework source.

All three are avoidable: use frameworks deliberately, keep business logic separable from framework code, and always be able to see what the model is actually receiving.

## The major frameworks

As of late 2025, these are the ones worth knowing. Inclusion doesn't mean endorsement — just that you'll encounter them.

### LangChain

The most widely-used framework. Broad scope: chat, RAG, agents, document loading, output parsing. Python-first with JavaScript/TypeScript support.

| | |
|---|---|
| Strengths | Broad ecosystem, large community, lots of examples, easy to prototype |
| Weaknesses | Leaky abstractions, API has changed significantly multiple times, debugging what the model sees requires work, reputation for over-abstraction |

Fine for prototyping and simple production uses. For anything complex, you'll often fight the framework. Many teams migrate away as their systems mature.

### LangGraph

From the LangChain team, but a different philosophy. Explicitly graph-based: you define nodes (LLM calls, tool calls, conditionals) and edges (control flow).

| | |
|---|---|
| Strengths | Explicit state and control flow, strong checkpointing, streaming and human-in-the-loop are first-class |
| Weaknesses | Still tied to the LangChain ecosystem, more ceremony than raw Python for simple cases |

A credible choice for complex agent systems. More senior-engineer-friendly than LangChain. If you're going to use something in the LangChain ecosystem, LangGraph is usually the right pick.

### Claude Agent SDK (Anthropic)

Anthropic's official toolkit for building agents on Claude. Tied to Claude models.

| | |
|---|---|
| Strengths | Well-integrated with Claude's tool use, documentation reflects real production patterns, code examples that work |
| Weaknesses | Claude-only, newer and smaller ecosystem, fewer prebuilt integrations |

If you're building on Claude and want a framework that plays to its strengths, this is the right starting point. The trade-off is single-provider lock-in.

### OpenAI Agents SDK / Assistants API

OpenAI's agent-building toolkit. Includes the Assistants API (hosted) and the Agents SDK (client-side).

| | |
|---|---|
| Strengths | First-party, well-maintained, Assistants API handles state/threading for you |
| Weaknesses | OpenAI-only, hosted state raises data governance questions, API has been in flux |

Reasonable if you're OpenAI-native and want to move fast. Consider the lock-in.

### AutoGen (Microsoft)

Multi-agent framework from Microsoft Research. Actor-model-influenced: agents are participants that send messages to each other.

| | |
|---|---|
| Strengths | Strong on multi-agent patterns, active research backing, supports human-as-agent |
| Weaknesses | More opinionated, steeper learning curve, pulls you toward multi-agent even when unnecessary |

If you have a genuine multi-agent use case, worth a look. For single-agent work, it's overkill.

### CrewAI

Role-based multi-agent coordination. Define "crew members" with roles, goals, and tools; they collaborate on tasks.

| | |
|---|---|
| Strengths | Intuitive for role-play scenarios, quick to prototype demos |
| Weaknesses | Role-play pattern is often more theatrical than useful, less mature |

Handle with skepticism. Role-playing agents look great in demos and often under-deliver in production.

### Semantic Kernel (Microsoft)

Enterprise-flavored, .NET-first (with Python and Java support). Emphasizes "plugins" (tools) and "planners."

| | |
|---|---|
| Strengths | Good fit for Microsoft/Azure stacks, .NET-native (unusual in this space) |
| Weaknesses | Smaller community, more enterprise ceremony than needed for many projects |

Right choice if your stack is Microsoft/Azure. Otherwise the ecosystem is smaller than alternatives.

### DSPy

Different philosophy entirely: treats prompts as compiled programs. You write the pipeline structure; DSPy optimizes the prompts against a metric (a numerical score you define for output quality).

| | |
|---|---|
| Strengths | Novel approach, can produce better prompts than hand-tuning for well-defined tasks, forces metric-driven thinking |
| Weaknesses | Conceptually heavy, less production-mature, optimization runs cost tokens |

Interesting for specific cases where you have clear metrics and want to optimize prompts programmatically. Not a general-purpose choice.

### The newer entrants

Mastra (TypeScript), Inngest AI, LlamaIndex (more RAG-focused, also does agents), Pydantic AI (Python, uses Pydantic types), and others keep appearing. Most will not survive. A few will.

The common pattern: a new framework promises a cleaner abstraction, takes off for a year, then either matures into something solid or quietly loses momentum. Wait 6-12 months on new entrants unless one closely matches your needs.

### Just write Python

Often the right answer, especially for medium-complexity systems. A loop, a dictionary for state, direct calls to the provider's API, structured logging. Maybe a few utility functions. Total dependency footprint: the provider's SDK, maybe Pydantic, maybe a vector DB client.

No framework to learn or fight. Debugging is trivial — everything is in your code. Zero lock-in. You control exactly what the model sees. Often less code than you'd write configuring a framework.

The downside: you reimplement some common patterns (retries, parsing, state persistence) and get less out-of-the-box integration.

For teams with senior engineers, "write it yourself" is often the right call up to a certain complexity. You can always adopt a framework later. The reverse is much harder.

## Deciding

When to use a framework:
- Your patterns align well with the framework's abstractions
- You need a specific feature the framework provides well (graph orchestration, state persistence, tracing)
- Your team will grow and consistency matters
- You're prototyping and want to move fast

When to skip and write it yourself:
- Your needs are simple enough that the framework adds complexity
- Your patterns don't match and you'd fight abstractions
- You need maximum debuggability
- You care about long-term lock-in risk

When to mix:
- Use a framework for one layer (tool orchestration) and raw code for another (prompt management)
- Use a lightweight helper (Pydantic for schemas, `instructor` for structured output) without committing to a full framework

Whatever you pick, keep your business logic — prompts, tool definitions, domain logic — separable from framework code. Use the framework as middleware, not as the application. If the framework changes or you need to swap it, the core of your system should survive the migration.

## What frameworks don't solve

All of these are yours regardless of what framework you pick:

- **Prompt quality and versioning.** No framework writes good prompts for you.
- **Evaluation.** No framework gives you test cases or quality rubrics.
- **Observability beyond traces.** Frameworks provide traces; understanding them is your job.
- **Cost management.** No framework forces you to respect a budget.
- **Security.** No framework sandboxes your tools for you.
- **Product sense.** No framework tells you which tools to build.

Frameworks solve syntax and glue. The hard parts are still yours.

## Gotchas

**Frameworks that hide the prompt.** If you can't easily see the exact text sent to the model, debugging is impossible. Before committing to a framework, check that you can log or inspect the final prompt on every call.

**Breaking changes between minor versions.** All these frameworks are young. APIs change. Pin your versions. Test upgrades in isolation.

**"Just add a memory node."** Framework abstractions that look simple often mask a lot of activity. "Add memory" might mean the framework is now making 3 LLM calls per turn instead of 1. Trace what's happening under the hood.

**Framework-provided observability as a substitute for your own.** LangSmith is fine, but tying all your tracing to one framework vendor is a lock-in risk. Keep at least some observability in your own infrastructure that survives framework changes.

**"Framework X does MCP" and other feature claims.** Check what "supports" actually means. Often it's a thin wrapper that works for the simplest case and breaks on the edges. Read the code.

**Abandoned frameworks.** Some frameworks that were popular 18 months ago are now unmaintained. Check commit activity, issue responsiveness, and community health before adopting.

## Where things stand

This space is still settling. LangChain is the de facto standard but not beloved. LangGraph is improving the story. Claude Agent SDK and OpenAI Agents SDK are strong if you're provider-committed. AutoGen and DSPy have research momentum. Nothing has decisively won.

My bet: in 2-3 years, a small number of frameworks will have matured into what Express/Flask were for web servers — well-known, reliable, understood by most engineers in the space. It's not obvious which ones. Today, hedge your bets: pick one, but build so you can switch.

## Go deeper

- [Anthropic's "Building Effective Agents"](https://www.anthropic.com/research/building-effective-agents) — still the best practical guide, framework-agnostic
- [LangGraph documentation](https://langchain-ai.github.io/langgraph/) — read the real docs, not tutorials
- Workshop W4 — Tool-Using Agent. Build an agent from scratch, then port to a framework.
- Each framework's source code — if you're going to adopt one seriously, read enough of the source to know what's really happening
