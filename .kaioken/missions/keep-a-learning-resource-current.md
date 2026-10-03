## Goal
Keep this learning resource for senior software engineers current with new AI and LLM trends, facts, corrections, and references while preserving its human tone. Each weekly pass should make a few focused improvements, including useful videos and courses about transformer architecture and other LLM concepts. Create a research work item for System 1 (JEV) and System 2 (LLM), then add material only after JEV is identified precisely and the comparison can be framed accurately and with appropriate qualifications.

## May change
- `docs/**/*.md`: correct facts, clarify explanations, refresh stale material, add qualified coverage of useful developments, improve cross-links, and add or replace references, papers, videos, and courses.
- `docs/01-foundations/01-llm-mental-model.md` and related foundation pages: strengthen transformer architecture and LLM concept explanations.
- `docs/04-leadership/20-staying-current.md` and `docs/reference/*.md`: improve guidance for staying current and maintain useful source, paper, glossary, tool, and model references.
- `README.md` and `docs/index.md`: keep summaries and curriculum maps aligned with substantive curriculum changes.
- `mkdocs.yml`: update navigation only when an approved document is added, moved, or renamed.
- Add a new Markdown page inside the existing `docs/` structure when a topic cannot be explained cleanly in an existing page. Prefer focused integration over expanding the curriculum by default.
- Open focused issues and pull requests that document research questions, evidence, and proposed curriculum improvements.

## Leaves alone
- `.github/workflows/pages.yml`, `.gitignore`, `.markdownlint-cli2.jsonc`, and `requirements.txt` unless the operator separately expands the mission beyond content maintenance.
- Build, deployment, dependency, and repository-administration behavior.
- Existing chapter numbering, information architecture, and navigation structure unless a content change strictly requires a small corresponding update.
- The author's personal perspective and first-person experience; factual corrections may qualify a claim but must not manufacture or replace personal experience.
- Broad rewrites, stylistic churn, SEO language, generic AI-generated filler, leaderboard trivia, and changes made only to appear active.
- Claims about JEV, System 1, or a JEV-versus-LLM comparison until the intended JEV concept and authoritative sources are established.
- Broken, unverifiable, promotional, or weakly relevant references. Never invent citations, credentials, dates, findings, quotations, or resource descriptions.
- Merging changes, publishing announcements outside repository issues and pull requests, or exceeding configured budgets without explicit approval.

## Style
Follow `docs/writing-style.md`. Write like an experienced engineer explaining the subject to a capable peer over coffee. Assume software-engineering competence but no prior ML vocabulary. Be direct, concise, practical, and opinionated only when evidence supports it. Explain jargon inline on first use, prefer plain words and short sentences, vary chapter structure naturally, and use diagrams or tables only when they make the idea easier to understand. Preserve the repository's honest distinction between established knowledge, working practice, emerging evidence, and open questions.

Material factual claims, dates, benchmark numbers, named techniques, and recommendations must link to a relevant source. Prefer original papers, official documentation, standards, and the original page for a talk, video, or course. Use secondary explanations when they are unusually helpful for learning, not as substitutes for available primary evidence. Qualify contested or early findings and include dates or versions when a claim can become stale. For videos and courses, identify the author or provider, link to the canonical page, explain briefly why the resource is useful, and clearly label paid access. Keep Markdown compatible with the repository's lint rules.

## Sources to watch

Treat this as a source menu, not a requirement to scan every link every week. For a normal weekly pass, consult the relevant first-party sources, one or two research discovery feeds, and one or two independent curators. A secondary source may identify a useful development, but confirm material claims against official documentation, an original paper, a standard, or a project release before changing the curriculum.

### First-party releases and research

