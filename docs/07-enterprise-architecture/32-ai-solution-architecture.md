# AI Solution Architecture in Practice

An AI solution architect connects four things that are often discussed separately:

- The business outcome
- The AI workload
- The cloud platform
- The path to production

The job is not to know every GPU register or every model architecture. It is to understand enough of the whole system to find the real constraint, make the main trade-offs visible, and bring in deeper specialists when needed.

## The core questions

Most engagements reduce to four conversations.

**What are we trying to improve?** Start with the outcome, current baseline, decision maker, budget, and deadline.

**What does the workload need?** Identify the model, data, framework, training or inference pattern, performance target, and quality bar.

**What platform fits it?** Design compute, networking, storage, orchestration, identity, security, reliability, and operations as one system.

**How does it reach production?** Define the proof of concept, success criteria, migration, ownership, cost model, and production gates.

A technically correct diagram that skips one of these conversations is incomplete.

## The role beyond the diagram

A solution architect is usually responsible for technical confidence in a decision. That work includes:

- Leading discovery with engineers, platform owners, security teams, and executives
- Turning requirements into an architecture and a measurable proof of concept
- Building enough of the solution to prove the difficult parts
- Running workshops, demonstrations, and technical enablement
- Connecting performance and platform choices to a business case
- Coordinating sales, product, engineering, support, and customer teams around risks and owners
- Bringing repeated customer problems back to product and engineering

The architect may not own the commercial agreement or production service. They do own making technical claims testable and keeping the design tied to the outcome.

## The expected knowledge map

An architect does not need equal depth everywhere. They do need to move across the layers without losing the thread.

| Area | What you should be able to do |
|---|---|
| AI and ML workloads | Explain pre-training, fine-tuning, batch inference, and online inference; identify their different bottlenecks |
| ML methods and evaluation | Match common algorithm families to use cases; discuss features, labels, baselines, error trade-offs, and suitable metrics |
| MLOps | Design a reproducible path from data and training through experiment tracking, validation, registry, deployment, monitoring, and rollback |
| Distributed training | Discuss memory, parallelism, collectives, checkpointing, MFU, and failure recovery |
| Inference serving | Reason about model fit, KV cache, batching, latency, goodput, quantization, and replicas |
| GPU systems | Choose hardware from memory, bandwidth, compute, interconnect, availability, and price |
| Networking | Explain RDMA, InfiniBand, RoCE, NCCL, topology, isolation, and the first debug steps |
| Storage and data | Design dataset loading, checkpoint, model-loading, caching, and migration paths |
| Kubernetes and Slurm | Choose an orchestrator, design GPU scheduling and quotas, and troubleshoot common failures |
| Software and automation | Read and write Python for APIs, health checks, benchmarks, and systems integration |
| Infrastructure as Code | Create repeatable network, identity, cluster, storage, and observability layers |
| Security and identity | Define trust boundaries, private access, workload identity, secrets, tenancy, and audit |
| Reliability | Set acceptance tests, observability, spare capacity, recovery, and operational ownership |
| Economics | Size the workload and explain utilization, reservations, migration costs, and TCO |
| Communication and delivery | Lead discovery, workshops, technical decisions, risk reviews, and handoff across mixed audiences |

You should be able to explain all of these in plain language. You should be hands-on enough to build or debug a representative path through the platform.

## Typical use cases

### Large-scale model training

A team needs to train a model by a fixed date.

The architecture work includes:

- Estimating GPU count from parameters, tokens, deadline, and expected MFU
- Choosing the parallelism strategy
- Keeping frequent communication on the fastest interconnect
- Sizing dataset and checkpoint storage
- Securing capacity and spare nodes
- Defining collective, scaling, checkpoint, and recovery tests

The success measure is not "the cluster exists." It is time-to-train at an acceptable cost and failure rate.

### Fine-tuning and post-training

Several teams need to adapt models using private data.

The design may need:

- Shared base models with separate adapters
- Full fine-tuning, LoRA, or QLoRA paths
- Experiment tracking and artifact storage
- Queues, quotas, and fair sharing
- Data isolation and lineage
- A promotion path from experiment to approved model

This is often a platform problem rather than a single large-cluster problem.

### Enterprise MLOps

A company has many models, but each team trains and deploys them differently.

The architecture work includes:

- Defining how code, data, features, environments, and runs are versioned
- Choosing experiment tracking, pipeline orchestration, and a model registry
- Creating quality, security, schema, latency, and cost gates
- Separating training, model promotion, and service deployment
- Designing batch, online, shadow, canary, and rollback paths
- Monitoring service health, data quality, model behavior, and business outcomes
- Defining who owns labels, retraining decisions, approvals, and production incidents

The goal is not to buy every MLOps product. It is to make one model lifecycle reproducible, safe, observable, and easy to operate before scaling the platform to many teams.

### Self-hosted online inference

A product needs lower cost, more control, or a model unavailable through its current API.

The work includes:

- Testing model quality before infrastructure
- Choosing precision and a serving engine
- Sizing weights and KV cache
- Benchmarking goodput at the latency target
- Designing replicas, autoscaling, and cold-start controls
- Adding a gateway, observability, canary rollout, and fallback

The benchmark must use realistic prompts, outputs, and concurrency.

### Batch inference and embeddings

A team needs to process a large dataset without an interactive latency target.

Priorities change:

