---
layout: default

title: "07 — Framework Landscape"
nav_order: 2
parent: "Agents"
---
# 07 — Agent Frameworks Landscape

## The mental model

**Agent frameworks are middleware: they abstract common patterns (tool use, loops, memory, multi-agent coordination) so you don't have to build from scratch. They also introduce their own bugs, opinions, and lock-in.**

Think of them the way you think about web frameworks. Rails and Django let you ship faster on common patterns, at the cost of working inside someone else's abstractions. When your needs align with the framework, it's a win. When they don't, you spend more time fighting the framework than you'd have spent building from scratch.

The current agent framework landscape is in flux. No framework has won. Some are two years old and still shaping their APIs. Treat framework choice as a reversible decision: build with clean boundaries so you can swap or drop the framework when it stops serving you.

**The thing to remember:** frameworks don't solve the hard problems. They solve the easy ones. The hard problems — good prompts, good evals, good tool design, good observability, good context engineering — are yours regardless of what framework you pick.

## Why this matters in production

Three patterns of framework regret, all common:

- **"We picked LangChain early; now we can't untangle it."** The team built deep into LangChain's abstractions. Business logic and framework glue became intertwined. Upgrading LangChain breaks things. Replacing it means rewriting the whole system.
- **"We picked a framework for a feature we never used."** Team chose CrewAI because multi-agent looked cool. They're using it for single-agent tool calling. They've inherited the complexity for nothing.
- **"The framework is a black box when things fail."** Something goes wrong in production. The actual prompt sent to the model is 3 levels of framework abstraction away. Debugging requires reading framework source.

All three are avoidable with the right discipline: use frameworks deliberately, keep business logic separable from framework code, and always be able to see what the model is actually receiving.

## The major frameworks

As of late 2025, these are the ones worth knowing. I'll give each a short honest read. Inclusion doesn't mean endorsement — just that you'll encounter them in the wild.

### LangChain

The most widely-used framework. Broad scope (chat, RAG, agents, document loading, output parsing). Python-first with JavaScript/TypeScript support.

**Strengths:**
- Broad ecosystem — integrations with everything
- Large community, lots of documentation and examples
- Easy to prototype quickly
- Works well for CRUD-style LLM calls (chat, simple RAG)

**Weaknesses:**
- Abstractions are often leaky or leaky-by-design
- API has changed significantly multiple times; older tutorials are misleading
- The "it's just a thin wrapper" claim is marketing; there's real complexity in the abstractions
- Debugging what the model actually sees requires work
- Has a reputation in senior-engineer circles for being over-abstracted

**Verdict:** LangChain is fine for prototyping and simple production uses. For anything complex, you'll often fight the framework. Many teams migrate away as their systems mature.

### LangGraph

From the LangChain team, but a different philosophy. Explicitly graph-based: you define nodes (LLM calls, tool calls, conditionals) and edges (control flow). More principled than LangChain itself.

**Strengths:**
- Explicit state and control flow — you can see what the agent is doing
- Strong checkpointing and persistence support
- Better suited to complex agent/workflow designs than LangChain
- Streaming and human-in-the-loop patterns are first-class

**Weaknesses:**
- Still tied to the LangChain ecosystem and its abstraction style
- More ceremony than raw Python for simple cases
- Conceptually more to learn than "just call the model"

**Verdict:** LangGraph is a credible choice for complex agent systems. More senior-engineer-friendly than LangChain. If you're going to use something in the LangChain ecosystem, LangGraph is usually the right pick.

### Claude Agent SDK (Anthropic)

Anthropic's official toolkit for building agents on Claude. Tied to Claude models.

**Strengths:**
- Well-integrated with Claude's tool use capabilities, including Claude-specific features (prompt caching, extended thinking)
- Documentation reflects real production patterns
- Code examples that actually work
- Written by the people who understand Claude best
- TypeScript and Python SDKs available, both well-maintained

**Weaknesses:**
- Claude-only — not a multi-provider abstraction
- Newer, smaller ecosystem than LangChain
- Fewer prebuilt integrations (databases, tools, etc.)

