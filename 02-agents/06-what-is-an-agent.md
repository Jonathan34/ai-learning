---
layout: default

title: "06 — What an Agent Actually Is"
nav_order: 1
parent: "Agents"
---
# 06 — What an Agent Actually Is

## The mental model

**"Agent" is the most overloaded term in AI today. Useful definitions exist on a spectrum, not as a single category.**

At one end: an LLM that retrieves documents and answers questions. People call this an agent. At the other end: an autonomous system that plans its own goals, acts in the world, and runs unsupervised for hours. People call that an agent too.

The spectrum that matters, roughly from simplest to most complex:

1. **Augmented generation** — LLM + retrieval. No tool use, no decisions. Not really an agent. Often called one.
2. **Tool use** — LLM can call functions to get information or act. The model decides what tools to call and with what arguments. Minimal agent.
3. **ReAct / planned tool use** — model alternates reasoning and action in a loop. Decides next step based on the result of the previous one.
4. **Goal-directed agent** — model decomposes a higher-level goal into steps, executes them, revises the plan if things change.
5. **Autonomous agent** — model operates in a loop with memory, tools, and initiative over long time horizons. Still rare in production.
6. **Multi-agent systems** — multiple LLMs in different roles coordinating via messages or shared state.

The most useful operational definition, borrowed from Anthropic's taxonomy: an agent is a system where **an LLM dynamically controls its own process and tool usage**. If the steps are predetermined and the LLM just fills in slots, it's a workflow, not an agent.

**The thing to remember:** knowing where your system sits on this spectrum is the single most important architectural decision. A lot of teams accidentally build toward the right-hand side when something simpler would work — and pay in latency, cost, and reliability.

## Why this matters

The "agent" label gets applied to marketing because it sounds ambitious. Engineering-wise, agents are more expensive, more complex, harder to evaluate, and more failure-prone than workflows. Every step up the spectrum adds cost and reduces predictability.

Real production AI teams mostly ship workflows with LLMs at nodes. The things sold as "autonomous agents" are usually tool-using LLMs inside a `for` loop with a stop condition. There's nothing wrong with that — but the framing matters for how you design, evaluate, and explain the system.

**Senior move:** when a team proposes an "agent" for a new use case, ask "what's the simplest thing that would work?" and work up from there. Most of the time the answer is "a workflow with an LLM doing one or two steps," and the feature ships faster with fewer production surprises.

## Workflow vs. agent: the distinction that matters

**A workflow is a system where the control flow is predetermined.** You decide, at design time, what the steps are and in what order. An LLM may be called at each step to transform inputs into outputs, but it doesn't decide what happens next.

Example workflow — customer support triage:
1. Classify the incoming ticket (LLM)
2. If urgent, route to on-call (deterministic)
3. If refund request, check order status (deterministic) then draft reply (LLM)
4. Otherwise, attempt auto-response (LLM), else route to human

The LLM is called at steps 1, 3, and 4. The graph of what happens when is fixed. This is cheap, reliable, easy to evaluate, and easy to debug.

**An agent is a system where the LLM decides the control flow.** It chooses what to do next based on observation and reasoning.

Example agent — customer support:
1. Given the ticket, the LLM plans what to do
2. It calls tools (lookup order, check refund policy, search knowledge base)
3. Based on results, it decides to either respond, ask for more info, or escalate
4. It may go through several iterations before producing a final output

Same task, different architecture. The agent is more flexible and handles novel situations the workflow wasn't designed for. It's also harder to predict, harder to evaluate, harder to debug, and more expensive.

**Both are valid.** The question is which is right for your problem.

Use a workflow when:
- The task decomposes cleanly into known steps
- The same steps work for most inputs
- You need predictable cost and latency
- You need reliable behavior

Use an agent when:
- The right sequence of steps depends on the input
- You can't anticipate all the cases at design time
- The flexibility is worth the complexity

**Pro tip:** many real systems are hybrids. The top-level flow is a workflow; one node of the workflow is an agent. That's often the right structure — the agent handles the open-ended part, the workflow handles the deterministic parts.

## The agent loop, concretely

Every agent has the same basic loop:

```
while not done:
    observation = read_current_state()
    decision = model(context + observation + tools)
    if decision.is_final_answer:
        return decision.output
    else:
        tool_result = execute(decision.tool_call)
        context = append(context, decision, tool_result)
    if exceeded_budget():
        return graceful_failure()
```

That's it. Everything else — frameworks, patterns, orchestrators — is variations on this.

The parameters that actually matter:

- **Budget**: max loop iterations, max tokens consumed, max wall-clock time
- **Stop conditions**: when does the agent decide it's done? When do you force it to stop?
- **Tool set**: what can the agent call? What can't it?
- **Context window management**: how do you keep the context from growing unboundedly?
- **Memory**: what persists across loop iterations? Across sessions?
- **Error handling**: what happens when a tool call fails? When the model produces malformed output?

A lot of agent engineering is getting these parameters right for your specific task.

## Agent decision-making patterns

**ReAct (Reason + Act)** is the baseline. The model outputs alternating "Thought" and "Action" tokens, reasoning about what to do next and then doing it. Most production agents are ReAct-flavored whether they call it that or not.

