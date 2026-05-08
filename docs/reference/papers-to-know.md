# Papers to Know

The ~20 papers that actually matter for a Principal Engineer in applied AI. Not exhaustive. Organized by category.

For each, the guidance is:

- **[Read]** — read the whole paper

- **[Skim]** — abstract, intro, conclusion, figures

- **[Know of]** — understand what the paper claims; don't need the details

## Foundations

- **"Attention Is All You Need"** (Vaswani et al., 2017) — the Transformer paper. **[Skim]**. You don't need the architecture details, you need to know this is the paper that started the current era.

- **"Language Models are Few-Shot Learners"** (Brown et al., 2020) — GPT-3 paper. **[Skim]**. Introduced few-shot prompting as a capability.

- **"Training language models to follow instructions with human feedback"** (Ouyang et al., 2022) — the InstructGPT / RLHF paper. **[Skim]**. How modern chat models get their instruction-following ability.

- **"Scaling Laws for Neural Language Models"** (Kaplan et al., 2020) — why bigger models are better, quantified. **[Know of]**.

- **"Training Compute-Optimal Large Language Models"** (Hoffmann et al., 2022) — the Chinchilla paper, which refined scaling laws and sparked the focus on training data. **[Know of]**.

## Prompting and reasoning

- **"Chain-of-Thought Prompting Elicits Reasoning in Large Language Models"** (Wei et al., 2022) — CoT. **[Read]**. Still foundational; production prompts still use CoT patterns.

- **"Self-Consistency Improves Chain of Thought Reasoning"** (Wang et al., 2022) — sample multiple CoTs, take majority. **[Know of]**.

- **"The Prompt Report: A Systematic Survey of Prompting Techniques"** (Schulhoff et al., 2024) — taxonomy and references. **[Skim as reference]**.

## RAG

- **"Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks"** (Lewis et al., 2020) — the original RAG paper. **[Skim]**.

- **"Lost in the Middle: How Language Models Use Long Contexts"** (Liu et al., 2023) — the canonical reference for the middle-position attention degradation. **[Read]**.

- **"Retrieval-Augmented Generation for Large Language Models: A Survey"** (Gao et al., 2023) — good taxonomy. **[Skim]**.

- **Anthropic's "Contextual Retrieval"** (blog post, 2024) — practical technique that materially improves RAG. **[Read]**.

## Agents

- **"ReAct: Synergizing Reasoning and Acting in Language Models"** (Yao et al., 2022) — the ReAct paper. **[Skim]**.

- **"Toolformer: Language Models Can Teach Themselves to Use Tools"** (Schick et al., 2023) — early tool-use. **[Know of]**.

- **Anthropic's "Building Effective Agents"** (blog post, 2024) — the pragmatic taxonomy of agent patterns. **[Read]**. Probably the single most useful contemporary piece on agent architecture.

- **"SWE-bench" and related papers** — benchmarks for coding agents. **[Know of]**.

## Safety and alignment

- **"Constitutional AI: Harmlessness from AI Feedback"** (Bai et al., 2022) — Anthropic's approach. **[Skim]**.

- **"Red Teaming Language Models with Language Models"** (Perez et al., 2022) — automated red-teaming. **[Know of]**.

- **"Universal and Transferable Adversarial Attacks on Aligned Language Models"** (Zou et al., 2023) — the "GCG" jailbreak paper. **[Know of]** — shows that alignment is not a solved problem.

- **"Prompt Injection: Parameterization of Fixed Inputs"** (Liu et al., 2023) — foundational work on prompt injection. **[Skim]**.

## Interpretability (for context)

- **Anthropic's "Scaling Monosemanticity"** (Templeton et al., 2024) — sparse autoencoders for interpretability. **[Know of]**.

- **Anthropic's "Mapping the Mind of a Large Language Model"** (blog post) — for a layperson's view of interpretability research. **[Read]**.

## Not-papers-but-should-read

- **Anthropic's "Engineering Challenges of Scaling Interpretability"** — how to read the field

- **Simon Willison's weblog** — ongoing practical commentary, including thoughtful coverage of prompt injection and LLM engineering

- **Ethan Mollick's "One Useful Thing"** newsletter — the best popular-level coverage of applied AI

- **Anthropic's courses on prompt engineering** — free, high quality, mostly applied

- **HuggingFace Agents Course** — practical, free

## On reading papers

You're not trying to understand the math. You're trying to understand:

1. What the paper claims

2. How they tested it

3. Whether the claim is as strong as the abstract suggests (often no)

4. What changes in practice if the claim holds

For most of these, that's achievable in 20-30 minutes per paper.

## A note on staleness

This list is current as of late 2025. In AI, papers age fast — results from 2 years ago are often obsolete. Prioritize recent surveys and blog posts from labs over individual papers when you want the current picture.
