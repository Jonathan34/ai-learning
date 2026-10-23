# AI Systems — Learning Path for Senior Engineers

A structured curriculum for experienced software engineers who want to become credible across AI software and the cloud infrastructure that runs it.

## Who this is for

You're a senior software engineer, architect, or engineering leader. You know distributed systems, cloud, operational excellence, and you've shipped production software for years. What you need is to become credible in the AI/agentic systems space — not "I read an article about transformers" credible, but "I can architect a production agent system and stand up in a design review with ML engineers" credible.

## Part I: Software engineering for AI

### Phase 1: Foundations (~60 min reading)

| # | Chapter |
|---|---|
| 01 | [How LLMs Actually Work](01-foundations/01-llm-mental-model.md) |
| 02 | [Prompting as Programming](01-foundations/02-prompting-as-programming.md) |
| 03 | [Context Engineering](01-foundations/03-context-engineering.md) |
| 04 | [Evaluation: The Hardest Unsolved Problem](01-foundations/04-evaluation.md) |
| 05 | [Security and Safety](01-foundations/05-security-and-safety.md) |
| 06 | [Structured Output and Type Safety](01-foundations/06-structured-output.md) |

### Phase 2: Agents (~50 min reading)

| # | Chapter |
|---|---|
| 07 | [What an Agent Actually Is](02-agents/07-what-is-an-agent.md) |
| 08 | [Agent Frameworks Landscape](02-agents/08-framework-landscape.md) |
| 09 | [MCP and Tool Interfaces](02-agents/09-mcp-and-tools.md) |
| 10 | [Multi-Agent Patterns](02-agents/10-multi-agent-patterns.md) |
| 11 | [State, Memory, and Long-Running Agents](02-agents/11-state-memory-durability.md) |

### Phase 3: Production (~50 min reading)

| # | Chapter |
|---|---|
| 12 | [Inference Economics](03-production/12-inference-economics.md) |
| 13 | [Observability for AI](03-production/13-observability.md) |
| 14 | [Deployment Patterns](03-production/14-deployment-patterns.md) |
| 15 | [Local and Edge Inference](03-production/15-local-and-edge.md) |
| 16 | [AI Platform Engineering](03-production/16-ai-platform-engineering.md) |

### Phase 4: Leadership (~40 min reading)

| # | Chapter |
|---|---|
| 17 | [Deciding What to Build with AI](04-leadership/17-deciding-what-to-build.md) |
| 18 | [AI Product Sense](04-leadership/18-ai-product-sense.md) |
| 19 | [Team and Org Patterns](04-leadership/19-team-and-org-patterns.md) |
| 20 | [Staying Current](04-leadership/20-staying-current.md) |

## Part II: Cloud infrastructure for AI

### Phase 5: AI workloads

| # | Chapter |
|---|---|
| 21 | [Distributed Training Systems](05-ai-workloads/21-distributed-training.md) |
| 22 | [Self-Hosted Inference](05-ai-workloads/22-self-hosted-inference.md) |
| 23 | [Workload Sizing and Economics](05-ai-workloads/23-workload-sizing.md) |

### Phase 6: AI infrastructure

| # | Chapter |
|---|---|
| 24 | [GPU Systems](06-ai-infrastructure/24-gpu-systems.md) |
| 25 | [Cluster Networking](06-ai-infrastructure/25-cluster-networking.md) |
| 26 | [Storage and the Data Path](06-ai-infrastructure/26-storage-and-data.md) |
| 27 | [Kubernetes and Slurm](06-ai-infrastructure/27-kubernetes-and-slurm.md) |
| 28 | [Infrastructure as Code](06-ai-infrastructure/28-infrastructure-as-code.md) |
| 29 | [Reliability and Performance](06-ai-infrastructure/29-reliability-and-performance.md) |

### Phase 7: Enterprise architecture

| # | Chapter |
|---|---|
| 30 | [Secure AI Platforms](07-enterprise-architecture/30-secure-ai-platforms.md) |
| 31 | [From Proof of Concept to Production](07-enterprise-architecture/31-poc-to-production.md) |

## Workshops (hands-on)

| # | Workshop |
|---|---|
| W1 | [Local LLM Setup](05-workshops/W1-local-llm-setup.md) |
| W2 | [First RAG Pipeline](05-workshops/W2-rag-pipeline.md) |
| W3 | [Real Evaluation](05-workshops/W3-real-evaluation.md) |
| W4 | [Tool-Using Agent](05-workshops/W4-tool-using-agent.md) |
| W5 | [Multi-Agent System](05-workshops/W5-multi-agent-system.md) |
| W6 | [Observability Setup](05-workshops/W6-observability-setup.md) |
| W7 | [Red-Team Exercise](05-workshops/W7-red-team-exercise.md) |
| W8 | [Cost Optimization](05-workshops/W8-cost-optimization.md) |
| W9 | [Agent Efficiency & KPIs](05-workshops/W9-agent-efficiency-kpis.md) |
| W10 | [Structured Output in Practice](05-workshops/W10-structured-output-in-practice.md) |
| W11 | [Python for AI Infrastructure](05-workshops/W11-python-ai-infrastructure.md) |
| W12 | [GPU Cluster Acceptance Test](05-workshops/W12-gpu-cluster-acceptance.md) |
| W13 | [Workload Sizing and TCO](05-workshops/W13-workload-sizing-tco.md) |
| W14 | [Enterprise AI Platform Capstone](05-workshops/W14-enterprise-ai-platform.md) |

### Reference

| Document | Description |
|---|---|
| [Glossary](reference/glossary.md) | Terms a PE needs in their vocabulary |
| [Papers to Know](reference/papers-to-know.md) | The ~20 papers that actually matter |
| [Tools and Models](reference/tools-and-models.md) | Quick reference (changes fast) |
| [Gotchas](reference/gotchas-consolidated.md) | Consolidated failure modes across all chapters |
| [AI Infrastructure Videos](reference/ai-infrastructure-videos.md) | Guided videos for workloads, GPU platforms, and architecture |

---

## Suggested paths

**AI software sprint (2 weekends, ~20 hours):** Read Phase 1 and 2. Do W1, W2, and W4.

**AI infrastructure bridge (4-6 weeks, part-time):** Read Phase 5 through 7. Do W11 through W14.

**Full stack (8-12 weeks, part-time):** Complete both parts and the core workshops. Ship one production-grade project.
