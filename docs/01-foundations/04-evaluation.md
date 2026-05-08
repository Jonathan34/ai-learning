# Evaluation

This is the chapter most people skip and then regret.

Evaluation is how you know your AI system works. In traditional software, you write unit tests with expected outputs. In LLM systems, you can't do that — a summary can be correct in a hundred different ways and wrong in a million subtle ones. There's no single "right answer" to compare against.

Every team I've seen ship LLM features in production hits the same wall: "we changed the prompt and we think it's better, but we can't actually prove it." That's an evaluation problem.

## Why it's hard

In normal software testing, you check: does the output match the expected output? With LLMs, the output is free-form text. Two completely different responses can both be correct. A response can be mostly right but wrong on one detail. A response can sound perfect but be entirely made up.

You need new approaches. Here are the ones that work.

## Three different things people call "evaluation"

These get mixed up constantly:

**Model benchmarks** — measuring how capable a model is in general (can it do math? code? follow instructions?). This is what papers report. It helps you pick a model but doesn't tell you if your system works.

**Task evaluation** — measuring whether your specific system produces good outputs for your specific inputs. This is what matters for production. There's no off-the-shelf benchmark for "does my customer support bot actually help our customers."

**Online monitoring** — measuring quality on live traffic over time. Catches drift and regressions that offline tests miss.

Most of this chapter is about task evaluation — the one you have to build yourself.

## What you need (minimum viable eval)

Three pieces:

**1. A test set.** A collection of inputs paired with some definition of "correct." This could be:
- Exact expected outputs (for classification or extraction)
- Properties the output must have ("must be valid JSON," "must mention the source document," "must be under 200 words")
- Human judgments ("is this response helpful? yes/no")

Start with 20-50 examples. That's enough to catch obvious regressions. Grow to 200+ as the system matures.

**2. A way to run your system on those inputs.** Your actual production prompt, context assembly, and model — not a simplified version. If your eval runs against a different setup than production, it's not testing what you think.

**3. A scorer.** Something that looks at each output and decides: good or bad?

## Scoring approaches (from cheapest to most expensive)

**Exact match.** Does the output equal the expected string? Works for classification, extraction, structured outputs. Cheap and reliable. Add tolerance for whitespace, casing, and minor formatting differences.

**Property checks.** Is it valid JSON? Does it contain a citation? Is it under 200 words? Does it avoid forbidden phrases? These are deterministic — fast and free. Run them first as a cheap filter.

**Embedding similarity.** Convert the output and a reference answer into vectors (using an embedding model) and measure how similar they are. Better than string matching for open-ended text, but still a rough signal.

**LLM-as-judge.** Use a language model to evaluate the output. "Given this input and this output, is the response accurate and helpful? Rate 1-5." Flexible and powerful, but has its own failure modes (see below).

**Human review.** A person reads the output and rates it. The gold standard, but slow and expensive. Use it to calibrate your automated scorers, not as your primary eval.

In practice, stack these: property checks first (cheap, catches obvious failures), then LLM-as-judge or embedding similarity for quality, with periodic human review to make sure your automated scores are trustworthy.

## LLM-as-judge: useful but tricky

Using one LLM to evaluate another LLM's output is now the standard approach for scoring open-ended responses. It works, but has specific failure modes:

**Self-preference.** If you use the same model to generate and judge, it rates its own outputs higher. Use a different model as the judge.

**Position bias.** When comparing two responses ("which is better, A or B?"), the model tends to prefer whichever comes first. Fix: randomize the order, run both orderings, only count cases where the judgment is consistent.

**Verbosity bias.** Longer responses get rated higher even when they're not better. The judge confuses "more words" with "more helpful."

**Vague rubrics fail.** "Rate this 1-10" gives inconsistent, clustered scores. Specific questions work much better: "Does this response cite at least one source document? Yes/No." "Does this response answer the user's actual question? Yes/No."

To check if your judge is trustworthy: have humans rate 50-100 examples, then compare the judge's ratings to the human ratings. If they agree 85%+ of the time, the judge is probably reliable enough. If agreement is below 75%, the judge needs work.

## Offline eval vs online eval

**Offline eval** runs your test set before you ship changes. It's fast, repeatable, and catches most regressions. But it only tests the cases you thought of — it's blind to everything else.

**Online eval** measures quality on real traffic after you ship:
- **Shadow mode** — run the new version alongside the old one, compare outputs without showing users the new version
- **Canary** — send a small percentage of traffic to the new version, monitor quality
- **Sampling for human review** — randomly pick 1-5% of production responses and have someone rate them over time

You need both. Offline eval for fast iteration; online eval for catching things your test set missed.

## Drift: four kinds

Your system's quality can degrade over time even if you don't change anything:

- **Model drift** — the provider updates the model. Even "same version" can shift subtly.
- **Prompt drift** — someone tweaks the prompt without running the eval. Small changes accumulate.
- **Data drift** — users start asking different questions than they used to. Your test set no longer represents real traffic.
- **Eval drift** — your test set itself changes as people add and remove cases.

If your quality score drops, it could be any of these. Track them separately so you know what to fix.

## Evaluating agents (harder)

Everything above assumes a simple input → output system. Agents are different — they take multiple steps, call tools, make decisions along the way.

To evaluate an agent, you need to think about:
- **Final output quality** — did it get the right answer?
- **Efficiency** — did it take 3 steps when 1 would have worked?
- **Safety** — did it avoid calling tools it shouldn't have? Did it stay within its budget?
- **Robustness** — what happens when a tool call fails?

Most teams evaluate agents on final output quality plus a few instrumented checks (step count, tools used, errors encountered). Full trajectory evaluation — scoring every decision the agent made — is still an open research problem.

## Things that trip people up

**"We'll add eval later."** By the time you add it, you've already shipped quality problems. Build eval before or alongside the feature, even if it's small.

**Overfitting to the test set.** If you tune your prompt until it aces all 50 test cases, you've probably overfit. Hold some cases back for validation.

**Eval as a one-time thing.** You built an eval, it passed, you shipped. Six months later everything has drifted. Run eval continuously — at minimum on every prompt or model change.

**Confusing "parseable" with "correct."** The model returns `Here's the JSON: {"answer": "..."}` instead of just the JSON. Your parser fails. Separate "did the output have the right format?" from "was the content right?"

**Underestimating the cost.** On a mature system, evaluation can take 20-40% of engineering time. That's normal. It's the cost of reliability in a domain where you can't write unit tests.

## Where things stand

Evaluation is the least mature part of the AI stack. Tools are improving (Braintrust, Langfuse, Promptfoo, Arize Phoenix) but there's no "just use X" answer. Every team builds custom infrastructure.

The upside: if you invest in eval early, it becomes a real advantage. Teams with good eval iterate faster because they can tell what's working. Teams without it are guessing.

## Go deeper

- **[Workshop W3 — Real Evaluation](../05-workshops/W3-real-evaluation.md).** Build an eval harness end-to-end.
- **"Judging LLM-as-a-Judge"** (Zheng et al., 2023) — research on LLM judge failure modes
- **Anthropic's evaluation docs** — practical patterns
- **Braintrust blog** — production eval patterns
