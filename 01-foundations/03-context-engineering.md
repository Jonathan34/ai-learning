---
layout: default

title: "03 — Context Engineering"
nav_order: 3
parent: "Foundations"
---
# 03 — Context Engineering

## The mental model

**Context engineering is the discipline of deciding what goes into the model's context window, why, and in what form.**

Prompts are one part of that. But in real systems, the context is dynamically assembled from:
- A system prompt (stable)
- Retrieved documents (from a knowledge base)
- Conversation history (from prior turns)
- Tool outputs (from function calls the agent made)
- User state (profile, permissions, session data)
- The current user input

The model sees all of this as one long sequence of tokens. Its response is a function of that entire sequence. Context engineering is about doing that assembly well.

This discipline has emerged as the more honest framing for what people used to call "prompt engineering" once they realized the prompt is usually the smallest part of the input.

## Why this matters

Most production AI systems fail because of bad context, not bad models. The model is given the wrong documents, too many documents, out-of-date state, or conflicting instructions — and produces a bad answer that gets blamed on "hallucination."

**The thing to remember:** *garbage in, garbage out* applies with compounding severity in LLMs. The model has no way to know which parts of its context are relevant, trustworthy, or current. You have to get that right before generation.

## The three jobs of context engineering

### 1. Deciding what to include

The context window is a finite budget. In a 200K-token window, you still have to decide what's worth spending tokens on. This is a classic relevance and ranking problem, dressed up with LLMs.

Typical trade-offs:
- More retrieved documents → better recall, more cost, more risk of "lost in the middle"
- More conversation history → better continuity, more cost, more risk of old instructions conflicting with current ones
- Tool outputs → essential when the agent just called a tool, clutter when stale
- Few-shot examples → helpful for novel tasks, wasteful for routine ones

**Senior move:** a context assembly strategy is a design artifact. Write it down. Diagram it. Review it the way you'd review a data flow diagram.

### 2. Formatting what's included

The same information can be presented many ways. The format affects both cost and performance.

- **Raw documents** — easy to produce, token-heavy, sometimes confusing to the model
- **Summaries** — cheap in tokens, risk losing the fact the user needs
- **Structured data (JSON, tables)** — good for machine-like information, can be awkward for narrative
- **Marked-up text with sections/headers** — often the best balance

Guidelines that hold up:
- Use consistent delimiters so the model can distinguish sections (`<document id="42">...</document>`)
- For retrieved documents, include source metadata (title, date, source) — this helps the model cite correctly
- For long context, put the most important content near the end (models attend better to recent tokens)

### 3. Keeping context coherent

The model has no mechanism to detect contradictions in its context. If retrieved document A says "policy X was retired in 2024" and document B says "policy X applies to all new users," the model may cite either, both, or synthesize a confused answer.

This is why retrieval quality matters so much: you're not just looking up relevant documents, you're trying to produce a coherent context that actually supports the answer.

## Retrieval-Augmented Generation (RAG), in one section

RAG is the dominant pattern for giving LLMs access to information they weren't trained on (internal documents, recent data, user-specific state). At its simplest:

1. Embed your documents into vectors (one vector per chunk)
2. Embed the user query into a vector
3. Find the top-K most similar document chunks (nearest neighbor search)
4. Inject those chunks into the prompt as context
5. Generate the response

This is "naive RAG." It works surprisingly well for easy cases and fails in predictable ways for hard ones.

**The hard parts of RAG:**

- **Chunking.** How you split documents determines what can be retrieved. Too-small chunks lose context; too-large chunks dilute relevance. Most teams end up with hybrid strategies (e.g., sentence-boundary chunks of 200-500 tokens, with 50-token overlaps and parent-document references).
- **Embedding model choice.** Different embedding models have different strengths. Use the one matched to your domain (general-purpose, code, multilingual). Expect to re-embed everything when you change models.
- **Query transformation.** User queries are often short and ambiguous. Rewriting them into better search queries (sometimes using an LLM) is a cheap improvement.
- **Re-ranking.** After nearest-neighbor retrieval, re-rank the top results with a more precise model (a cross-encoder or small LLM) to improve the order. Often the biggest single quality win.
- **Hybrid search.** Dense retrieval (vector similarity) plus sparse retrieval (BM25 or similar keyword search) together often beats either alone. Keywords catch exact-match cases; vectors catch semantic matches.
- **Citation.** If the user needs to verify an answer, you need to surface *which* documents supported it. Design this in from the start.

