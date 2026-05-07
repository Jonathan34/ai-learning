---
layout: default
title: "03 — Context Engineering"
nav_order: 3
parent: "Foundations"
---

# Context Engineering

Prompts get all the attention, but in real systems the prompt is usually the smallest part of what the model sees. The bigger challenge is assembling the right context — the full input the model receives on each call.

A typical production call includes:
- A system prompt (stable, versioned)
- Retrieved documents (from a knowledge base)
- Conversation history (from prior turns)
- Tool outputs (from function calls the agent made)
- User state (profile, permissions, session data)
- The current user input

The model sees all of this as one long sequence of tokens. Its response is a function of that entire sequence. Context engineering is about doing that assembly well — deciding what goes in, in what form, and in what order.

This is the more honest framing for what people used to call "prompt engineering" once they realized the prompt is usually 5% of the input.

## Why this is where most systems fail

Most production AI failures are context failures, not model failures. The model gets the wrong documents, too many documents, stale state, or conflicting instructions — and produces a bad answer that gets blamed on "hallucination."

Garbage in, garbage out applies with compounding severity here. The model has no way to know which parts of its context are relevant, trustworthy, or current. That's your job.

## The three jobs

### 1. Deciding what to include

The context window is a finite budget. Even in a 200K-token window, you have to decide what's worth spending tokens on.

Typical trade-offs:
- More retrieved documents → better recall, more cost, more risk of the model getting lost
- More conversation history → better continuity, more cost, more risk of old instructions conflicting with current ones
- Tool outputs → essential when fresh, clutter when stale
- Few-shot examples → helpful for novel tasks, wasteful for routine ones

A context assembly strategy is a design artifact. Write it down. Diagram it. Review it the way you'd review a data flow diagram. Most teams don't do this and end up with ad-hoc context that works until it doesn't.

### 2. Formatting what's included

Same information, different formats, different results.

What tends to work:
- Consistent delimiters so the model can distinguish sections (`<document id="42">...</document>`)
- Source metadata on retrieved documents (title, date, source) — helps the model cite correctly
- For long context, put the most important content near the end — models attend better to recent tokens
- Structured data (JSON, tables) for machine-like information; prose for narrative

What doesn't work:
- Dumping raw documents without any structure
- Mixing instructions and data without clear boundaries
- Putting critical information in the middle of a 100K-token context and hoping the model finds it

### 3. Keeping context coherent

The model can't detect contradictions. If document A says "policy X was retired in 2024" and document B says "policy X applies to all new users," the model may cite either, both, or synthesize something confused.

This is why retrieval quality matters so much. You're not just finding relevant documents — you're trying to produce a coherent context that actually supports a correct answer.

## RAG in one section

Retrieval-Augmented Generation is the dominant pattern for giving LLMs access to information they weren't trained on. The basic loop:

1. Embed your documents into vectors (one vector per chunk)
2. Embed the user query into a vector
3. Find the top-K most similar chunks (nearest neighbor search)
4. Inject those chunks into the prompt as context
5. Generate the response

This is "naive RAG." It works surprisingly well for easy cases and fails predictably for hard ones.

The hard parts, in order of how much they'll bite you:

**Chunking.** How you split documents determines what can be retrieved. Too small and chunks lose context. Too large and they dilute relevance. Most teams end up with sentence-boundary chunks of 200-500 tokens with overlaps and parent-document references. There's no universal right answer — it depends on your documents.

**Query transformation.** User queries are often short and ambiguous. Rewriting them into better search queries (sometimes using an LLM) is a cheap improvement that most teams skip.

**Re-ranking.** After nearest-neighbor retrieval, re-rank the top results with a cross-encoder or small LLM. Often the single biggest quality win you can get. The initial retrieval casts a wide net; the re-ranker picks the best fish.

**Hybrid search.** Dense retrieval (vectors) plus sparse retrieval (BM25 keyword search) together usually beats either alone. Keywords catch exact matches; vectors catch semantic matches. Use both.