**Plan-and-Execute** separates planning from execution. The model first generates a full plan, then executes step by step. More predictable cost and latency; less flexible (harder to recover if a step fails in a way the plan didn't anticipate). Good for tasks with predictable structure.

**Tree-of-Thought** explores multiple reasoning paths before committing. The model generates several possible next steps, evaluates them, picks the best. Expensive. Usually overkill.

**Reflexion / self-correction** has the agent review its own work and retry. "Here's my answer. Let me check if it's right. Actually, it's wrong because X. Let me try again." Adds cost but can meaningfully improve quality on tasks where verification is easier than generation (like code, where you can run tests).

**What actually works in production**: basic ReAct with solid tool descriptions, clear stop conditions, good error handling, and eval-driven iteration. Fancy patterns rarely pay for themselves on routine tasks.

## When not to use an agent

- **When a workflow is simpler.** Most tasks.
- **When predictable cost matters.** Agent loops have highly variable token consumption.
- **When low latency matters.** Each loop iteration is a round trip to the model.
- **When reliability is critical.** Agents fail in more ways than workflows.
- **When you can't afford to debug exotic failure modes.** Agent debugging is harder by an order of magnitude.
- **When the task has a clear structure.** If you can write the steps, you probably shouldn't ask the model to decide them.

**Gotcha:** teams often default to "let's build an agent" because it sounds more impressive. Senior engineering judgment pushes back: the simpler architecture that solves the problem is usually the right one, and "agent" is often over-specification.

## The design primitives

When you do build an agent, these are the primitives you're composing:

- **Goal** — what the agent is trying to accomplish (usually given in the user's message plus system prompt)
- **Plan** — the agent's current intended sequence of steps (explicit in plan-and-execute, implicit in ReAct)
- **Tools** — the functions the agent can call
- **Observation** — what the agent has learned from tool calls so far
- **Memory** — state that persists (conversation, prior actions, user context)
- **Context budget** — how much the agent can afford to do (tokens, steps, wall clock, dollars)
- **Stop condition** — when the agent decides to stop, or is forced to stop

Every design choice is about how to structure these. A good agent design document names each explicitly.

## Production realities

**Most "agents" are loops around tool-using LLMs.** That's not a dismissive statement — it's accurate. The loop, the tools, the stop conditions are the agent. The "autonomy" and "reasoning" framing is marketing on top.

**Agent reliability in production is often below 80%.** Not "the model is wrong 20% of the time" — "the full agent trajectory gets to the right outcome 80% of the time." For many use cases, 80% isn't good enough, and the path to 95% is engineering (eval, iteration, human-in-the-loop), not a better model.

**Cost scales with loop depth.** A 10-step agent is at least 10x the cost of a single call. Longer contexts on each step make it worse. Budget your agent use cases carefully.

**Latency scales with loop depth too.** A 10-step agent is ~10x the latency of a single call, minimum. For interactive use, this matters — users hate waiting.

**Agents need eval harnesses even more than single calls do.** Because the trajectory can go many ways, small changes in prompt or tool schema can introduce subtle regressions. Without eval, you're flying blind.

## Gotchas

**Gotcha: Loops that don't terminate.**

The model keeps calling tools, getting results, calling more tools, never deciding it's done. Always have a max step count and a max token budget. Without them, a runaway agent can blow your bill in hours.

**Gotcha: Tool hallucinations.**

The model calls a tool you didn't give it, or passes arguments that don't fit the schema. Validate every tool call before executing. Return a graceful error to the model ("that tool doesn't exist" or "missing required argument") so it can recover.

**Gotcha: Plan drift.**

The model generates a plan at step 0, starts executing, learns something that invalidates the plan, but keeps following the original plan anyway. Good agents re-plan in response to observations; poor ones charge ahead.

**Gotcha: Over-trusting autonomy.**

"The agent decided to do X" is not a satisfying explanation when X is expensive or wrong. Agents need observability, budgets, and human checkpoints proportional to the stakes of what they can do.

**Gotcha: Escalating complexity.**

A team starts with a simple agent, it works OK, they add memory, then multi-agent, then self-reflection, and now the system is brittle and hard to debug. Complexity has costs. Add it only when the simpler version demonstrably fails.

**Gotcha: The "agent" label confuses stakeholders.**

Product managers, executives, and customers have wildly different mental models of what "agent" means. Be specific: "This is a tool-using assistant with three tools and a max budget of 5 steps" is clearer than "an autonomous agent."

**Gotcha: Cost surprises from edge cases.**

The agent works fine on the test set but occasionally encounters an input that makes it loop hard — 50 tool calls, a 100K-token context, and a $5 query cost. Rare but it happens. Monitor for it.

## Honest status

Agent architecture is the most active area of applied AI research and engineering as of late 2025. New patterns, frameworks, and benchmarks emerge monthly. What's settled:
- Workflows with LLMs at nodes are a strong, cheap, reliable pattern for many tasks
- Simple tool-using agents work well for tasks that need flexibility
- Multi-agent systems are still experimental; they work for some narrow use cases

What's not settled:
- How to evaluate agents at scale
- How to guarantee agents stay safe across long horizons
- Whether "autonomous" agents are a meaningful production category yet

Expect the field to change substantially over the next 2-3 years. The mental model of "where on the spectrum" will remain useful even as the specific techniques evolve.

## What to read next

- **Anthropic's "Building Effective Agents"** (blog post, 2024) — the single most useful contemporary piece on agent architecture. Read this before reading anything else about agents.
- **07 — Agent Frameworks Landscape.** Now that you know what an agent is, you can reason about frameworks that help you build one.
- **Workshop W4 — Tool-Using Agent.** Build one from scratch without a framework. This is the workshop that makes the concepts concrete.
- **"ReAct: Synergizing Reasoning and Acting in Language Models"** (Yao et al., 2022) — the paper that kicked off the current wave.
- **HuggingFace's agents course** — practical, free, code-heavy.