- [OpenAI API changelog](https://developers.openai.com/api/docs/changelog) and [developer blog](https://developers.openai.com/blog/) — model, API, agent, eval, and platform changes.
- [Anthropic release notes](https://platform.claude.com/docs/en/release-notes/overview), [news](https://www.anthropic.com/news), and [research](https://www.anthropic.com/research) — Claude API and model releases, system cards, interpretability, alignment, and agent research.
- [Gemini API release notes](https://ai.google.dev/gemini-api/docs/changelog) and [Google DeepMind](https://deepmind.google/) — Gemini and Gemma releases plus broader model research.
- [Meta AI](https://ai.meta.com/) and [Meta AI research](https://ai.meta.com/research/) — open-model releases and architecture or training research.
- [Mistral news](https://mistral.ai/news/) — European frontier and open-weight model releases.
- [Amazon Bedrock document history](https://docs.aws.amazon.com/bedrock/latest/userguide/bedrock-ug-doc-history.html) — model availability and production-platform feature changes; the page offers an RSS feed.
- [Model Context Protocol](https://modelcontextprotocol.io/specification/latest) — current specification, security guidance, SDKs, and protocol changelogs.

### Papers and open-model discovery

- [Hugging Face Daily Papers](https://huggingface.co/papers) — curated paper discovery; follow through to the original paper.
- [Hugging Face trending models](https://huggingface.co/models?sort=trending) — useful for spotting open-model adoption, not proof of model quality.
- [arXiv cs.CL](https://arxiv.org/list/cs.CL/recent), [cs.AI](https://arxiv.org/list/cs.AI/recent), and [cs.LG](https://arxiv.org/list/cs.LG/recent) — recent language, AI, and machine-learning preprints.
- [OpenReview](https://openreview.net/) — submissions, reviews, revisions, and decisions for major research venues.
- [ACL Anthology](https://aclanthology.org/) — canonical computational linguistics and NLP proceedings.
- [NeurIPS proceedings](https://papers.nips.cc/) — accepted papers from NeurIPS.

### Independent technical curators

- [Simon Willison's weblog](https://simonwillison.net/) — practical LLM releases, tools, security, and hands-on experiments.
- [Import AI](https://importai.substack.com/) by Jack Clark — weekly research and policy synthesis.
- [The Batch](https://www.deeplearning.ai/the-batch/) by DeepLearning.AI — broad weekly awareness and accessible research summaries.
- [Ahead of AI](https://magazine.sebastianraschka.com/) by Sebastian Raschka — deeper model architecture, training, and research analysis.
- [Interconnects](https://www.interconnects.ai/) by Nathan Lambert — open models, post-training, and frontier-model analysis.
- [Latent Space](https://www.latent.space/) — AI engineering, agents, infrastructure, and technical interviews.
- [Chip Huyen's blog](https://huyenchip.com/blog/) — production AI and ML systems.
- [Hamel Husain's blog](https://hamel.dev/) — applied AI engineering, evaluation, and debugging.

### Evaluation, security, and risk

- [Artificial Analysis](https://artificialanalysis.ai/) and [Arena](https://arena.ai/) — useful comparative signals for capability, cost, latency, and human preference. Never treat a leaderboard as sufficient evidence for a recommendation.
- [METR research](https://metr.org/blog/) and the [UK AI Security Institute](https://www.aisi.gov.uk/research) — agent capability, autonomy, and frontier-risk evaluations.
- [OWASP GenAI Security Project](https://genai.owasp.org/) — application and agent security risks and mitigations.
- [NIST AI Resource Center](https://airc.nist.gov/) — AI risk-management, measurement, evaluation, and governance guidance.

Record the date of each research pass. Prefer developments corroborated by multiple independent sources, but do not count several articles repeating the same announcement as independent confirmation. Treat vendor benchmark claims as leads to investigate, not settled facts.

## Done when
Each weekly pass delivers a small, coherent set of focused improvements, typically two to four substantive changes rather than a broad rewrite. Every changed claim is accurate and appropriately qualified, references resolve to the material described, new resources add real learning value for senior engineers, and surrounding navigation or cross-links remain consistent. The resulting prose sounds like the existing human author and passes the repository's Markdown checks. For the JET request, the first acceptable pass creates a clearly scoped research work item; curriculum text follows only when the exact concept, evidence, limits, and relationship to System 1/System 2 terminology are clear.

## Decisions

- What exact paper, project, model, or expansion do you mean by JEV in System 1 (JEV) and System 2 (LLM)?: Treat JEV as unresolved: create a research work item first and add no curriculum claims until authoritative sources identify the intended concept.
- Where should the requested JEV research work item live?: Open a GitHub issue first
- How should substantial new concepts normally enter the curriculum?: Integrate into existing chapters first
- May recommended videos and courses require payment?: Free-first; clearly label paid resources