**Verdict:** if you're building on Claude and want a framework that plays to Claude's strengths, this is the right starting point. The trade-off is single-provider lock-in, which is a real concern but often acceptable.

### OpenAI Agents SDK / Assistants API

OpenAI's agent-building toolkit. Includes the Assistants API (hosted) and the Agents SDK (client-side).

**Strengths:**
- First-party, well-maintained
- Assistants API handles state/threading for you
- Integrated with OpenAI's tool use and code interpreter features

**Weaknesses:**
- OpenAI-only
- Hosted state raises some data governance questions
- API has been in flux; breaking changes are common
- Less flexible than doing it yourself

**Verdict:** reasonable if you're OpenAI-native and want to move fast. Consider the lock-in.

### AutoGen (Microsoft)

Multi-agent framework from Microsoft Research. Actor-model-influenced: agents are participants that send messages to each other.

**Strengths:**
- Strong on multi-agent patterns
- Active research backing; novel patterns appear here first
- Supports human-as-agent seamlessly

**Weaknesses:**
- More opinionated than the others
- Learning curve
- Multi-agent is often unnecessary; AutoGen pulls you toward it

**Verdict:** if you have a genuine multi-agent use case, AutoGen is worth a look. For single-agent work, it's overkill.

### CrewAI

Role-based multi-agent coordination. Define "crew members" with roles, goals, and tools; they collaborate on tasks.

**Strengths:**
- Intuitive for role-play scenarios (e.g., "researcher + writer + editor")
- Quick to prototype impressive demos
- Python-first, simple API

**Weaknesses:**
- The role-play pattern is often more theatrical than useful
- Real production value beyond demos is debated
- Less mature than other options

**Verdict:** handle with skepticism. Role-playing agents look great in demos and often under-deliver in production. Use for specific cases where the roles genuinely differ in context or tools.

### Semantic Kernel (Microsoft)

Enterprise-flavored, .NET-first (with Python and Java support). Emphasizes "plugins" (tools) and "planners."

**Strengths:**
- Good fit for Microsoft/enterprise stacks
- .NET-native, unusual in this space
- Solid documentation

**Weaknesses:**
- Smaller community than LangChain ecosystem
- More enterprise-ceremony than needed for many projects

**Verdict:** right choice if your stack is Microsoft/Azure. Otherwise the ecosystem is smaller than alternatives.

### DSPy

Different philosophy: treats prompts as compiled programs. You write the pipeline structure; DSPy optimizes the prompts against a metric.

**Strengths:**
- Novel, research-grade approach
- Can produce better prompts than hand-tuning for well-defined tasks
- Forces you to think in terms of metrics

**Weaknesses:**
- Conceptually heavy; learning curve is real
- Less production-mature than other options
- Optimization runs cost tokens; retraining pipelines isn't free

**Verdict:** interesting for specific cases where you have clear metrics and want to optimize prompts programmatically. Not a general-purpose choice for most teams.

### The "newer entrants"

Mastra (TypeScript), Inngest AI, LlamaIndex (more RAG-focused, also does agents), Pydantic AI (Python, uses Pydantic types), and others keep appearing. Most will not survive. A few will. It's hard to predict which.

The common pattern: a new framework promises a cleaner abstraction, takes off for a year, then either matures into something solid or quietly loses momentum. Wait 6-12 months on new entrants unless one closely matches your needs.

### Just write Python

Often the right answer, especially for medium-complexity systems. A loop, a dictionary for state, direct calls to the provider's API, structured logging. Maybe a few utility functions. Total dependency footprint: the provider's SDK, maybe Pydantic, maybe a vector DB client.

**Strengths:**
- No framework to learn or fight
- Debugging is trivial — everything is in your code
- Zero lock-in
- You control exactly what the model sees
- Often less code than you'd write configuring a framework

**Weaknesses:**
- You reimplement some common patterns (retries, parsing, state persistence)
- Less out-of-the-box integration

