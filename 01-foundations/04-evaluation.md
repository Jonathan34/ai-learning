---
title: "04 — Evaluation"
nav_order: 4
parent: "Foundations"
---
# 04 — Evaluation: The Hardest Unsolved Problem

## The mental model

**Evaluation is how you know your AI system works. In traditional software, you write unit tests. In LLM systems, you don't have that luxury, and if you ship without a substitute, you're flying blind.**

The core problem: LLM output is a sample from a probability distribution over an effectively infinite output space. There's no "correct answer" to diff against. A summary can be correct in 100 different ways and wrong in a million subtle ones. Your eval set is tiny; the real input distribution is vast.

Every senior engineer I've worked with who's shipped LLM features in production has converged on roughly the same conclusion: evaluation is the bottleneck, the work that keeps teams from moving fast is almost always "we don't have a way to tell if this change made things better or worse."

**The thing to remember:** evaluation is not a phase you do before shipping. It's infrastructure you build early and maintain continuously, because every prompt change, model upgrade, and user distribution shift can regress quality in ways you won't notice without it.

## Why this matters in production

Three common failures trace back to missing or weak evaluation:

- **"It worked in the demo."** A team ships a feature based on 5 impressive examples. Production is the long tail — user queries you didn't anticipate, edge cases, adversarial inputs. Without structured eval, you discover quality problems from user complaints, which is the worst feedback loop.
- **"We can't upgrade the model."** The team pinned a model version a year ago and is afraid to upgrade because they don't know what will break. Meanwhile, newer models are faster and cheaper. The missing piece is a regression harness.
- **"We don't know if this change helps."** Someone tweaks a prompt. Qualitatively it seems better. Ships it. Weeks later, someone notices a regression on a different class of inputs that was working fine before. The prompt change caused it. No one knows.

All three are expensive in different ways, and all three are solved by investing 20-40% of engineering time in eval infrastructure early.

## Three things people confuse

**ML evaluation** is about measuring how well a trained model does on a task, usually with accuracy or F1 on a labeled test set. This is what your ML friends do. It's largely irrelevant for most applied AI work — you're not training models, you're using them.

**LLM evaluation** is about measuring the base capability of a model: its reasoning, coding, safety, instruction-following. This is what benchmark papers measure (MMLU, HumanEval, GPQA, etc.). This is vendor-facing; it informs your choice of model, not your production work.

**LLM system evaluation** is about measuring whether *your* specific system works for *your* specific users on *your* specific inputs. This is the one that matters for production, and it's the hardest — there's no off-the-shelf benchmark for "does my customer support agent actually help our customers."

Most of this chapter is about system evaluation.

**Senior move:** when someone claims "model X is better than model Y at task Z based on benchmark B," ask what's in their production eval set. If they can't answer, their claim is worth less than they think.

## What a production eval looks like

At minimum, an evaluation harness has three pieces:

1. **A test set.** Representative inputs, paired with either ground truth outputs or properties you can check (e.g., "this output must be valid JSON with these keys," "this output must mention at least one of the source documents").
2. **A way to run the system under test.** Your production prompt, context assembly, model, and post-processing — as close to prod as you can get.
3. **A scorer.** Something that decides whether each output is acceptable. This is where it gets hard.

The test set is the bottleneck for most teams. A good test set needs to be:
- **Diverse enough to represent real inputs.** If your production users ask questions in French, your test set has French.
- **Large enough to distinguish signal from noise.** 20 examples is usually too few for subtle changes; 200 is often enough; 1000 is solid for most applied work.
- **Stable but evolvable.** You want the same test set across runs for regression testing, but you also need to add new cases as production surfaces them.
- **Labeled with what "correct" means.** Hardest part. Depending on the task, "correct" might be an exact string, a set of acceptable answers, a structural property, or a human judgment.

**Gotcha:** teams often use their demo cases as their test set. That's how they know the feature works. Of course it works on those cases. You need the cases that you haven't thought of yet — which is why production traffic, suitably anonymized, is usually your best source of new test cases over time.

## Scoring strategies

Depending on the task, you have different options:

**Exact match.** Works for classification, extraction with a small output space, structured responses. Cheap, reliable, underused because people assume it won't work — often it does once you add a tolerance layer (case-insensitive, whitespace-normalized, synonym-aware).

**Structural / property checks.** "Is this valid JSON?" "Does it contain a citation?" "Is it under 200 words?" "Does it avoid the forbidden phrases?" These are deterministic and fast. Run them first as cheap filters before any expensive judgment.

**Reference-based metrics.** BLEU, ROUGE, BERTScore. Borrowed from machine translation and summarization research. They measure overlap between generated and reference text. Cheap. Weak — they correlate imperfectly with human judgment — but useful as directional signals.

