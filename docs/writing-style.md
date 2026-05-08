# Writing Style Guide

Reference for all content in this curriculum. Use this to maintain consistency across chapters and to course-correct when the writing drifts toward AI-generated patterns.

## Audience

A senior software engineer (5-15+ years) who has never worked in ML or AI. They know distributed systems, APIs, databases, CI/CD, and production operations. They don't know what a tensor is, what RLHF stands for, or why "attention" is a technical term.

Write for someone smart who is new to this specific domain.

## Voice

- Write like an experienced engineer explaining something to a peer over coffee — not like a textbook, not like a blog post optimized for SEO, not like a curriculum document.

- First person is fine. "I've seen teams..." or "Here's how I think about it..." when it adds credibility.

- Be direct. Say what you mean in the fewest words that are still clear.

- Be opinionated where you have a basis for it. "This usually doesn't work" is more useful than "results may vary."

- Be honest about uncertainty. "Nobody knows yet" is a valid statement.

## Language rules

### Never drop unexplained jargon

Every technical term gets explained the first time it appears. Not in a footnote — inline, immediately.

Bad: "The model uses multi-head attention with RoPE positional encodings."

Good: "The model uses multi-head attention — running the attention process multiple times in parallel, each copy learning to focus on different things. Position information is added separately using a technique called RoPE."

If a term is important enough to use, it's important enough to explain. If it's not important enough to explain, don't use it.

### Prefer plain words

| Instead of | Write |
|---|---|
| utilize | use |
| facilitate | help / enable |
| leverage | use |
| implement | build |
| demonstrate | show |
| subsequently | then / after that |
| fundamentally | (usually delete) |
| essentially | (usually delete) |
| comprehensive | full / complete |
| robust | solid / reliable |

### Short sentences

If a sentence has more than one comma or a semicolon, consider splitting it. If it has a parenthetical inside a parenthetical, definitely split it.

Bad: "The attention mechanism, which computes a weighted sum over all tokens in the context (using the Query-Key dot product to determine weights), is the core operation that enables transformers to handle long-range dependencies."

Good: "The attention mechanism computes a weighted sum over all tokens in the context. It uses a dot product between Query and Key vectors to figure out which tokens matter most. This is what lets transformers handle connections across long distances."

### No filler phrases

Delete these on sight:

- "It's worth noting that..."

- "It's important to understand that..."

- "In order to..."

- "At the end of the day..."

- "The key takeaway here is..."

- "Let's dive into..."

- "As we discussed earlier..."

Just say the thing.

## Structure rules

### No rigid template across chapters

Chapters should NOT all follow the same structure. Vary based on what the content needs:

- Some chapters open with a concept, then examples

- Some open with a problem, then the solution

- Some are mostly a survey (frameworks chapter)

- Some are mostly practical (workshops)

The reader should not be able to predict the section headers of the next chapter based on the current one.

### Sections vary in length

Some sections are two sentences. Some are a full page. Match the length to the content, not to a template.

### Use diagrams when they help

Mermaid diagrams for:

- Flows and sequences (generation loop, RAG pipeline, agent loop)

- Architecture (layer stacks, system components)

- Decision trees (when to use X vs Y)

Don't use diagrams for things that are clearer as prose or tables.

### Tables for comparisons and quick reference

Use tables when comparing options, translating jargon, or providing quick-reference definitions. Don't use tables for narrative content.

### Code blocks for concrete examples

Show real code or prompt examples when they clarify. Keep them short — 5-15 lines. If longer, explain what to focus on.

## Tone rules

### No AI-generated patterns

Avoid these tells:

- Bolded thesis statements at the start of sections ("**The key insight is:**")

- "Senior move:" or "Pro tip:" callouts (just say it inline)

- Three-item parallel lists with identical sentence structure

- Rhetorical questions immediately answered ("Why does this matter? Because...")

- Every section ending with a neat summary sentence

- "Let's explore..." / "Let's dive into..."

- Overly clean transitions between sections

### Be comfortable with incomplete thoughts

Not everything needs a conclusion. Sometimes "this is still messy and nobody has a great answer" is the right ending for a section.

### Honest hedging over false confidence

- "This usually works" not "This is the best approach"

- "In my experience" not "It is well established that"

- "Nobody knows yet" not "Further research is needed"

- "This might not apply to your case" not "Results may vary"

### No performative humility or enthusiasm

- Don't say "I'm excited to share..."

- Don't say "This is just my humble opinion..."

- Don't say "This is a game-changer!"

- Just state things plainly.

## Navigation

Every chapter ends with:

1. A "Go deeper" section with 3-5 links (papers, docs, videos)

2. A horizontal rule

3. Previous/Next navigation links

Format:

```markdown

```

## Front matter

Every page needs:

```yaml
---
layout: default
title: "Chapter title"
nav_order: N
parent: "Section name"
---

```

## File naming

- Lowercase with hyphens: `01-llm-mental-model.md`

- Numbered for ordering within sections

- Workshops prefixed with W: `W1-local-llm-setup.md`
