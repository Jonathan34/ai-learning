# Workshop W18 — Applied AI Customer Pilot

**Goal:** Build and measure one production-shaped OpenAI API workflow that you
can explain in a project deep dive.

**Time:** 6-10 hours

**Prerequisites:** Chapters 35-38 and W3

Use public or synthetic data only. Keep API keys in environment variables and
never commit them.

---

## Choose a narrow problem

Pick one:

- classify public software-release notes by user impact;
- create a structured compatibility report from public documentation;
- answer questions over a small set of public policies with citations;
- review a public pull request against a short engineering rubric.

Write:

```text
user:
current workflow:
baseline:
target:
highest-cost error:
first supported cases:
cases explicitly excluded:
```

If the target cannot be measured, narrow the problem.

## Part 1: Create the evaluation first

Build 20-50 hand-checked cases. Include:

- normal requests;
- ambiguous requests;
- missing information;
- conflicting sources;
- inputs that should produce "I cannot determine this";
- content that tries to override system instructions;
- very long and malformed inputs.

Use deterministic checks for the output schema, IDs, citations, and required
fields. Write a small rubric for qualities such as correctness and clarity.

Record the current baseline. It may be a manual workflow, a simple prompt, or
a rule-based method.

## Part 2: Build one vertical slice

Use the official OpenAI SDK and Responses API.

The slice should include:

- one typed request;
- Structured Outputs for the result;
- one function tool with mocked or public data;
- explicit refusal and missing-data handling;
- a request ID;
- timeouts and bounded retries;
- structured logs for latency, token use, model version, and errors.

Keep the application layer small. The point is to understand and defend the
control flow.

```text
input
  → retrieve or call one tool
  → model produces a typed proposal
  → code validates business rules
  → result or explicit handoff
  → trace and evaluation record
```

## Part 3: Run the evaluation

For every case, store:

- pass or fail by grader;
- model and prompt version;
- latency;
- input, cached-input, and output tokens where available;
- estimated cost;
- tool calls;
- refusal or error.

Inspect every failure. Group them by cause:

- missing evidence;
- retrieval failure;
- wrong tool or arguments;
- model interpretation;
- validator gap;
- timeout or rate limit;
- grader problem.

Fix one category at a time and rerun the whole set.

## Part 4: Compare two or three designs

Examples:

- one strong model versus a smaller model;
- low versus higher reasoning effort;
- full document in the prompt versus retrieval;
- one call versus a classify-then-answer workflow;
- stable repeated prefix versus a changing prefix;
- sequential versus parallel read tools.

Use the same cases for every comparison.

| Design | Quality | p50 latency | p95 latency | Cost per correct result | Main failure |
|---|---:|---:|---:|---:|---|
| A | | | | | |
| B | | | | | |
| C | | | | | |

Recommend one design and say what evidence would change the recommendation.

## Part 5: Inject failures

Test:

- model timeout;
- 429 rate limit;
- tool timeout;
- malformed tool result;
- duplicate write request;
- missing source;
- output that passes the schema but violates a business rule;
- a model or prompt change that lowers quality.

The application should fail clearly and stay inside a total deadline. It must
not invent a result when evidence is missing.

## Part 6: Write the production plan

Cover:

- identity and data boundaries;
- rate-limit and peak-traffic estimate;
- retry and idempotency behavior;
- online quality signals;
- privacy-safe traces;
- rollout stages and rollback trigger;
- ownership after handoff;
- the first three production risks.

Do not claim the workshop prototype is production-ready. Explain what remains.

## Deliverables

Keep these in one public-safe repository:

1. One-page problem and architecture note.
2. Small working application.
3. Evaluation dataset and runner.
4. Results table across configurations.
5. Failure tests.
6. Production-readiness checklist.
7. Five-minute demo script.

## Five-minute deep-dive script

```text
0:00  user problem and baseline
0:40  why this scope and architecture
1:30  evaluation design
2:20  one failure and what changed
3:10  quality, latency, and cost results
4:00  production gaps and rollout
4:40  recommendation
```

Use real measurements. "The newer model was better" is weak. "It improved
policy accuracy from X to Y, added Z milliseconds at p95, and cost N more per
correct task" supports a decision.

## Review questions

- Which component does not need a model?
- What is the release-blocking failure?
- What does the schema guarantee, and what does it not guarantee?
- How does a retry avoid repeating a write?
- Which result surprised you?
- What would you tell a customer to build next?

## Go deeper

- [Applied AI Engineering](../08-applied-ai-engineering/index.md)
- [Official OpenAI Responses API guidance](https://developers.openai.com/api/docs/guides/migrate-to-responses)
- [Official OpenAI Structured Outputs guidance](https://developers.openai.com/api/docs/guides/structured-outputs)
- [Official OpenAI evaluation best practices](https://developers.openai.com/api/docs/guides/evaluation-best-practices)
