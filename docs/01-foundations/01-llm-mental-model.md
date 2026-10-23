# How LLMs Actually Work

You don't need to understand backpropagation to be a good AI architect. But you do need a working mental model of what happens when you call the API.

Here's the core of it: an LLM takes a sequence of tokens and predicts what the next token should be. That's it. Chat, instruction following, tool use, agents, "reasoning" — all built on top of that one operation, repeated in a loop.

When you send a prompt:

1. Your text gets split into tokens (sub-word chunks)

2. The model processes all those tokens through its layers

3. It outputs probabilities for what the next token should be

4. A sampling strategy picks one token

5. That token gets appended to the sequence

6. Go back to step 3, repeat until done

That's the generation loop. Everything else in this curriculum is about controlling, constraining, or augmenting it.

```mermaid
flowchart LR
    A[Input text] --> B[Tokenizer]
    B --> C[Token IDs]
    C --> D[Model layers<br/>forward pass]
    D --> E[Next token<br/>probabilities]
    E --> F[Sampling<br/>pick one token]
    F --> G[Append to sequence]
    G -->|repeat| D
    G -->|done?| H[Final output]

```

## Key concepts

Terms you'll see throughout. Brief definitions here; deeper explanations below.

| Term | What it means |
|---|---|
| **Token** | A sub-word unit. The model sees tokens, not words. "Strawberry" might be 1-3 tokens depending on the model. |
| **Context window** | Max tokens the model can handle at once (input + output combined). |
| **Attention** | The mechanism that lets each token "look at" other tokens to figure out what's relevant. Makes transformers work. Standard dense attention lets every token look at every earlier token, which costs roughly O(n²) in sequence length. Some models limit this to a local window. |
| **Forward pass** | One run of the input through all the model's layers to produce an output. In conventional decoding, each new token takes one forward pass. |
| **Inference** | Running the trained model to generate output. What you do when you call the API. |
| **Temperature** | Controls randomness. 0 = always pick the most likely token. Higher = more variety. |
| **Hallucination** | Model confidently produces false output. Inherent to how generation works — not a bug you can patch. |
| **KV cache** | Stored computation from previous tokens so the model doesn't redo work. Why memory is often the bottleneck for long contexts. |

## Why things work the way they do

Once you have the generation loop in your head, a lot of LLM behavior becomes predictable:

