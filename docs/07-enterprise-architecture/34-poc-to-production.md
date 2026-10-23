# From Proof of Concept to Production

A proof of concept should answer a decision, not produce a demo.

The technical work matters. So do the success criteria, owners, economics, risks, and handoff. Without those, a fast benchmark can sit unused for months.

## Discovery before architecture

Start with the outcome:

- What must improve?
- What is the current baseline?
- Who decides whether the project continues?
- Which constraints cannot move?
- What would make the result untrustworthy?

Then capture the workload:

- Training, fine-tuning, batch inference, or online inference
- Model, framework, precision, and data
- Performance and availability targets
- Security and residency requirements
- Current environment and dependencies
- Team skills and preferred operating model
- Budget and timeline

Do not hide missing information behind architecture diagrams.

## Write success criteria first

Good criteria are measurable and tied to the decision.

For training:

- Target MFU or step time
- Scaling efficiency at the target node count
- Collective bandwidth against the platform baseline
- Checkpoint write and restore time
- Recovery from a failed node
- Estimated time and cost for the full run

For inference:

- Task-quality score
- TTFT and inter-token latency percentiles
- Goodput at a defined concurrency and request mix
- Cost per million tokens at expected utilization
- Scale-out and recovery time

For the platform:

- Environment creation from code
- Identity and network controls
- Monitoring and audit evidence
- Ownership and support path

For an ML lifecycle:

- Training run links to versioned code, data, features, and environment
- Model quality passes task and important-segment thresholds
- Schema, latency, resource, security, and governance gates pass
- Deployed service reports its exact model version
- Monitoring covers service, data, model, and business behavior
- Promotion and rollback are tested

Agree on the test data, commands, and pass/fail thresholds.

## A two-week shape

**Days 1-2: foundation**

- Confirm criteria and owners.
- Provision through IaC.
- Validate access, data path, and capacity.

**Days 3-5: platform baseline**

- Run GPU diagnostics.
- Run single-node and multi-node collective tests.
- Run storage benchmarks.
- Save the environment and results.

**Days 6-9: real workload**

- Run representative code and data.
- Measure quality, performance, and cost.
- Tune only after capturing a baseline.

**Days 10-11: failure and security**

- Drain or remove a node.
- Test restart, logging, access boundaries, and rollback.

**Days 12-14: decision**

- Compare with the current environment.
- Document gaps and risks.
- Produce the production plan and cost range.

The schedule can change. The sequence should not: criteria, platform proof, workload proof, failure proof, decision.

## The architecture document

Keep it useful:

1. Context and goals
2. Requirements and constraints
3. Assumptions
4. Proposed architecture
5. Workload sizing and cost range
6. Security and data boundaries
7. Reliability and operations
8. Alternatives and trade-offs
9. Migration plan
10. Open decisions

Include a diagram, but do not make the diagram carry every detail.

## Technical and business cases meet

A platform can pass its benchmark and still fail the adoption decision.

Include:

- Capacity commitment
- Utilization forecast
- Data transfer and migration
- Platform staffing
- Support and escalation
- Contract or provider concentration risk
- Exit path
- Value of faster experiments or lower latency

TCO should show a range and the assumptions that move it.

## Production gates

Before production:

- IaC reviewed and repeatable
- Capacity secured
- Identity federation and workload identity in place
- Private network and controlled egress
- Data protection and retention defined
- Dashboards, alerts, and runbooks ready
- Acceptance tests automated
- Model validation, registry, promotion, and rollback path tested
- Production labels, drift review, and retraining ownership defined
- Backup and restore tested
- Failure and rollback drills passed
- Cost ownership assigned
- Platform and workload support owners named

Not every system needs every control at the same depth. Every skipped gate should be a conscious risk decision.

## Handoff

The team operating the platform should be involved before the end.

Handoff includes:

- Source repositories and versioned configurations
- Architecture decisions
- Baseline results
- Upgrade and patch procedure
- Runbooks and escalation paths
- Capacity and cost dashboards
- Known limits
- A backlog for gaps found during the PoC

A platform that only its builders can run is still a prototype.

## Practice scenarios

Use the same method on different problems:

1. Size a cluster to train a large dense model by a fixed date.
2. Move steady inference traffic from a managed API to an open model.
3. Build a regulated platform for private fine-tuning and serving.
4. Diagnose a workload that is 30% slower than its previous environment.

The answer should always include clarifying questions, rough math, architecture, risks, a test plan, and the decision gate.

## Things to watch for

**Starting the PoC before defining the decision.** Teams optimize whatever is easiest to measure.

**Using synthetic benchmarks only.** They prove the platform, not the customer's workload.

**Using the real workload only.** When it is slow, you cannot isolate the layer.

**Treating the end presentation as documentation.** Operators need code, baselines, and runbooks.

## Go deeper

- [Workshop W13 — Workload Sizing and TCO](../05-workshops/W13-workload-sizing-tco.md)
- [Workshop W14 — Enterprise AI Platform Capstone](../05-workshops/W14-enterprise-ai-platform.md)
- [Workshop W15 — End-to-End MLOps](../05-workshops/W15-end-to-end-mlops.md)
- [Google Cloud Architecture Framework](https://cloud.google.com/architecture/framework)
- [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)
- [Azure Well-Architected Framework](https://learn.microsoft.com/azure/well-architected/)

---

[← Previous: Secure AI Platforms](33-secure-ai-platforms.md)
