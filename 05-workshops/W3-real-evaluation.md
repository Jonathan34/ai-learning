---
title: "W3 — Real Evaluation"
nav_order: 3
parent: "Workshops"
---
# Workshop W3 — Real Evaluation

> Status: **outline**. Will expand in full depth.

**Goal:** build an evaluation harness for an LLM task, including an LLM-as-judge component and a regression test suite.

**Time:** 3-4 hours

**Planned contents:**

1. Pick a task: classification, extraction, summarization, or Q&A on your RAG system from W2
2. Design a golden test set (20-50 examples, diverse)
3. Write property-based tests (length, format, forbidden content)
4. Implement LLM-as-judge with calibration (use a different model than the one you're evaluating)
5. Run on three different models/prompts; compare
6. Build a regression harness: when the prompt or model changes, the suite runs and flags regressions
7. Explore failure modes of LLM-as-judge (bias toward verbosity, self-preference, etc.)
8. Optional: set up Braintrust, Promptfoo, or Langfuse and run the same evals through it

**Key gotchas demonstrated:**
- Self-evaluation failure (judging a model with itself)
- The difference between "looks right" and "is right"
- Drift in the eval set itself over time
- The hidden cost of good evals (compute, human review)
