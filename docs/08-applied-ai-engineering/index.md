# Applied AI Engineering

Applied AI work sits between product discovery, software engineering, and
customer architecture. The goal is not to demonstrate a model. It is to find a
valuable problem, build enough of the real system to test it, measure whether
it works, and help the customer run it in production.

This section is for a senior engineer or architect who already understands the
basics of LLM applications. It focuses on what real customer work exposes
quickly: how you think when the problem is unclear. The same practice can also
help with technical job interviews when needed.

| # | Chapter |
|---|---|
| 35 | [From Use Case to Engineering Plan](35-use-case-to-engineering-plan.md) |
| 36 | [Prototype, Evaluate, Launch, Learn](36-prototype-evaluate-launch.md) |
| 37 | [Applied Coding and System Design Practice](37-applied-engineering-exercises.md) |
| 38 | [Deep Dive: Production Support Agent](38-deep-dive-production-support-agent.md) |

## What you should be able to do

- Turn "we want AI" into a narrow problem with an owner and a measurable goal.
- Decide whether the answer is a model call, retrieval, a fixed workflow, or
  an agent.
- Build a small vertical slice with real interfaces and failure handling.
- Create an evaluation set before arguing about models.
- Compare quality, latency, cost, safety, and operating effort.
- Move from prototype to staged production without losing the evidence.
- Explain the result to an engineer, product leader, or executive.
- Turn repeated customer failures into useful product feedback.

## Suggested order

Read chapters 35 and 36 first. Work through four coding and four system design
problems from chapter 37. Then do the chapter 38 deep dive on a whiteboard
without notes. Finish with
[W18 — Applied AI Customer Pilot](../05-workshops/W18-applied-ai-customer-pilot.md)
so you have a small implementation and real measurements to discuss.

The most useful prerequisites are:

- [Evaluation](../01-foundations/04-evaluation.md)
- [Security and Safety](../01-foundations/05-security-and-safety.md)
- [Structured Output](../01-foundations/06-structured-output.md)
- [MCP and Tools](../02-agents/09-mcp-and-tools.md)
- [Deployment Patterns](../03-production/14-deployment-patterns.md)
- [AI Solution Architecture](../07-enterprise-architecture/32-ai-solution-architecture.md)
