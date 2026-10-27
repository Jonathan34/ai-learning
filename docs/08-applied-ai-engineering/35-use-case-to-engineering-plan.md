# 35 — From Use Case to Engineering Plan

The first request is rarely the real problem.

"Build us an agent" might mean support costs are rising. "Add semantic search"
might mean users cannot find the right product. "Use the newest model" might
mean the current feature has stopped improving.

Do not start with the model. Start with the decision or workflow that needs to
change.

## The working loop

```text
outcome → current baseline → smallest useful change → prototype
        → evaluation → production rollout → learn from real use
```

Each arrow should produce evidence. If a prototype looks impressive but does
not answer a decision, it is a demo.

## Write the problem in one paragraph

Use this shape:

> **For** a named user, **the current workflow** causes a measurable problem.
> We want to improve a named outcome from a baseline to a target, while
> respecting specific constraints. We will test the idea on a defined set of
> cases before production.

Example:

> For support agents, reading policy pages and order history adds four minutes
> to an average refund request. We want to cut that time to two minutes without
> increasing incorrect refunds. The first test will cover 200 past cases and
> will only draft a recommendation; a person still approves the action.

That statement already tells you more than "build a support agent":

- The user is the support agent, not the end customer.
- The first outcome is handling time.
- Incorrect refunds are the main risk.
- Historical cases can form an evaluation set.
- The first version has no authority to move money.

## Discovery that changes the design

Ask questions whose answers remove or select architecture options.

### Outcome

- What user or business result should change?
- What is the current baseline?
- Who owns the metric?
- What result would make the work worth continuing?

### Workflow

- What happens today, step by step?
- Where do people wait, repeat work, or make errors?
- Which step needs judgment and which is a lookup or rule?
- What systems hold the required data?

### Failure

- What is the cost of a wrong answer?
- Can a person review the result before an action?
- Which actions are reversible?
- When should the system refuse or hand off?

### Scale and experience

- Requests per day and peak requests per second?
- Typical and worst-case input size?
- Interactive, near-real-time, or offline?
- Which latency matters: first response, full response, or job completion?

### Delivery

- Who will integrate and operate the result?
- Which security, data-retention, and regional rules apply?
- What date or product decision drives the schedule?
- What has already been tried?

End discovery with a written problem, assumptions, open questions, success
criteria, owners, and the next test.

## Pick use cases by evidence, not excitement

Score a candidate from `1` to `5` in each area:

| Area | A high score means |
|---|---|
| Value | Solving it changes revenue, cost, risk, or user experience |
| Measurability | The team can define and collect correct examples |
| Feasibility | The needed data and system access exist |
| Failure tolerance | Early mistakes are reviewable or reversible |
| Time to proof | A useful test can run in days or weeks |
| Adoption | A named team will own the changed workflow |

A high-value problem with no way to measure correctness is not ready. A
low-risk task with no user or owner will become an unused demo.

Good early use cases often:

- produce a draft that a person can review;
- classify, extract, route, summarize, or search;
- have many past examples;
- save time in a workflow that already exists;
- fail in ways the team can detect.

Hard first use cases often:

- take irreversible actions with weak supervision;
- depend on private data that is not accessible;
- have no stable definition of correct;
- need near-perfect accuracy across rare cases;
- replace a workflow nobody has mapped.

## Choose the least complicated design that works

Use an architecture ladder:

| Level | Pattern | Use it when |
|---|---|---|
| 0 | Rules or search | The answer can be stated directly |
| 1 | One model call | The task is one transformation |
| 2 | Model plus retrieval | The answer needs private or changing knowledge |
| 3 | Model plus tools | The system must read live state or take an action |
| 4 | Fixed multi-step workflow | The steps are known in advance |
| 5 | Agent loop | The model must decide which steps to take |

Start low. Move up only when the evaluation shows the simpler design cannot
meet the target.

An agent is not a synonym for an AI application. If every refund request must
run `load order → load policy → check eligibility → draft response`, write
that workflow in code. Let the model handle the parts that need language or
judgment.

## Change the right part of the system

When quality is weak, choose the change that matches the failure:

| Problem | First change to try |
|---|---|
| Instructions are unclear | Rewrite the prompt and add examples |
| Private or changing facts are missing | Retrieve the facts at request time |
| Live state or an action is needed | Add a narrow, typed tool |
| The steps are known | Build a fixed workflow in code |
| The steps depend on the case | Add a bounded agent loop |
| A repeated behavior stays weak at scale | Consider fine-tuning after the eval and data exist |

Fine-tuning changes learned behavior. It is usually a poor way to keep product
facts current. Retrieval is easier to update and can show the source.

## Define the evaluation before the prototype

Write down:

- the input cases;
- the expected answer or expected properties;
- the grader for each property;
- the minimum acceptable score;
- the latency and cost limit;
- the failures that always block release.

Use deterministic checks for facts a program can verify:

- output matches a schema;
- the selected order belongs to the user;
- a refund amount is within the allowed limit;
- every citation points to retrieved text;
- a required tool was called once.

Use a model grader for qualities that need judgment:

- Is the answer supported by the policy?
- Does the summary retain the important facts?
- Is the response clear and appropriately cautious?

Calibrate model graders against human labels. A model agreeing with itself is
not validation.

## Separate a prototype from a production commitment

A prototype should test the largest uncertainty. It does not need every
production feature, but it should use the real shape of the system:

- realistic inputs;
- the actual API or a close adapter;
- typed outputs;
- representative tools and data;
- logging of quality, latency, tokens, and errors;
- bounded retries and timeouts;
- an evaluation that can be rerun.

Do not spend a week on a polished interface when the unknown is retrieval
quality. Do not build a full agent framework when the unknown is whether the
model can classify the cases.

## Present options as decisions

A useful customer recommendation has this shape:

| Option | Quality | Latency | Cost | Operating work | Main risk |
|---|---:|---:|---:|---:|---|
| Simple model call | Medium | Low | Low | Low | Missing private facts |
| Retrieval plus model | High | Medium | Medium | Medium | Weak retrieval |
| Tool-using workflow | High | Medium-high | Medium | High | Unsafe actions |

Recommend one. Say what evidence would make you choose another.

## What senior interviewers listen for

- You clarify before designing.
- You know which fact is measured and which is assumed.
- You can explain why AI is needed.
- You define quality before choosing a model.
- You place human review according to the cost of failure.
- You can build the prototype, not only draw it.
- You have a path from prototype to adoption.

## Review exercise

Take this request:

> "We want an AI assistant that reviews pull requests."

In ten minutes, write:

1. The user and desired outcome.
2. Five discovery questions.
3. The simplest first design.
4. Ten evaluation cases.
5. One blocking safety rule.
6. A two-week proof-of-concept decision.

If the answer begins with a model name, start again.

## Go deeper

- [Deciding What to Build with AI](../04-leadership/17-deciding-what-to-build.md)
- [Evaluation](../01-foundations/04-evaluation.md)
- [AI Product Sense](../04-leadership/18-ai-product-sense.md)
- [From Proof of Concept to Production](../07-enterprise-architecture/34-poc-to-production.md)