**Verdict:** underrated choice. For teams with senior engineers, "write it yourself" is often the right call up to a certain complexity. You can always adopt a framework later. The reverse is much harder.

## Decision framework

When to use a framework:
- Your patterns align well with the framework's abstractions
- You need a specific feature the framework provides well (graph orchestration, state persistence, tracing integration)
- Your team will grow and consistency matters
- You're prototyping and want to move fast

When to skip and write it yourself:
- Your needs are simple enough that the framework adds complexity
- Your patterns don't match (and you'd fight abstractions)
- You need maximum debuggability
- You care about long-term lock-in risk

When to mix:
- Use a framework for one layer (e.g., tool orchestration) and raw code for another (e.g., prompt management)
- Use a lightweight helper (Pydantic for schemas, `instructor` for structured output) without committing to a full framework

**Senior move:** whatever you pick, keep your business logic (prompts, tool definitions, domain logic) separable from framework code. Use the framework as middleware, not as the application. If the framework changes or you need to swap it, the core of your system should survive the migration.

## What frameworks don't solve

All of these are yours regardless of what framework you pick:

- **Prompt quality and versioning.** No framework writes good prompts for you.
- **Evaluation.** No framework gives you test cases or quality rubrics.
- **Observability beyond traces.** Frameworks provide traces; understanding them is your job.
- **Cost management.** No framework forces you to respect a budget.
- **Security.** No framework sandbox's your tools for you.
- **Product sense.** No framework tells you which tools to build.

Frameworks solve syntax and glue. The hard parts are still yours.

## Gotchas

**Gotcha: Frameworks that hide the prompt.**

If you can't easily see the exact text sent to the model, debugging is impossible. Before committing to a framework, check that you can log or inspect the final prompt on every call. Any framework that makes this hard is a framework you'll regret.

**Gotcha: Breaking changes between minor versions.**

All these frameworks are young. APIs change. Pin your versions. Test upgrades in isolation before adopting.

**Gotcha: "Just add a memory node" — what's actually happening?**

Framework abstractions that look simple often mask a lot of activity. "Add memory" might mean the framework is now making 3 LLM calls per turn instead of 1. Always trace what's happening under the hood; budget accordingly.

**Gotcha: Framework-provided observability as a substitute for your own.**

LangSmith is fine, but tying all your tracing to one framework vendor is a lock-in risk. Keep at least some observability in your own infrastructure (logs, metrics, OpenTelemetry) that survives framework changes.

**Gotcha: Fighting the abstraction.**

The framework wants you to do X; you need Y. You end up writing adapters, inheriting framework classes, monkey-patching. This is a sign the framework isn't right for your use case. Step back and evaluate.

**Gotcha: "Framework X does MCP" and other feature claims.**

Check what "supports" actually means. Often it's a thin wrapper that works for the simplest case and breaks on the edges. Read the code.

**Gotcha: Abandoned frameworks.**

Some frameworks that were popular 18 months ago are now unmaintained. Check commit activity, issue responsiveness, and the state of the community before adopting. AutoGPT, for example, had a moment and then faded.

## Honest status

This space is still settling. LangChain is the de facto standard but not beloved. LangGraph is improving the story. Claude Agent SDK and OpenAI Agents SDK are strong if you're provider-committed. AutoGen and DSPy have research momentum. Nothing has decisively won.

My bet: in 2-3 years, a small number of frameworks will have matured into what Express/Flask were for web servers — well-known, reliable, understood by most engineers in the space. It's not obvious which ones. Today, hedge your bets: pick one, but build so you can switch.

## What to read next

- **08 — MCP and Tool Interfaces.** Tools are what agents actually do; MCP is how they'll increasingly connect.
- **Workshop W4 — Tool-Using Agent.** Build an agent from scratch, then port to a framework. Both exercises teach you something.
- **LangChain and LangGraph documentation** — read the real docs, not tutorials.
- **Anthropic's "Building Effective Agents"** — still the best practical guide, framework-agnostic.
- Each framework's **source code** — if you're going to adopt one seriously, read enough of the source to know what's really happening.