**Citation.** If users need to verify answers, you need to surface which documents supported them. Design this in from the start — retrofitting citation is painful.

The gap between "demo RAG" and "production RAG" is large. Demo RAG works because the test query exactly matches a stored document. Real users ask questions the documents don't directly answer, or the right document doesn't exist, or the question is ambiguous. Production RAG is mostly about handling those cases gracefully.

## Memory patterns for agents

An agent running across multiple turns needs memory. The model itself is stateless — memory is a system you build around it.

**Short-term (within a session):** conversation history, current task state, recent tool results. Usually just kept in the context, truncated or summarized as it grows.

**Long-term (across sessions):** user facts, prior conversations, learned preferences. Usually a vector store or structured database, queried at the start of each session to produce relevant context.

The gotcha with memory: it decays. Stale facts, outdated preferences, contradictions between what was remembered and what's currently true. If memory is in your architecture, so is memory pruning, correction, and conflict resolution. Most teams skip these and pay for it later.

## The "lost in the middle" problem

Research finding (Liu et al., 2023): LLMs attend better to tokens at the beginning and end of long contexts than those in the middle. If critical information is buried in the middle of a 100K-token context, the model may miss it.

Practical consequences:
- Put the most likely-relevant retrieved documents near the end, close to the query
- For multi-document synthesis, don't pile everything in — re-rank and select
- The effect varies by model; newer long-context models are better but not immune

## When to use RAG vs fine-tuning vs agents

Quick decision framework:

- **RAG** when the knowledge is factual, changes over time, or is proprietary. You want freshness and citability.
- **Fine-tuning** when you want to change the style or behavior of responses, or teach a structured task. Fine-tuning doesn't add knowledge well; it changes how the model acts.
- **Agents** when the task requires multiple steps, tool use, or decisions that unfold over time.

Most production systems are hybrids: an instruction-tuned model, driven by agents, using RAG for knowledge.

## Things that trip people up

**"We'll just throw everything in the context."** Tempting with large context windows. Terrible idea. Costs balloon, latency increases, "lost in the middle" kicks in, and you still have no guarantee the right information gets used.

**Embedding drift.** If you re-index documents with a new embedding model, old queries in logs will compare against new embeddings incorrectly. Either re-embed everything or version your index carefully.

**PII in context.** Retrieved documents, chat history, and tool outputs often contain user PII. That PII now flows to the model provider. Know your data flow and comply with your governance requirements.

**RAG without a "say I don't know" path.** A system that always answers will hallucinate when retrieval fails. Design the prompt to acknowledge insufficient context and test for it explicitly.

**Tight coupling to the embedding model.** Changing your embedding model is a multi-week project because you have to re-embed everything and re-tune retrieval parameters. Pick carefully upfront.

## Where things stand

Context engineering is less a settled discipline and more an active practice. The tooling is improving (vector databases, re-ranking models, hybrid search) but best practices are still shifting. The gap between "demo RAG" and "production RAG" is where most of the real engineering lives.

If you take one thing from this chapter: the model is only as good as the context you give it. Most "the AI is wrong" complaints are actually "the context assembly is wrong" complaints. Fix the context first.

## Where to go deeper

- **Workshop W2 — First RAG Pipeline.** Build one end-to-end. Nothing teaches this like doing it.
- **Anthropic's "Contextual Retrieval" blog post.** Practical technique that materially improves RAG quality.
- **"Lost in the Middle" paper** (Liu et al., 2023). The canonical reference for attention degradation in long contexts.
- **LangChain's RAG docs.** Concrete patterns, decent starting point even if you don't use LangChain.
- Next chapter: **Evaluation** — you can't improve context quality without measuring it.

---

[← Previous](02-prompting-as-programming.html){: .mr-4 }[Next: Evaluation →](04-evaluation.html)
