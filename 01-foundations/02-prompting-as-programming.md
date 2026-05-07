---
layout: default
title: "02 — Prompting as Programming"
nav_order: 2
parent: "Foundations"
---

# Prompting as Programming

Here's the reframe that changed how I think about this: a prompt is a program written in natural language, executed by a probabilistic interpreter.

It's not magic. It's not "just text." It's a specification — in a language the model was trained to follow — that steers a probability distribution toward outputs you want. The more you treat prompts like code (with inputs, outputs, contracts, versioning, testing, debugging), the better your systems will be.

Most production AI failures trace back to treating prompts as prose you can "tweak" rather than contracts you can specify and verify. I've seen teams ship a prompt that works for six months, then a model provider releases an update, and suddenly 5% of responses break downstream parsing. Nobody noticed for two weeks because there was no regression test.

Your prompt is an API contract between your system and the model. Treat it like one.

## What a production prompt looks like

A well-structured prompt has identifiable parts. Different providers format them differently, but the parts are universal:

1. **Role / system prompt** — who the model is, what it cares about, what it won't do
2. **Task specification** — the concrete job
3. **Context** — information the model needs (retrieved docs, user state, prior conversation)
4. **Examples** — optional input-output pairs that anchor the task
5. **Input** — the specific thing to process right now
6. **Output format** — the exact shape the response should take
7. **Constraints** — what the model must not do

Here's a skeleton that works:

```
SYSTEM:
You are a compliance analyst. You summarize privacy incidents
for executives. You are precise, factual, and never speculate.

TASK:
Summarize the incident report below in three sections:
- Impact (who was affected, how)
- Root cause (what went wrong)
- Remediation (what has been or will be done)

CONSTRAINTS:
- Each section is 1-3 sentences.
- If the input lacks information for a section, write "Not available in report".
- Do not infer intent.

OUTPUT FORMAT:
Return JSON with keys: impact, root_cause, remediation.

INPUT:
<incident_report>
{report_text}
</incident_report>
```

The key insight: separate the reusable parts (system, task, format) from the per-request parts (input). Check the reusable parts into source control. Tag releases. Diff them across model versions. This isn't over-engineering — it's the minimum viable discipline for production.

## What actually works in production

I've seen a lot of prompting advice. Most of it is noise. These are the principles that survive contact with real systems:

**Be specific about output format.** "Return JSON" is weak. "Return JSON with keys `impact`, `root_cause`, `remediation`, each a string of 1-3 sentences" is strong. The more deterministic you need the output, the more specification you need in the prompt.

**Use structural markers.** XML tags, markdown headers, clear delimiters — they help the model distinguish parts of the prompt. This is especially important when the prompt includes untrusted user input. You want the model to treat that input as data, not instructions.

**Put stable content first, variable content last.** Prompt caching (offered by Anthropic, OpenAI, and others) only works if the prefix is identical across requests. System prompt and task spec at the top; per-request data at the bottom.

**Give the model a way out.** If the input doesn't fit the task, tell the model what to do. "If the report lacks relevant information, write 'Not available in report'." Otherwise it'll make something up to complete the pattern. This is one of the most common causes of hallucination in production.

**Ask for reasoning when it helps, suppress it when it doesn't.** "Think step by step" genuinely improves reasoning tasks. For simple classification, it wastes tokens and can actually hurt accuracy by giving the model more rope to talk itself into the wrong answer. Measure, don't assume.

**Few-shot examples are expensive and powerful.** Each example uses tokens (cost + latency) but can dramatically improve performance on novel tasks. Curate them carefully — bad examples actively hurt.

## What doesn't work (despite what tutorials say)

- **"Act as an expert in X"** — no measurable effect on modern instruction-tuned models. They're already instruction-tuned. Just describe the task.
- **Emotional prompts** ("This is very important to my career") — ethically dubious, marginal effect on current frontier models.
- **"Take a deep breath"** — was briefly popular, mostly a fad.
- **Threatening the model** — don't.

The real improvements come from structure, specificity, and examples. Not clever incantations.

## Prompt injection (the short version)

If your prompt includes user input and the user writes "ignore all previous instructions," current LLMs are not guaranteed to resist. This is prompt injection — the SQL injection of the LLM era.

Quick defenses:
- Treat user input as untrusted data, marked with structural delimiters
- Don't concatenate user input into instructions
- For critical decisions, don't let user input reach the model at all
- Accept that perfect defense isn't possible today; design so a successful injection can't do catastrophic damage

There's a full chapter on security later. For now just know: system prompts are not sandboxes, and "the model will refuse" is not a security control.

## Versioning and testing

Prompts should be versioned like code. Minimum viable setup:

```
prompts/
  incident_summarizer/
    v1.md
    v2.md
    CHANGELOG.md
    eval_cases.jsonl
```

When you change a prompt, run the eval harness and compare. If v2 regresses on any case, you know before shipping. Many teams treat prompts as strings in code, change them casually, and discover weeks later that something broke. The cost of versioning is small; the cost of not versioning is painful.

## Prompts and model versions

Prompts are coupled to specific model versions. A prompt tuned on `claude-3-opus` may behave subtly differently on `claude-3.5-sonnet` and radically differently on `claude-3-haiku`.

This means:
- Pin model versions in production. Don't just say "latest."
- When you upgrade models, run your eval harness first.
- Be prepared for some prompts to need re-tuning across upgrades.

Think of model upgrades the way you think of runtime or library upgrades — compatible in expectation, never guaranteed, and you need a regression test suite.

## Things that trip people up

**The cat cookbook problem.** A prompt that works on your 20 test cases may fail in ways you didn't imagine on real user inputs. The input space is effectively infinite; your eval set is tiny. This is the central challenge of AI engineering.

**"The model understood me yesterday."** You see the model do something impressive on a tricky input, assume it handles the general case, then watch it fail on a slightly different input. LLMs are not doing robust generalization the way a human does. Test the distribution, not the point.

**Overconfidence bleed.** If you tell the model to be "confident and authoritative," it will be — including when it's wrong. Calibration is an open research problem.

**System prompts are not secrets.** Don't put API keys or sensitive data in your system prompt. With enough effort, users can extract them.

**Verbose prompts aren't always better.** More specification helps up to a point. Past that, the prompt becomes so long the model loses track of instructions ("lost in the middle" effect). If your prompt is 8K tokens, you have a design problem, not a specification problem.

## Honest take on the state of things

Prompt engineering is currently a craft. There's real skill in it, but the field is still learning which techniques transfer, which are model-specific, and which are superstition. Newer models need less prompt engineering than older ones. Over time, "prompting as programming" may give way to "just describe what you want" — but we're not there yet for production-grade reliability.

The people who are good at this tend to be good at writing clear specifications in general. If you can write a good API doc or a clear requirements document, you can probably write good prompts. The skill transfers.

## Where to go deeper

- **Anthropic's prompt engineering guide** (in their docs). Practical, current, from people who build the models.
- **OpenAI's prompt engineering best practices.** Different voice, complementary.
- **"The Prompt Report"** (Schulhoff et al., 2024). Academic survey — good for taxonomy if you want the full landscape.
- Next chapter: **Context Engineering** — prompts are one lever. Context (retrieved documents, memory, tool outputs) is the bigger one.

---

[← Previous](01-llm-mental-model.html){: .mr-4 }[Next: Context Engineering →](03-context-engineering.html)
