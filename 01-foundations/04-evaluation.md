---
layout: default
title: "04 — Evaluation"
nav_order: 4
parent: "Foundations"
---

# Evaluation

This is the chapter most people skip and then regret. Evaluation is how you know your AI system works — and in LLM systems, it's genuinely hard because there's no "correct answer" to diff against.

A summary can be correct in a hundred different ways and wrong in a million subtle ones. Your eval set is tiny; the real input distribution is vast. Every senior engineer I've talked to who's shipped LLM features in production has converged on the same conclusion: evaluation is the bottleneck. The thing that keeps teams from moving fast is almost always "we don't have a way to tell if this change made things better or worse."

So let's talk about how to actually do it.

## Three things people confuse

**ML evaluation** — measuring how well a trained model does on a task (accuracy, F1 on a labeled set). This is what your ML friends do. Mostly irrelevant for applied AI work since you're not training models.

**LLM evaluation** — measuring base model capability (MMLU, HumanEval, GPQA). This is what benchmark papers measure. It informs your choice of model, not your production work.

**LLM system evaluation** — measuring whether *your* specific system works for *your* specific users on *your* specific inputs. This is the one that matters, and it's the hardest because there's no off-the-shelf benchmark for "does my customer support agent actually help our customers."

Most of this chapter is about system evaluation.

## What you actually need

At minimum, an evaluation harness has three pieces:

**A test set.** Representative inputs, paired with either ground truth outputs or checkable properties. A good test set is diverse (covers real usage patterns), large enough to distinguish signal from noise (20 examples is usually too few; 200 is often enough), and stable but evolvable (same set for regression testing, but you add new cases as production surfaces them).

The hardest part is labeling what "correct" means. Depending on the task, that might be an exact string, a set of acceptable answers, a structural property, or a human judgment call.

**A way to run the system under test.** Your production prompt, context assembly, model, and post-processing — as close to prod as you can get. If your eval runs against a different prompt than production uses, it's not testing what you think it's testing.

**A scorer.** Something that decides whether each output is acceptable. This is where it gets interesting.

## Scoring strategies

Depending on the task:

**Exact match.** Works for classification, extraction with a small output space, structured responses. Cheap, reliable, underused. People assume it won't work — often it does once you add tolerance (case-insensitive, whitespace-normalized, synonym-aware).

**Structural checks.** "Is this valid JSON?" "Does it contain a citation?" "Is it under 200 words?" Deterministic and fast. Run these first as cheap filters before anything expensive.

**Embedding similarity.** Embed the output and a reference; compare cosine similarity. Better than string matching for semantic similarity; still a rough signal.

**LLM-as-judge.** Use an LLM to score the output. Flexible, powerful, expensive, and has specific failure modes (see below).

**Human evaluation.** The ground truth, but slow and expensive. Use it to calibrate your automated scorers, not as your primary eval.

Production systems typically stack these: cheap structural checks first, then LLM-as-judge or embedding similarity, with periodic human-labeled samples for calibration.

## LLM-as-judge: powerful and treacherous

Using an LLM to evaluate another LLM's output has become the default because it's the only scalable way to score open-ended outputs. It works. It also has specific failure modes you need to know about.

Where it's good:
- Pairwise comparison ("which response is better?") — more reliable than absolute scoring
- Factual accuracy checks against a reference document
- Detecting refusals, incomplete responses, format violations
- Checking specific properties ("does this mention a recommendation?")

Where it fails:
- **Self-preference bias.** A model scoring its own outputs rates them higher. Use a different model as the judge.
- **Position bias.** When comparing A vs B, the model prefers whichever comes first. Randomize order; evaluate both orderings.
- **Verbosity bias.** Longer responses get rated higher even when they're not better.
- **Narrow rubric failure.** "Rate this 1-10" gives inconsistent, clustered ratings. Specific rubrics ("does the response cite at least one source? Yes/No") are far more reliable.