**Embedding similarity.** Embed the generated output and a reference answer; compare cosine similarity. Better than ROUGE for semantic similarity; still a rough signal.

**LLM-as-judge.** Use an LLM to score the output. Flexible, powerful, expensive, and fraught with subtle failures (see next section).

**Human evaluation.** The ground truth, but slow and expensive. You use it to calibrate your automated scorers, not as your primary eval.

Production systems typically stack these: cheap structural checks first, then either an LLM-as-judge or embedding similarity, with periodic human-labeled samples for calibration.

## LLM-as-judge: powerful and treacherous

Using an LLM to evaluate another LLM's output has become the default because it's the only scalable way to score open-ended outputs on qualitative dimensions (helpfulness, accuracy, style). It works. It also has specific failure modes you need to know.

**What it's good at:**
- Pairwise comparison ("which of these two responses is better?") — more reliable than absolute scoring
- Factual accuracy checks against a reference document
- Detecting refusals, incomplete responses, format violations
- Checking specific properties ("does this response include a recommendation?")

**Where it fails:**
- **Self-preference bias.** A model scoring its own outputs rates them higher. Use a *different* model as the judge.
- **Position bias.** When comparing A vs B, the model often prefers whichever comes first. Randomize order; evaluate both orderings; take consistent judgments.
- **Verbosity bias.** Longer responses are rated as higher quality even when they're not. Control for length or calibrate.
- **Confidence bias.** Confident-sounding responses are rated higher even when wrong. This is alignment-of-human-preferences-eaten-by-the-same-bias.
- **Narrow rubric failure.** If you ask for "rate this 1-10," you'll get inconsistent, clustered ratings. Specific rubrics ("does the response cite at least one of the provided sources? Yes/No") are far more reliable.

