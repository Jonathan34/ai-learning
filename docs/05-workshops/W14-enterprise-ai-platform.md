# Workshop W14 — Enterprise AI Platform Capstone

**Goal:** Design and defend a production AI platform from workload to operating model.

**Time:** One weekend

**Prerequisites:** Part II and Workshops W11-W13

---

## Scenario

A regulated enterprise wants one platform for:

- Fine-tuning open models on private data
- Serving two online models
- Running nightly batch inference
- Giving researchers notebook access

The first production service must launch in 90 days.

Constraints:

- Data must stay in one approved region.
- Users authenticate through the company identity provider.
- GPU workers cannot have public IP addresses.
- The platform team knows Kubernetes but not Slurm.
- Research wants `sbatch`.
- Traffic is bursty, but one model has a strict latency target.
- Security needs audit evidence.
- Finance needs a three-year cost range.

## Part 1: Discovery

Write the questions you need answered before design.

Cover:

- Workload shape
- Model and data size
- Latency and availability
- Training frequency
- Team workflows
- Data classification
- Recovery targets
- Capacity and budget
- Current cloud dependencies
- Decision makers and success criteria

Mark which missing answers block the design and which can be assumptions.

## Part 2: Architecture

Produce a diagram and a short document covering:

- Projects, accounts, or subscriptions
- Identity and workload identity
- Network zones and egress
- Kubernetes and optional Slurm layer
- GPU pools and capacity policy
- Compute fabric
- Object, shared, and local storage
- Model serving and gateway
- Observability and audit
- CI/CD, IaC, and image supply chain

Name at least two alternatives and why you did not choose them.

## Part 3: Capacity and cost

Size:

- The online inference baseline and burst
- One representative fine-tuning job
- Batch capacity
- Spare capacity
- Storage and data movement

Show low, expected, and high cost cases. State which capacity is committed and which is elastic.

## Part 4: Production plan

Create a 90-day plan:

- Weeks 1-2: criteria and landing zone
- Weeks 3-4: platform baseline
- Weeks 5-6: representative workload
- Weeks 7-8: security, failure, and cost tests
- Weeks 9-10: first service pilot
- Weeks 11-12: production gate and handoff

Adjust the schedule, but give each milestone an owner and exit criterion.

## Part 5: Failure review

Walk through:

1. One GPU node fails during fine-tuning.
2. The online model's p95 TTFT doubles.
3. Object storage is throttled during scale-out.
4. A notebook credential is exposed.
5. Reserved capacity sits at 30% use for a month.

For each, explain detection, containment, recovery, and prevention.

## Part 6: Present it

Prepare two versions:

- A 20-minute technical design review
- A 5-minute decision summary

The short version should cover the decision, cost range, largest risk, and next gate. Do not shrink the technical talk by speaking faster.

## Scoring rubric

| Area | Strong answer |
|---|---|
| Workload | Design follows measured training and inference needs |
| Infrastructure | Compute, network, storage, and scheduling fit together |
| Security | Boundaries and ownership are explicit |
| Reliability | Baselines, spare capacity, recovery, and runbooks exist |
| Economics | Cost is a range with utilization and migration included |
| Delivery | PoC criteria lead to a production gate |
| Communication | Trade-offs are clear and jargon is explained |

## What you should have

- Discovery questions
- Architecture diagram
- Ten-page maximum design document
- Capacity model
- 90-day plan
- Risk register
- Technical and executive presentations

## Go deeper

- [Chapter 30 — AI Solution Architecture in Practice](../07-enterprise-architecture/30-ai-solution-architecture.md)
- [Chapter 31 — Secure AI Platforms](../07-enterprise-architecture/31-secure-ai-platforms.md)
- [Chapter 32 — From Proof of Concept to Production](../07-enterprise-architecture/32-poc-to-production.md)