- Throughput matters more than TTFT
- Interruptible GPUs may be useful
- Work should checkpoint its progress
- Data locality and output writes can dominate
- Scheduling should avoid competing with protected online workloads

Do not copy the online serving architecture just because it already exists.

### A shared enterprise AI platform

Many teams need notebooks, training, inference, and governed access to data.

The architecture must cover:

- Tenant and workload boundaries
- Identity and self-service access
- GPU pools and quota policy
- Kubernetes, Slurm, or both
- Approved images and model sources
- Cost allocation
- Platform support and upgrade ownership

The hard part is usually the operating model, not provisioning the first cluster.

### Migration from on-premises or another cloud

The customer wants better capacity, price, or performance without moving everything at once.

Cover:

- Data gravity and egress
- Identity and network integration
- Container, driver, and framework compatibility
- Performance baselines from the current environment
- A representative workload first
- Hybrid operation during migration
- An exit path if the result misses the target

A migration plan without a data plan is not credible.

### Regulated or sensitive AI

The workload uses private or controlled data.

The design starts with:

- Region and residency
- Data classification
- Identity and separation of duties
- Private networking and controlled egress
- Encryption and key ownership
- Audit evidence and retention
- Model, dataset, checkpoint, and log handling

Security specialists should validate the controls. The architect still owns making them part of the design rather than a late review.

### Performance investigation

A training or inference workload is slower or more expensive than expected.

Use a layer-by-layer comparison:

1. Make the workload and software versions comparable.
2. Prove single-GPU performance.
3. Prove one server.
4. Prove the cluster network.
5. Prove storage and data loading.
6. Run the full workload.
7. Find stragglers and scaling cliffs.

The deliverable is a measured cause and regression test, not a list of tuning flags.

### AI applications on open models

A team wants retrieval, tool use, or an agent workflow on a self-hosted model.

This joins both parts of the curriculum:

- Part I covers prompts, context, tools, evaluation, safety, and agent state.
- Part II covers the serving platform, capacity, security, and economics.

Model compatibility matters. Test tool calling, structured output, context length, and task quality before committing to the serving design.

## Discovery that changes the design

Useful questions uncover constraints:

| Topic | Questions |
|---|---|
| Outcome | What improves, by how much, and who decides? |
| Workload | Training or inference? Which model, data, framework, and precision? |
| ML lifecycle | Where do labels come from? How are runs tracked, models approved, deployed, monitored, retrained, and rolled back? |
| Scale | Average, peak, growth, deadline, and concurrency? |
| Performance | Which latency, throughput, quality, or completion-time target? |
| Data | Where is it now, how large is it, and where may it go? |
| Platform | Kubernetes, Slurm, managed service, or existing standards? |
| Security | Identity provider, tenancy, residency, audit, and egress rules? |
| Operations | Who upgrades, monitors, responds, and pays? |
| Decision | What will the proof of concept prove, and what happens after it passes? |

If the answers do not affect the architecture, the questions are probably too generic.

## Typical deliverables

A strong engagement leaves reusable artifacts:

- Discovery notes and agreed success criteria
- Architecture diagram and decision record
- Workload sizing and TCO model
- Proof-of-concept plan
- Infrastructure as Code and deployment examples
- Benchmark and acceptance-test results
- Model lifecycle, promotion gates, and monitoring plan
- Security and data-flow review
- Production-readiness checklist
- Runbooks, ownership, and escalation path
- Technical workshop and short decision summary

The artifacts should help the team operate without the architect in the room.

## How hands-on is hands-on?

You should be able to:

- Write a small Python integration or diagnostic tool
- Read and modify Terraform
- Inspect Kubernetes events, resources, scheduling, networking, and storage
- Run and interpret GPU, NCCL, and storage tests
- Read a training or inference performance report
- Trace a model from its training data and run to its deployed version and production signals
- Explain where the workload is waiting

You do not need to replace the kernel, fabric, storage, security, or ML specialist. You need enough depth to identify the right problem, provide useful evidence, and work with that specialist as a peer.

## Things to watch for

**Jumping from requirements to a product list.** Architecture starts with workload behavior and constraints.

**Treating the proof of concept as the finish line.** Identity, operations, capacity, and ownership decide whether it reaches production.

**Overstating technical depth.** A clear boundary plus a good diagnostic plan builds more trust than pretending to know.

**Explaining only to infrastructure engineers.** The same design needs a technical version and a decision-level version.

**Becoming a permanent dependency.** Leave code, measurements, decisions, and runbooks behind.

## Go deeper

- [Workload Sizing and Economics](../05-ai-workloads/24-workload-sizing.md)
- [The MLOps Lifecycle](../05-ai-workloads/25-mlops-lifecycle.md)
- [Secure AI Platforms](33-secure-ai-platforms.md)
- [From Proof of Concept to Production](34-poc-to-production.md)
- [Workshop W13 — Workload Sizing and TCO](../05-workshops/W13-workload-sizing-tco.md)
- [Workshop W14 — Enterprise AI Platform Capstone](../05-workshops/W14-enterprise-ai-platform.md)
- [Workshop W15 — End-to-End MLOps](../05-workshops/W15-end-to-end-mlops.md)

---

[← Previous: Reliability and Performance](../06-ai-infrastructure/31-reliability-and-performance.md) · [Next: Secure AI Platforms →](33-secure-ai-platforms.md)
