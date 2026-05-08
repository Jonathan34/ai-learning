# Team and Org Patterns

How you organize AI work determines how well it scales. The most common pattern (every product team rolls their own) creates chaos. The other common pattern (a central AI team owns everything) creates bottlenecks. Good organizations evolve toward something in between.

## Anti-patterns

**"AI Center of Excellence" that becomes a bottleneck.** A central team that every product team must go through to ship AI features. Starts well-intentioned. Becomes a queue. Product teams wait weeks for the AI team to build their feature. The AI team is overwhelmed and builds generic solutions that don't fit anyone's specific needs.

**Every team builds their own.** No shared infrastructure. Team A builds their own prompt versioning. Team B builds their own eval harness. Team C builds their own provider integration. All slightly different. All with different quality standards. Nobody learns from each other.

**Research team disconnected from product.** An ML research team publishes papers and builds demos. Product teams can't use any of it because it's not production-ready. The research team doesn't understand production constraints. The product team doesn't understand what's possible.

## Patterns that work

### Platform + embedded

The most common successful pattern:

- A **central platform team** builds shared infrastructure: model routing, observability, eval tooling, prompt registry, cost tracking, governance.

- **Product teams** build features on top of the platform. They own their prompts, their tools, their eval sets, their domain logic.

- Optionally, **embedded AI engineers** sit within product teams to bridge the gap — they know the platform and the product domain.

This is the same pattern that worked for DevOps (central platform, product teams use it) and for data engineering (central data platform, product teams build on it).

### Community of practice

Across all teams, a community of practice shares knowledge:

- Internal Slack channel for AI questions and wins

- Monthly demo day where teams show what they've built

- Shared eval harness templates and prompt patterns

- Paper reading group (optional, for those interested)

- Regular architecture reviews for new AI features (like security reviews, but for AI concerns)

This doesn't require a formal org structure. It just requires someone to organize it and leadership to support the time investment.

## Roles in an AI-enabled org

| Role | What they do | Notes |
|---|---|---|
| **AI/Applied AI Engineer** | Build AI features end-to-end: prompts, tools, eval, integration | The most common role. Software engineers who've learned AI. |
| **AI Platform Engineer** | Build shared infrastructure (routing, observability, eval tooling) | Similar to a DevOps/platform engineer but AI-focused. |
| **ML Engineer** | Train, fine-tune, and optimize models | Less common in applied AI teams (you're usually using pre-trained models). |
| **Prompt/Evals Engineer** | Specialize in prompt design and evaluation | Emerging role. Some orgs have dedicated people for this. |
| **AI Product Manager** | Define what to build, success metrics, user research for AI features | Needs to understand AI capabilities and limitations. |

For most applied AI teams (using pre-trained models, not training their own), you need AI engineers and platform engineers. You probably don't need ML engineers unless you're fine-tuning or training.

## Hiring

**What to screen for:**

- Has built something with LLMs (work, side project, or significant experimentation)

- Can reason about failure modes and trade-offs (not just "it works in the demo")

- Writes good prompts (give them a prompt engineering exercise in the interview)

- Thinks about evaluation naturally ("how would you know if this is working?")

- Learns fast in new domains (AI changes quickly; you need people who adapt)

**What to avoid:**

- People who only know the hype ("agents will replace all software engineers")

- People who can't write code (prompt engineering without engineering is fragile)

- People who've only done demos, never shipped to production

- People who are rigid about one framework or one provider

**A useful signal:** generalist software engineers who've gotten interested in AI and built things on their own often make better applied AI engineers than ML specialists who've never shipped a product. The engineering fundamentals (testing, debugging, production operations) transfer directly. The AI-specific knowledge can be learned.

## Career ladders

How do you evaluate and promote AI engineers? The same way you evaluate other engineers, with AI-specific additions:

- **Junior:** Can implement AI features given clear specifications. Writes prompts, builds basic eval sets, integrates with APIs.

- **Mid:** Can design AI features end-to-end. Chooses appropriate patterns (workflow vs agent), designs eval strategies, handles production concerns.

- **Senior:** Can architect AI systems across multiple features. Makes build-vs-buy decisions, designs platform components, mentors others, influences product direction.

- **Principal/Staff:** Sets technical direction for AI across the organization. Defines standards, evaluates new technologies, represents the org externally, makes decisions with long-term consequences.

The AI-specific dimension at senior+ levels: can you tell the organization where AI should and shouldn't be used? Can you predict which approaches will work before building them? Can you design systems that stay reliable as models and patterns evolve?

## Scaling the team

**Phase 1 (1-3 people):** One or two engineers building the first AI features. No platform needed. Just ship.

**Phase 2 (4-10 people):** Patterns emerge. Multiple features, multiple teams starting to use AI. Time to extract shared infrastructure. Designate someone to own the platform layer.

**Phase 3 (10-30 people):** Formal platform team. Community of practice. Architecture reviews for new AI features. Governance and cost tracking become necessary.

**Phase 4 (30+):** Embedded AI engineers in product teams. Specialized roles (evals, platform, security). Formal career ladder. Training programs for engineers new to AI.

## Things that trip people up

**Over-hiring "AI specialists" early.** For the first few AI features, you need good software engineers who can learn AI, not AI researchers who can't ship products. The engineering fundamentals matter more than the AI knowledge at this stage.

**Under-investing in platform.** "Just build features" works until team 4 or 5, then everything slows down because everyone is solving the same problems independently.

**No one owns quality.** If nobody is responsible for "are our AI features actually good?", quality drifts. Someone needs to own continuous evaluation across features.

**Letting the AI team become a service team.** If product teams throw requirements over the wall and the AI team builds everything, you get a bottleneck. Product teams should own their AI features; the platform team provides tools and guidance.

**Ignoring change management.** AI changes how engineers work. Some people are excited. Some are threatened. Some are skeptical. Acknowledge this. Provide training. Show concrete value. Don't mandate adoption without support.

## Where things stand

Most organizations are somewhere between Phase 1 and Phase 2. The patterns for Phase 3+ are emerging but not yet standardized. If you're building an AI team or org structure today, expect to iterate on it as the field matures.

The organizations doing best are the ones that treat AI as an engineering discipline (with rigor, testing, and production standards) rather than as magic (with hype, demos, and hope).

## Go deeper

- [Anthropic's team structure](https://www.anthropic.com/careers) — how a frontier AI company organizes (research + applied + policy)

- [Eugene Yan's "Patterns for Building LLM-based Systems"](https://eugeneyan.com/writing/llm-patterns/) — practical patterns from industry

- [Chapter 19](19-staying-current.md) covers staying current as the field evolves
