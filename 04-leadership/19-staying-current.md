---
layout: default
title: "19 — Staying Current"
nav_order: 4
parent: "Leadership"
---

# Staying Current

The field moves fast. Most of what you read will be noise. The skill isn't "read everything" — it's "filter effectively and go deep on what matters."

## What to follow

**Lab blogs (high signal).** When Anthropic, OpenAI, Google DeepMind, or Meta AI publish a blog post, it's usually worth reading. These are the people building the models. Their posts are often more actionable than the papers they're based on.

**A small set of individuals (curated).** Find 5-10 people who do real work (not just commentary) and follow them. Good signals: they share what they've built, they're honest about what doesn't work, they cite sources. Bad signals: they post "10 AI tools that will change your life" threads.

**A few newsletters:**
- *Import AI* by Jack Clark — weekly, research-focused, well-curated
- *The Batch* by Andrew Ng — weekly, broader, good for staying aware
- *Ahead of AI* by Sebastian Raschka — monthly, deeper technical content
- *Simon Willison's blog* — ongoing, practical, especially good on tools and security

**Conference proceedings (skim, don't attend).** NeurIPS, ICML, ICLR, ACL. Skim the accepted paper lists for titles that seem relevant. Read 2-3 papers per conference. You don't need to attend.

## What to skip

- LinkedIn "AI agents will take your job" content
- Most Twitter/X threads with screenshots of prompts
- "Top 10 AI tools of 2026" listicles
- Vendor blog posts as primary sources (they're marketing)
- AI-generated AI commentary (yes, this exists and it's circular)
- Benchmark drama ("Model X beats Model Y on MMLU by 0.3%!")

## How to read papers

You don't need to understand the math. You need to understand:
1. What does the paper claim?
2. How did they test it?
3. Is the claim as strong as the abstract suggests? (Often no.)
4. What changes in practice if the claim holds?

For most papers, reading the abstract, introduction, and conclusion gets you 80% of the value. Go deeper only if the paper is directly relevant to something you're building.

## A sustainable routine

**2 hours per week.** That's enough to stay current without drowning. Split it:
- 30 min: skim newsletters and lab blogs from the week
- 30 min: read one paper or long blog post in depth
- 30 min: try something hands-on (new tool, new model, new technique)
- 30 min: write down what you learned (even just notes to yourself)

**A personal knowledge base.** Keep a simple document (markdown file, Notion page, whatever) where you note things you've learned. Date them. When you need to reference something later, you'll have it.

**Teach one thing a month.** Write an internal doc, give a short talk, explain something to a colleague. Teaching forces you to structure your understanding. It's the best test of whether you actually know something.

## Filtering for signal

The hardest part is distinguishing "genuinely important" from "temporarily exciting."

**Genuinely important (changes how you build):**
- New model capabilities that enable new product patterns
- New attack classes that change your security posture
- New evaluation techniques that improve your quality process
- Price drops that make previously-uneconomical features viable
- New standards or protocols that affect interoperability (like MCP)

**Temporarily exciting (interesting but not actionable yet):**
- New SOTA on a benchmark (rarely translates to production improvement)
- New framework launches (wait 6 months to see if it survives)
- "AGI is coming" discourse (not actionable regardless of truth)
- Demos of capabilities that aren't available via API yet

A useful heuristic: if something is genuinely important, you'll hear about it from multiple independent sources within a week. If you only see it in one thread, it's probably noise.

## Building a network

The fastest way to learn is from people doing the work. A few ways to build connections:

- **Internal community of practice.** If your org has one, participate actively. If it doesn't, start one.
- **Open source contributions.** Even small ones (documentation, bug reports, small features) connect you to the people building the tools.
- **Conference hallway track.** If you attend AI conferences, the conversations between talks are often more valuable than the talks.
- **Writing publicly.** Blog posts, technical articles, even thoughtful comments on others' posts. This attracts people working on similar problems.

You don't need a large network. 5-10 people you can message with questions or share findings with is enough.

## Things that trip people up

**Firehose anxiety.** You can't keep up with everything. Nobody can. The field produces more content in a week than anyone can read in a month. Accept this. Filter aggressively. Miss things. It's fine.

**Confusing novelty with importance.** New ≠ useful. Most new papers, tools, and techniques won't matter in 6 months. The ones that do will still be around in 6 months — you can catch up then.

**Learning in breadth without depth.** Knowing a little about everything is less useful than knowing a lot about the things relevant to your work. Go deep on 2-3 topics rather than shallow on 20.

**Hype cycles.** Every few months, something gets hyped (agents, RAG, fine-tuning, multi-modal, etc.). The hype is always ahead of the reality. Wait for the "trough of disillusionment" — that's when the practical patterns emerge.

**Not building.** Reading about AI without building with it is like reading about swimming without getting in the water. The workshops in this curriculum exist for this reason. Keep building things, even small ones.

## Where things stand

The field is moving fast but not uniformly. Some areas are maturing (RAG, tool use, basic agents). Others are still research (long-horizon autonomous agents, reliable multi-modal reasoning, interpretability). Knowing which is which helps you invest your learning time wisely.

The meta-skill is learning how to learn in a fast-moving field. If you can do that — filter signal from noise, go deep when it matters, stay hands-on, teach others — you'll stay current without burning out.

## Go deeper

- [Papers With Code](https://paperswithcode.com/) — papers with implementations, good for finding what's real
- [Anthropic research page](https://www.anthropic.com/research) — high-quality, relevant to applied work
- [Simon Willison's blog](https://simonwillison.net/) — ongoing practical commentary
- [HuggingFace Daily Papers](https://huggingface.co/papers) — curated daily paper picks

---

[← Previous](18-team-and-org-patterns.html){: .mr-4 } [Back to Home →](../index.html)
