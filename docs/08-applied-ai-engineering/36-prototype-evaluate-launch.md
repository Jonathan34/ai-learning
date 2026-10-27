# 36 — Prototype, Evaluate, Launch, Learn

An applied AI engineer should be able to move through the whole delivery loop.
The prototype, evaluation, production design, and customer conversation are
one piece of work.

## Build a vertical slice

A vertical slice follows one request through the real boundaries:

```text
input → model or workflow → data and tools → validated result
      → user action → trace and evaluation
```

It is intentionally narrow. It might handle only one support intent, one
document type, or one code language. Narrow scope lets you test real behavior
without pretending the whole product is ready.

Include early:

- a typed input and output contract;
- one real data or tool integration;
- timeouts and bounded retries;
- refusal and handoff behavior;
- trace IDs and useful logs;
- token, latency, and cost measurements;
- an evaluation command anyone can rerun.

Leave out until the main uncertainty is resolved:

- broad feature coverage;
- elaborate orchestration;
- perfect user interface;
- automatic actions with a large blast radius;
- optimization before a baseline exists.

## Map the task to platform building blocks

For an OpenAI-based implementation, the durable choices are:

| Need | Starting point |
|---|---|
| Model response, streaming, tools, or conversation state | [Responses API](https://developers.openai.com/api/docs/guides/migrate-to-responses) |
| Arguments or output consumed by code | [Structured Outputs](https://developers.openai.com/api/docs/guides/structured-outputs) |
| Access to application actions or live data | Function tools or [remote MCP](https://developers.openai.com/api/docs/guides/tools-connectors-mcp) |
| Private or changing documents | Retrieval or [file search](https://developers.openai.com/api/docs/guides/tools-file-search) |
| Agent loop, handoffs, guardrails, and tracing in the app | [Agents SDK](https://developers.openai.com/api/docs/guides/agents-sdk) |
| Low-latency voice interaction | [Realtime API](https://developers.openai.com/api/docs/guides/realtime) |
| Offline work that does not need an immediate answer | [Batch API](https://developers.openai.com/api/docs/guides/batch) |

Product names and limits change. The design questions do not:

- Where does state live?
- Which component may take an action?
- What is stored or sent to a provider?
- How is every result tied to a request and model version?
- How does the system stop, retry, resume, or hand off?

## Write tools as production interfaces

A tool is an API contract presented to a model. Give it:

- one clear job;
- a name that describes the action;
- a small typed schema;
- explicit units and allowed values;
- compact structured results;
- actionable error responses;
- an idempotency key: a request ID that makes a retried write return the
  original result instead of performing the action twice;
- separate permission for reading and changing data.

Bad:

```text
manage_order(data)
```

Better:

```text
get_order(order_id)
check_refund_eligibility(order_id, reason)
create_refund(order_id, amount_cents, idempotency_key)
```

The application still authenticates the user, checks permissions, validates
the amount, and records the action. The model does not become the security
boundary.

## Evaluate the system, not only the final sentence

For a tool-using workflow, grade:

| Layer | Example check |
|---|---|
| Input handling | The request is assigned to the right task |
| Retrieval | The required policy appears in the top results |
| Tool choice | The correct tool is called |
| Arguments | The order ID and amount are valid |
| Final answer | The explanation matches the tool result |
| Safety | No write happens without the required approval |
| Efficiency | The task stays within turn, token, and latency budgets |

Keep failure categories. A pass rate that moves from 84% to 86% says little if
the new failures issue incorrect refunds.

The [official OpenAI evaluation guidance](https://developers.openai.com/api/docs/guides/evaluation-best-practices)
recommends task-specific evals, logging, automation, and continuous evaluation.
Those are good platform-independent habits.

## Debug in layers

When the output is wrong, ask these in order:

1. Did the system receive the correct input and identity?
2. Did retrieval return the needed evidence?
3. Did the model select the correct tool?
4. Were the arguments valid?
5. Did the tool return correct, fresh data?
6. Did the model use the result correctly?
7. Did an output check or approval catch the failure?

"The model hallucinated" is often too vague to be useful.

## Treat latency as a path

```text
total latency =
  queue + retrieval + model + tools + retries + output delivery
```

Measure the median, called `p50`, and the slower end, often `p95`. A p95 of two
seconds means 95% of requests finished within two seconds.

Common improvements:

- stream output when partial results help;
- shorten irrelevant context;
- keep repeated prompt prefixes stable for caching;
- run independent tools in parallel;
- use a smaller model for simple routing or extraction;
- lower reasoning effort on easy tasks;
- move offline work to asynchronous processing;
- set deadlines so one slow dependency cannot hold the request forever.

Do not save 300 milliseconds in the model while ignoring a five-second tool.

## Measure cost per completed task

Include:

- every model call and retry;
- input, cached input, output, and reasoning tokens;
- retrieval and tool infrastructure;
- failed and abandoned runs;
- human review;
- the cost of errors.

A more expensive model can be cheaper per completed task if it needs fewer
turns or fewer human corrections. A cheaper model is not cheaper if its
failures create support work.

## Design the failure path before launch

| Failure | Expected behavior |
|---|---|
| Provider timeout | Retry within a deadline, then use a fallback or hand off |
| Rate limit | Back off with jitter; queue work that can wait |
| Tool timeout | Return a bounded error; do not guess |
| Invalid tool arguments | Reject before execution and ask for correction |
| Duplicate write | Use an idempotency key |
| Unsafe or high-value action | Require explicit approval |
| Missing evidence | Say the system cannot answer and route to a person |
| Model or prompt change | Run the eval and roll out gradually |

The [OpenAI latency guide](https://developers.openai.com/api/docs/guides/latency-optimization)
puts process design before small implementation tweaks: make fewer requests,
generate fewer tokens, parallelize independent work, and avoid using a model
where a rule or classical method works.

## Launch in steps

Use increasing authority:

1. Offline replay on past cases.
2. Shadow mode: run on live inputs but show nothing to users.
3. Draft mode: a person reviews every result.
4. Limited production: low-risk cases or a small traffic percentage.
5. Broader production with automatic action inside clear limits.

Define the rollback trigger before each step. A rollout is not controlled if
nobody knows which metric stops it.

## Adoption is part of engineering

A technically correct system can fail because it adds steps, hides its
evidence, or gives users no way to correct it.

Before handoff:

- train the team that will own it;
- give them the eval, runbook, dashboards, and known limits;
- measure usage and task completion;
- watch edits, overrides, escalations, and repeated questions;
- schedule a review after real use.

## Send useful feedback upstream

Product and research teams need more than "the model is bad."

Use this format:

```text
use case:
expected behavior:
smallest reproducible input:
actual behavior:
frequency:
user or business impact:
workaround:
evidence attached:
```

Aggregate repeated patterns. Ten examples across four customers are more
useful than one loud complaint.

## A 45-minute interview structure

```text
0-5 min    clarify the user, goal, scale, and failure cost
5-10       define the metric and rough numbers
10-20      draw the simplest end-to-end design
20-28      explain data, tools, state, and trust boundaries
28-34      define the evaluation
34-39      cover latency, cost, and failures
39-43      stage the rollout and ownership
43-45      summarize the recommendation and open risks
```

## Review questions

- Which part of the design does not need an LLM?
- What is the highest-cost error?
- Which assumption should the prototype test first?
- What would be in the first 50 evaluation cases?
- What data crosses a trust boundary?
- What stops a repeated tool call from performing the action twice?
- What metric pauses the rollout?

## Go deeper

- [Structured Output](../01-foundations/06-structured-output.md)
- [MCP and Tools](../02-agents/09-mcp-and-tools.md)
- [Observability](../03-production/13-observability.md)
- [Secure AI Platforms](../07-enterprise-architecture/33-secure-ai-platforms.md)
- [Official OpenAI production best practices](https://developers.openai.com/api/docs/guides/production-best-practices)