**Why are LLMs slow?** In conventional autoregressive decoding (generate one token, append it, generate the next), each new token needs its own forward pass through every layer. Thanks to the KV cache, the model doesn't reprocess earlier tokens — but it still has to run the new token through all layers, and it can't produce token 501 before it knows token 500. A 500-token response means roughly 500 sequential steps. Some serving stacks shorten that chain with [speculative decoding](https://arxiv.org/abs/2211.17192): a small, cheap model drafts a few tokens ahead, and the big model checks them all in one pass and keeps the ones that pass. The output follows the same distribution the big model would have produced on its own. The paper reports 2-3x speedups over a standard implementation on T5-XXL (Leviathan et al., 2022, checked 2026-10-11). The tokens are still ordered, and you can still stream them to the user, but the work doesn't disappear.

**Why does context length cost money?** Standard dense attention compares every token with every earlier token. For a full pass over the sequence — processing your prompt (called prefill) or training — doubling the length roughly quadruples the attention work. Generation is different: with a KV cache, each new token only compares itself against the stored keys, so its attention work grows roughly linearly with context length. Either way, a 128K-token call costs significantly more than a 4K-token call. But the real cost also depends on the feed-forward layers, memory traffic, batching, and how attention is implemented. [FlashAttention](https://proceedings.neurips.cc/paper_files/paper/2022/hash/67d57c32e20fd0a7a302cb81d36e40d5-Abstract-Conference.html) is a good example: it computes exactly the same dense attention, still O(n²), but moves far less data through GPU memory, and it trained GPT-2 about 3x faster in the paper's tests (Dao et al., NeurIPS 2022, checked 2026-10-11). Big-O tells you the shape of the problem, not the bill.

**Why do models lose track of instructions in long prompts?** Attention spreads across all tokens. Your system prompt at position 0 is competing with 100K tokens of documents for the model's attention budget. Sometimes the documents win.

**Why does "think step by step" help?** More output tokens = more computation. When the model generates reasoning steps, it's doing additional processing that can lead to better answers. You're buying more compute by asking for more output.

**Why do structured outputs sometimes break?** The model picks tokens one at a time based on probabilities. It's biased toward valid JSON by your prompt, but it can still pick a token that breaks the syntax. It's probabilistic, not guaranteed.

## The building blocks

A transformer model is made of layers stacked on top of each other (dozens to hundreds of them). Each layer has two main parts:

**Attention layer** — this is where tokens look at each other. Each token asks "which other tokens are relevant to me?" and pulls information from them. Explained in detail in the next section.

**Feed-forward layer** — a simple neural network that processes each token independently after attention. Think of it as a learned compression of patterns — not an exact lookup table, but a function that transforms tokens based on patterns seen during training. Most of the model's factual "knowledge" lives here, encoded as numerical weights. When people say a model "knows" something, what they really mean is that the feed-forward layers have weights that produce the right tokens in the right context.

These two layers alternate: attention (tokens talk to each other) → feed-forward (each token gets processed individually) → attention → feed-forward → ... repeated many times. By the final layer, the model has enough information to predict the next token.

```mermaid
flowchart TB
    subgraph "Transformer block — repeated N times"
        direction TB
        ATT[Attention Layer<br/>tokens look at each other] --> FF[Feed-Forward Layer<br/>each token processed alone]
    end

    IN[Token embeddings] --> ATT
    FF --> OUT[Next token probabilities]

```

Other pieces:

- **Tokenizer** — splits text into token IDs before anything else happens. Different models use different tokenizers, so the same text produces different token counts.

- **Embedding layer** — converts each token ID into a vector (a list of numbers) that the network can work with.

- **Output layer** — converts the final vector back into probabilities over all possible next tokens.

## Attention: how tokens talk to each other

This is the mechanism that makes transformers special. Older architectures (Recurrent Neural Networks aka RNNs) processed tokens one by one in order. Attention lets every token look at every other token directly — which is why LLMs can connect a pronoun on page 3 to a name on page 1.

Here's how it works:

Each token gets transformed into three vectors:

- **Query (Q)** — "what am I looking for?"

- **Key (K)** — "what do I contain?"

- **Value (V)** — "what information do I provide if you pick me?"

```mermaid
flowchart LR
    subgraph "For each token"
        T[Token] --> Q[Query: what am I looking for?]
        T --> K[Key: what do I contain?]
        T --> V[Value: what info do I give?]
    end

    Q --> |dot product with all Keys| W[Attention weights<br/>relevance scores]
    W --> |weighted sum of Values| O[Updated token<br/>representation]

```

For each token, the model checks how well its Query matches every other token's Key (using a dot product — basically measuring similarity). High match = high attention weight. Then it takes a weighted average of all the Values.

Result: each token's representation becomes a mix of information from all other tokens, weighted by relevance.

Example: in "The cat sat on the mat. It was tired". — the token "It" will attend strongly to "cat" because their Q-K match is high. The model learns these Q/K/V transformations during training.

**Multi-head attention** runs this process multiple times in parallel — typically 32 to 128 independent copies, called "heads". Why multiple? Each head learns to pay attention to different things. One head might learn to track grammatical relationships (subject-verb), another might track semantic similarity (synonyms), another might track nearby tokens. Running many heads in parallel and combining their outputs gives the model a richer understanding than any single attention pass could.

**Causal masking** (during generation): token at position 5 can only attend to positions 0-4, not 6+. This enforces left-to-right generation — the model can't peek at future tokens.

**Sliding-window and sparse attention**: everything above describes standard dense attention, and it's the right mental model to start with. But not every deployed model attends globally. [Mistral 7B](https://arxiv.org/abs/2310.06825), for example, uses sliding-window attention: in each layer, a token only attends to a fixed window of recent positions, which cuts inference cost on long sequences. Information from further back still reaches it indirectly, passed forward through the stacked layers (Jiang et al., 2023, checked 2026-10-11).

### Why this matters for you

- **Cost scales quadratically, for standard dense attention.** Every token attends to every earlier token. Double the context = ~4x the attention computation for a full pass over the sequence. This is why long contexts are expensive. Cached decoding, sliding-window models, and faster kernels all change the real numbers, so treat this as the shape of the problem rather than a pricing formula.

- **"Lost in the middle."** In practice, tokens at the start and end of the context get more attention than those in the middle. Put important information near the end of your context, close to the query.

- **Memory is the real limit.** The KV cache (stored Keys and Values from previous tokens) grows with context length. On a GPU, this memory is often what limits how long a context you can use — not the compute itself.

- **Position isn't built in.** Attention just compares vectors — it doesn't inherently know position. Position information gets added separately (via techniques called RoPE or ALiBi). This is why extending a model's context window requires specific engineering.

## Training (what you need to know)

Three stages, each building on the last:

1. **Pretraining** — the model learns to predict the next token on massive text (books, web, code). Costs hundreds of millions of dollars. Produces a "base model" that can complete text but doesn't follow instructions well.

2. **Supervised fine-tuning (SFT)** — train on curated instruction/response pairs. This teaches the model to be helpful and follow directions.

3. **Reinforcement Learning from Human Feedback (RLHF) / preference tuning** — train the model to prefer outputs that humans rate highly. This is where safety behavior, personality, and refusal patterns come from.

What this means for you:

- Model behavior comes from training. You can't easily override it from outside.

- Refusals and safety are trained preferences, not hard rules — they can be bypassed (hence prompt injection).

- Different providers (Anthropic, OpenAI, Google) make different training choices, which is why models feel different.

- You work with what the training gave you. Your job is to steer it, not reprogram it.

## Common jargon, translated

| What people say | What they mean |
|---|---|
| "It's just autocomplete" | True at the mechanism level. Dismissive about what emerges from it. |
| "The model is reasoning" | It's generating tokens that look like reasoning. Sometimes correct, sometimes confident nonsense. |
| "Hallucination" | Confidently wrong output. Inherent to generative models. Design around it. |
| "Parameters" | The model's size (number of learned weights). More = more capable, more expensive. |
| "Frontier model" | The current most-capable class. Changes every few months. |
| "Emergent capabilities" | Things the model does well that weren't explicitly trained. Real but overhyped. |

## Things to watch for

**"The model knows X."** It doesn't know anything. It has statistical patterns. It fails on obscure-but-real facts and succeeds on common-but-wrong ones. Never use it as a factual database without retrieval.

**Token ≠ word.** 1,000 tokens ≈ 750 English words ≈ 500 French words. Code tokenizes differently. Use the actual tokenizer when estimating costs. You can try [Huggingface's tokenizer for testing](https://huggingface.co/spaces/Xenova/the-tokenizer-playground).

**Non-determinism is the default.** Same prompt, different outputs (at temperature > 0). Even at temperature 0, you'll see minor variance in practice — not from the sampling itself (which is deterministic) but from tiny floating-point rounding differences in how GPUs parallelize the math. Design for it.

**Instruction-following is soft.** The model is biased toward following your instructions, not guaranteed to. 99% success rate = fails 1 in 100 times in production.

**Context ≠ memory.** Within one call, the model sees everything. Across calls, it remembers nothing. "Memory" in agents is a system you build around the model.

## Where the field actually is

We don't fully understand why LLMs do what they do. Research on interpretability (what's happening inside the model) is active but incomplete. What this means for you: you can't reason about LLM behavior the way you reason about deterministic code. You have to treat it empirically — test, measure, iterate.

This is uncomfortable for engineers used to reading source code to understand behavior. Get comfortable with it. The whole curriculum is built around this reality.

## Go deeper

- [Andrej Karpathy's Deep Dive into LLMs](https://www.youtube.com/watch?v=7xTGNNLPyMI) — 3 hours, excellent

- [The Illustrated Transformer](https://jalammar.github.io/illustrated-transformer/) by Jay Alammar — visual architecture walkthrough

- [Anthropic's Mapping the Mind of a Large Language Model](https://www.anthropic.com/research/mapping-mind-language-model) — interpretability research