**Senior move:** always calibrate your LLM judge against a small set of human-labeled examples. Measure the agreement rate (Cohen's kappa or similar). If the judge and humans agree 85%+ of the time on clear cases, your judge is probably OK for production. If it's 70%, it's not ready.

## Offline vs online evaluation

**Offline eval** runs against your test set. It's fast, reproducible, and you can run it on every change. It's also blind to everything that's in production but not in your test set — which is usually most of it.

**Online eval** measures quality on live traffic. Options:
- **Shadow mode.** Your new system runs alongside the current one; users see the current output; you compare. Safe but costly (you pay for every call twice).
- **Canary.** Route a small percentage of traffic to the new system. Measure quality signals (user feedback, follow-up rates, task completion) and roll forward or back.
- **A/B tests.** Proper randomized comparison between two versions. Requires volume and statistical discipline.
- **Continuous sampling for human review.** Sample 1-5% of production calls for human rating. Tracks quality over time without gating rollouts.

**Most teams need both.** Offline eval for rapid iteration; online eval for ground truth on what actually happens in production. Skipping either is a mistake, but if you can only do one, start with offline — it's cheaper and catches most regressions.

## Drift: four kinds to watch for

- **Model drift.** The provider updates the model. Anthropic, OpenAI, Google all release updates. Even "same model version" can subtly shift as they patch safety issues or infrastructure. Run your eval suite before and after any suspected change.
- **Prompt drift.** Your prompt evolves over time as people tweak it. Without versioning and re-eval, you accumulate regressions you don't know about.
- **Data drift.** The inputs users send change — new topics, new edge cases, new languages. Your test set from six months ago doesn't reflect current traffic.
- **Eval set drift.** Your eval set itself drifts if it's not curated. People add cases for bugs that got fixed, retire cases that are "too easy," and the eval set stops representing production.

**Senior move:** track all four separately. A dashboard with "eval score over time" that drops could be a model regression, a prompt regression, or an eval set change. If you don't know which, you can't fix it.

## Evaluating agents is much harder

Everything above assumes a single-turn system: input → output, score the output. Agents don't work that way. An agent takes a goal, makes multiple decisions, calls tools, updates state, and may take hundreds of tokens of intermediate reasoning before producing a final answer.

To evaluate an agent, you need to decide:

- **Is the final output correct?** Easier to score, but hides the failure modes along the way.
- **Is the trajectory efficient?** Did it take 3 steps when 1 would do? Did it call the wrong tool?
- **Did it avoid unsafe actions?** Did it hit tool budget limits, try forbidden operations, leak sensitive data?
- **Is it robust?** Does it fail gracefully when a tool errors, or does it spiral?

Trajectory-level evaluation is still an open research problem. Most production teams evaluate on final outputs plus a handful of instrumented properties (step count, tools used, error rate) and accept that they're not capturing everything.

**Gotcha:** a purely output-focused eval can mask catastrophic intermediate behaviors. An agent that happens to produce the right answer after reading the user's private files is worse than one that produces the wrong answer without reading anything. Safety-relevant evals need to check the trajectory.

## Measuring hallucination

Hallucination — confidently false output — is inherent to LLMs. You can't eliminate it. You can measure it, design around it, and reduce it.

- **Faithfulness / groundedness** measures whether the output is supported by the provided context. Implementation: for each claim in the output, check whether it's present in the source. Can be done with an LLM judge or retrieval-based verification.
- **Citation accuracy** measures whether references the model provides actually support the claims. Important in RAG systems where the user may rely on cited sources.
- **Refusal calibration** measures whether the model correctly says "I don't know" when it doesn't know. This is the dual of hallucination — a model that always answers is hallucinating; a model that correctly refuses is doing its job.

A well-built RAG eval includes all three of these, not just "is the answer correct."

## The tooling landscape

As of late 2025, the specialized tools are mostly:

- **Braintrust** — hosted, strong eval focus, good UX for managing test sets and running LLM-as-judge
- **Langfuse** — open-source with hosted offering; tracing + eval
- **LangSmith** — LangChain's companion; tightly integrated if you're in that ecosystem
- **Promptfoo** — CLI-first, open-source, simple
- **Inspect** (UK AI Safety Institute) — research-grade evals, more rigorous than most commercial tools
- **Arize Phoenix** — open-source, strong on traces and evals together
- **OpenAI Evals** — reference implementation, widely used

You don't need all of them. Pick one. The important part is having a process, not the specific tool.

**Senior move:** most teams end up with a mix — a specialized tool for the eval harness, plus custom Python scripts for the project-specific parts (custom scorers, test case generation, reporting). Tools are good; tools that fit your workflow are better; tools that force you to fit their workflow are a trap.

## Gotchas

**Gotcha: "We'll eval it once we have users."**

No. By then the feature is shipped, quality problems are public, and rebuilding evals on production traffic is painful. Build eval first, even if it's small.

**Gotcha: LLM-as-judge with the same model you're evaluating.**

Evaluating GPT-4 with GPT-4 as the judge is systematically optimistic. Use a different model family for the judge, or calibrate carefully.

**Gotcha: Public benchmarks in training data.**

Many frontier models have seen MMLU, HumanEval, and similar benchmarks during training. Strong performance on public benchmarks can be memorization, not generalization. Always have a private eval set.

**Gotcha: The demo works → the feature works.**

If you don't have a structured eval showing performance on a diverse distribution, you have a demo, not a shippable feature. This is the most common failure mode in applied AI teams.

**Gotcha: Overfitting to the eval set.**

If you tune your prompt until it aces all 50 test cases, you've overfit. Hold out some of your data for validation, rotate, and keep looking for cases you haven't seen.

**Gotcha: Eval as a one-time exercise.**

You built an eval before shipping, it passed, you shipped. Six months later, usage has drifted, models have updated, prompts have been tweaked. The eval is stale. If you don't run the eval continuously — at least on every prompt/model change — it doesn't help you.

**Gotcha: Scoring on outputs you can't parse.**

If your model sometimes returns "Here's my answer: {json}" instead of just `{json}`, your parser fails, and your scorer counts it as a failure — even though the content may be correct. Always separate "did the model output something parseable" from "was the content right."

**Gotcha: Cost of good eval.**

On a mature system, evaluation can easily consume 20-40% of engineering time. This is not a sign something is wrong — it's the cost of reliability in this domain. Budget for it.

## Honest status

Evaluation is the least mature part of the applied AI stack. The tools are getting better, the patterns are emerging, but there's no "just use X" answer yet. Every production team is building custom infrastructure. Expect this to remain true for another year or two; in the meantime, invest deliberately and be proud of the infrastructure you build — it's a significant moat.

Research on evaluation is active and relevant. Papers on LLM-as-judge calibration, agent evaluation, and hallucination detection are closer to production-ready than most ML research. Following this literature is one of the higher-signal-to-noise things you can do.

## What to read next

- **05 — Security and Safety.** Evaluation and safety intersect at red-teaming; the next chapter builds on the eval mindset.
- **Workshop W3 — Real Evaluation.** Build an eval harness end-to-end. Nothing teaches this like doing it.
- **"Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena"** (Zheng et al., 2023) — foundational work on LLM judges and their failure modes.
- **Anthropic's evaluation documentation** — practical, from people who do this seriously at scale.
- **Braintrust's blog on evals** — concrete production patterns.