**Gotcha:** naive RAG feels magical in demos because the test query exactly matches one of the stored documents. Real users ask questions the documents don't directly answer — they require synthesis across documents, or the right document doesn't exist, or the question is ambiguous. Production RAG is mostly about handling those cases.

## Memory patterns for agents

An agent that runs across multiple turns needs memory. This doesn't exist in the model — it's a system you build around the model.

**Short-term memory (within a session):**
- Conversation history, truncated or summarized as it grows
- Current task state (what the agent is trying to do, what it's tried)
- Tool call results

**Long-term memory (across sessions):**
- User facts ("likes terse responses")
- Prior conversations (relevant ones, retrieved when needed)
- Learned preferences or corrections

Architecturally, long-term memory is usually a vector store or structured database, queried at the start of each session or each turn to produce context.

**Gotcha:** memory systems decay in quality over time. Stale facts, outdated preferences, contradictions between remembered and observed behavior. If memory is in your architecture, so is memory pruning, memory correction, and memory conflict resolution. Don't skip these.

## The "lost in the middle" problem

Empirical finding from research (Liu et al., 2023 and follow-ups): LLMs attend better to tokens at the beginning and end of a long context than those in the middle. If the critical information is buried in the middle of a 100K-token context, the model may miss it.

Practical consequences:
- In retrieved context, put the most likely-relevant documents at the end, not the beginning
- For multi-document synthesis, don't pile everything into context — re-rank and select
- If the user's question needs something from deep memory, put it close to the query

The effect varies by model. Newer long-context models are better, but not immune.

## When to use RAG vs fine-tuning vs agents

Common architecture decision, worth being crisp on:

- **Use RAG when** the knowledge is factual, changes over time, or is proprietary. You want freshness and citability.
- **Use fine-tuning when** you want to change the *style* of responses, or teach the model a structured task it's not good at by default. Fine-tuning doesn't add knowledge well; it changes behavior.
- **Use agents when** the task requires multiple steps, tool use, or decision-making that unfolds over time.

Most production AI systems are hybrids: a fine-tuned or instruction-tuned model, driven by agents, using RAG for knowledge.

## Gotchas

**Gotcha: "We'll just throw everything in the context"**

Tempting with large context windows. Terrible idea. Costs balloon, latency increases, "lost in the middle" kicks in, and you still have no guarantee the right information gets used. Context assembly is a design problem, not a dump.

**Gotcha: Embedding drift**

If you re-index documents with a new embedding model, old queries in logs will compare against new embeddings incorrectly. Either re-embed everything or version your index carefully.

**Gotcha: PII in context**

Retrieved documents, chat history, and tool outputs often contain user PII. That PII now flows to the model provider (if you're using a hosted model). Understand your data flow and comply with your data governance.

**Gotcha: RAG with no guardrails against refusing**

A well-built RAG system should be willing to say "I don't have information to answer that." A poorly-built one will synthesize an answer from whatever was retrieved, even if the retrieval was off-topic. Design the prompt to acknowledge insufficient context and test for it.

**Gotcha: Tight coupling to the embedding model**

Changing your embedding model is a multi-week project because you have to re-embed everything and usually re-tune retrieval parameters. Pick carefully.

## Honest status

Context engineering is less a discipline with settled patterns and more an active practice. The tooling is improving (vector databases, re-ranking models, hybrid search) but best practices are still shifting. The gap between "demo RAG" and "production RAG" is large and most teams underestimate it.

## What to read next

- **04 — Evaluation.** You can't improve what you don't measure, and this applies doubly to context engineering.
- **Workshop W2 — First RAG Pipeline.** Build one end-to-end.
- **"Retrieval-Augmented Generation for Large Language Models: A Survey"** (Gao et al., 2023). Academic survey, good taxonomy.
- **Anthropic's "Contextual Retrieval"** blog post. Practical recent work that materially improves RAG quality.
- **LangChain's RAG docs.** Concrete patterns, decent starting point even if you don't use LangChain.