Always calibrate your LLM judge against human-labeled examples. Measure agreement (Cohen's kappa or similar). If judge and humans agree 85%+ on clear cases, you're probably OK. If it's 70%, the judge isn't ready.

## Offline vs online eval

**Offline eval** runs against your test set. Fast, reproducible, catches most regressions. Blind to everything in production that's not in your test set — which is most of it.

**Online eval** measures quality on live traffic:
- Shadow mode (new system runs alongside current; compare outputs)
- Canary (small % of traffic to new version; measure quality signals)
- A/B tests (proper randomized comparison)
- Continuous sampling for human review (1-5% of calls, rated over time)

You need both. Offline for rapid iteration; online for ground truth. If you can only do one, start with offline — it's cheaper and catches most regressions.

## Drift

Four kinds, all real:

- **Model drift.** Provider updates the model. Run your eval before and after.
- **Prompt drift.** Your prompt evolves as people tweak it. Without versioning and re-eval, you accumulate regressions.
- **Data drift.** User inputs change over time. Your test set from six months ago doesn't reflect current traffic.
- **Eval set drift.** Your eval set itself changes as people add and remove cases. Track this separately.

A dashboard showing "eval score over time" that drops could be any of these. If you don't know which, you can't fix it.

## Evaluating agents (much harder)

Everything above assumes single-turn: input → output, score the output. Agents don't work that way. An agent takes a goal, makes multiple decisions, calls tools, and may take hundreds of tokens of intermediate reasoning.

To evaluate an agent you need to decide:
- Is the final output correct?
- Is the trajectory efficient? (Did it take 3 steps when 1 would do?)
- Did it avoid unsafe actions?
- Is it robust when tools fail?

Trajectory-level evaluation is still an open research problem. Most production teams evaluate on final outputs plus instrumented properties (step count, tools used, error rate) and accept they're not capturing everything.

One important point: a purely output-focused eval can mask catastrophic intermediate behaviors. An agent that produces the right answer after reading the user's private files is worse than one that produces the wrong answer without reading anything. Safety-relevant evals need to check the trajectory.

## The tooling landscape

As of late 2025: Braintrust, Langfuse, LangSmith, Promptfoo, Arize Phoenix, Inspect (UK AI Safety Institute), OpenAI Evals. Pick one. The important part is having a process, not the specific tool.

Most teams end up with a mix — a specialized tool for the harness, plus custom Python for project-specific scorers, test case generation, and reporting. Tools are good; tools that fit your workflow are better; tools that force you to fit their workflow are a trap.

## Things that trip people up

**"We'll eval it once we have users."** By then the feature is shipped and quality problems are public. Build eval first, even if it's small.

**Overfitting to the eval set.** If you tune your prompt until it aces all 50 test cases, you've overfit. Hold out some data for validation.

**Eval as a one-time exercise.** You built an eval, it passed, you shipped. Six months later everything has drifted. If you don't run eval continuously, it doesn't help you.

**Scoring on outputs you can't parse.** The model returns "Here's my answer: {json}" instead of just the JSON. Your parser fails, scorer counts it as wrong. Separate "did the model output something parseable" from "was the content right."

**Cost of good eval.** On a mature system, evaluation can consume 20-40% of engineering time. This is not a sign something is wrong — it's the cost of reliability in this domain. Budget for it.

## Where things stand

Evaluation is the least mature part of the applied AI stack. The tools are getting better, the patterns are emerging, but there's no "just use X" answer yet. Every production team is building custom infrastructure. Expect this to remain true for another year or two.

The good news: if you invest in eval early, it becomes a genuine competitive advantage. Teams with good eval iterate faster because they can tell what's working. Teams without it are guessing.

## Where to go deeper

- **Workshop W3 — Real Evaluation.** Build an eval harness end-to-end.
- **"Judging LLM-as-a-Judge"** (Zheng et al., 2023). Foundational work on LLM judges and their failure modes.
- **Anthropic's evaluation documentation.** Practical, from people who do this at scale.
- **Braintrust's blog on evals.** Concrete production patterns.
- Next chapter: **Security and Safety** — evaluation and safety intersect at red-teaming.

---

[← Previous](03-context-engineering.html){: .mr-4 }[Next: Security and Safety →](05-security-and-safety.html)
