---
title: "W2 — First RAG Pipeline"
nav_order: 2
parent: "Workshops"
---
# Workshop W2 — First RAG Pipeline

> Status: **outline**. Will expand in full depth.

**Goal:** build a retrieval-augmented generation pipeline end-to-end, understand each component well enough to have opinions about it.

**Time:** 3-4 hours

**Planned contents:**

1. Pick a corpus (your own docs, Wikipedia snapshot, or a public dataset)
2. Chunking strategies (fixed-size, sentence-boundary, semantic) — build two, compare
3. Embedding with an open model (sentence-transformers) and a hosted one (OpenAI or Cohere), compare
4. Set up a vector store (Chroma or Qdrant locally; not Pinecone for this workshop)
5. Build the basic retrieval → prompt → generation loop
6. Add a re-ranker (cross-encoder) and measure the quality change
7. Add hybrid search (BM25 + dense) and measure
8. Build 10 test queries with expected-correct sources; measure precision@K
9. Try a "lost in the middle" test (inject correct answer in a 50K-token context at different positions)
10. Reflect on what you'd change for production

**Key gotchas demonstrated by the workshop:**
- Why chunk size matters
- How re-ranking changes the distribution of results
- Why the same query can retrieve wildly different docs with different embedding models
- The gap between "retrieval works" and "answers are correct"
