---
layout: default

title: "01 — How LLMs Actually Work"
nav_order: 1
parent: "Foundations"
---
# 01 — How LLMs Actually Work (The Mental Model a Senior Engineer Needs)

## The mental model

**An LLM is a function from a sequence of tokens to a probability distribution over the next token. That's it.**

Everything else — chat, instruction following, tool use, agents, reasoning — is built on top of that core operation applied repeatedly.

When you call the model with a prompt, the model doesn't "read" it the way you do. It converts your text into tokens (sub-word units), computes a forward pass through the network, and outputs a distribution over the next possible token. A sampling strategy (greedy, top-p, temperature) picks one token, which gets appended to the context. Then the whole thing repeats.

**The token-by-token generation loop is the single most important thing to internalize.** Every practical property of LLMs — their latency curve, their context-window economics, their failure modes, their reasoning style — comes from this loop.

## Why this matters in production

Once you hold this model, a lot of things stop being mysterious:

- **Why are LLMs slow?** Each token requires a full forward pass. A 500-token response is 500 passes. You can't just "make it faster" — you need streaming, prompt caching, or smaller models.
- **Why does context length matter for cost?** Attention in transformer models is roughly O(n²) in context length at inference time. Doubling context doesn't double cost. This is why 128K-context models cost what they do.
- **Why do LLMs lose track of instructions in long contexts?** Attention is a finite budget. Instructions at the start of a 100K-token context compete with everything else for the model's attention.
- **Why do chain-of-thought and "think step by step" work?** Because reasoning costs tokens. More generation = more computation = more chances for the right answer to emerge. It's literally buying more compute.
- **Why do structured outputs sometimes fail?** Because you're asking a probability distribution to produce exact syntax. The model is *biased* toward your schema by the prompt, but it can still sample a token that breaks it.

**This is the thing to remember:** everything else in the stack is about controlling, constraining, or augmenting this loop.

## The components, briefly

You don't need to be able to implement a transformer to be a good AI architect. But you should know the names and what they do.

- **Tokenizer** — converts text to token IDs. Different models use different tokenizers; a Claude tokenizer and a Llama tokenizer will split "strawberry" differently. This matters when you measure context length or do subword analysis.
- **Embedding layer** — maps each token ID to a vector. This is what the rest of the network operates on.
- **Attention** — every token "looks at" every other token and decides how much to weight each one. This is the core mechanism that lets language models handle long-range dependencies.
- **Feed-forward layers** — stateless transformations that happen between attention layers. Most of a model's "knowledge" is stored here, in the form of learned weights.
- **Output projection** — converts the final hidden state back into token probabilities.

**Senior move:** when someone uses "attention" as a buzzword, they usually mean "the model can focus on relevant context." That's not wrong, but it's not the right mental model either. Attention is *every* token computing a weighted sum over every other token. It's not selective focus; it's diffuse weighted mixing. The reason it looks like focus is that most of the weights are small.

## Training, briefly

Three stages, increasingly important as you go down the list for engineers:

1. **Pretraining** — predict the next token on vast amounts of text. Builds the base model. This is what costs hundreds of millions of dollars.
2. **Supervised fine-tuning (SFT)** — train the model on high-quality conversation/instruction data. This teaches the model to follow instructions.
3. **RLHF / preference tuning** — train the model to produce outputs humans prefer, using reinforcement learning from human feedback (or variants like DPO). This is where helpfulness, harmlessness, and honesty come from. This is also where most of the "personality" of a model gets baked in.

You don't need to know how to train one. You need to know that:
- The model's behavior reflects what it saw in training
- The model's "refusals" and "guardrails" are trained, not hard-coded — which means they can be overridden by clever prompting (hence prompt injection attacks)
- Different providers (Anthropic, OpenAI, Google, Meta) make different trade-offs in training, which is why the models feel different

## What you'll hear and what it means

| Jargon | Translation |
|---|---|
| "It's just autocomplete" | Technically true at the base mechanism, but dismissive. The emergent behavior is the point. |
| "It's AGI" | Almost always hype. |
| "The model is reasoning" | The model is generating tokens that look like reasoning. Sometimes this corresponds to correct reasoning, sometimes it's pattern-matched nonsense. Be specific. |
| "Emergent capabilities" | Things the model does well that weren't explicitly trained. Real but often overhyped. |
| "Hallucination" | The model confidently produces false output. Not a bug — an inherent property of generative models. Your job is to design around it, not pretend it's solvable. |
| "Parameters" | Roughly, the model's size. More parameters ≈ more knowledge and capability, with sharply diminishing returns and rising cost. |
| "Context window" | How many tokens the model can attend to at once. |
| "Inference" | Running the trained model to generate output. Different from training. |
| "Frontier model" | The largest, most capable class — GPT-5, Claude 4, Gemini 2.5 Pro, etc. (at time of writing). Changes every few months. |

## Gotchas

**Gotcha: "The model knows X"**

It doesn't. The model has *statistical associations* with X. When it seems to "know" something, what's happening is that the token sequence corresponding to X is strongly associated with the tokens in your prompt. This is why LLMs fail on obscure-but-real facts and succeed on common-but-wrong ones. Never ship a system that relies on the model as a factual database without retrieval.

**Gotcha: Token ≠ word**

A 1,000-token context is roughly 750 English words, but it's ~500 words in French, and much less in CJK languages. Code has its own tokenization properties. When estimating costs, use the actual tokenizer for the model.

**Gotcha: Non-determinism by default**

At temperature > 0, the same prompt will produce different outputs. Even at temperature = 0, subtle floating-point or hardware variations can produce different outputs across runs. If your tests require determinism, design for it explicitly (fixed seed, temperature 0, and accept some residual variance).

**Gotcha: Instruction-following is soft**

The model is probabilistically biased toward following your instructions, not guaranteed to. A 99% success rate looks great in a demo and fails 1 in 100 times in production. This is why eval harnesses matter.

**Gotcha: Context ≠ memory**

Inside a single completion, the model sees the whole context. Across completions, it remembers nothing. "Memory" in an agent is a design pattern (state stored outside the model, rehydrated into the next prompt), not a model capability.

## Honest status

The field's understanding of *why* LLMs do what they do is incomplete. Mechanistic interpretability — research into what specific parts of a model compute — is an active frontier. You don't need to track it day-to-day, but know that "we don't fully understand what's happening inside" is the current state, and it's a meaningful constraint on safety, reliability, and debugging.

## What to read next

- **02 — Prompting as Programming.** Now that you know how the token loop works, you can reason about how prompts steer it.
- **Andrej Karpathy's "Deep Dive into LLMs like ChatGPT"** (YouTube). 3 hours, deeper than this chapter, worth it.
- **"The Illustrated Transformer" by Jay Alammar.** Classic. Visual. Skippable if you're not going deeper on architecture.
- **Anthropic's "Mapping the Mind of a Large Language Model"** post. For the interpretability angle.
