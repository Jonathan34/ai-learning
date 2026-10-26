# Quiz — AI Infrastructure Solutions Architect Interview

This is a practice set for senior, customer-facing AI infrastructure roles.
It is based on the usual responsibilities of the job, not on leaked or
confirmed questions from one company.

The model answers show the shape of a strong answer. Do not memorize them.
Use examples from your own work and say when your knowledge is conceptual
rather than hands-on.

## How to use it

1. Answer without opening the model answer.
2. Keep a technical answer under three minutes.
3. For an architecture case, spend two minutes clarifying before designing.
4. Open the answer and mark what you missed.
5. Repeat weak questions a day later.

Score each answer from `0` to `2` in five areas:

| Area | What good looks like |
|---|---|
| Directness | Answer the question before adding background |
| Technical depth | Explain one layer below the product name |
| Trade-offs | Name an alternative and when it would be better |
| Customer judgment | Connect the design to an SLO, risk, or business goal |
| Evidence | Use rough math, a measurement, or a real example |

`8-10` is strong. `5-7` needs another pass. Below `5` usually means the
answer is a list of terms rather than an explanation.

## Role and customer discovery

### 1. Walk me through your background

??? note "Model answer"

    Keep this to about 60-90 seconds:

    1. What you do now and the scale or outcomes you own.
    2. Two earlier experiences that explain why you fit this work.
    3. Why you want to work closer to AI infrastructure and customers.

    A useful opening is: "I am a senior engineer and architect with a
    background in distributed systems, cloud platforms, and production
    operations. More recently, I have been applying that experience to AI
    systems. I am now looking for a role where I can connect customer
    workloads to the infrastructure and operating model they need."

    Replace the generic claims with your own numbers and results. Do not read
    your resume aloud.

### 2. What does an AI Solutions Architect actually do?

??? note "Model answer"

    The architect moves a customer from an unclear goal to a production
    decision:

    - Discover the workload, business goal, constraints, and stakeholders.
    - Turn them into an architecture and a capacity range.
    - Prove the uncertain parts with a benchmark or proof of concept.
    - Plan security, operations, cost, migration, and production ownership.
    - Explain trade-offs to engineers and decision-makers.
    - Bring recurring customer problems back to product and engineering.

    The output is not only a diagram. It is a decision supported by evidence.

### 3. How do you start when a customer says, "We need an AI platform"?

??? note "Model answer"

    Ask what decision or workflow the platform must improve. Then clarify:

    - Training, fine-tuning, batch inference, or an online service?
    - Model size, data location, traffic, latency, and deadline?
    - Existing cloud, Kubernetes, MLOps, and security standards?
    - Who builds it, who operates it, and who approves it?
    - What result would make the project worth funding?

    Separate facts from assumptions. Do not select GPUs or products until the
    workload is clear.

### 4. A customer asks for an exact GPU count before sharing workload data. What do you do?

??? note "Model answer"

    Give a range, not false precision. State the assumptions that drive it,
    such as model size, precision, context length, concurrency, training
    tokens, deadline, and expected efficiency.

    Then propose the smallest benchmark that can replace the weakest
    assumptions with measurements. A useful answer sounds like: "The current
    range is 32-64 GPUs. The largest uncertainty is measured throughput at
    your latency target. A two-day test on four or eight GPUs will narrow
    that range."

### 5. How do you handle a customer who has already chosen the wrong design?

??? note "Model answer"

    Start with the goal behind the choice. Show the consequence using their
    own SLO, cost, risk, or operating constraints. Offer an alternative and a
    small test that compares both designs.

    Do not win the argument and lose the customer. If the chosen design is
    safe but imperfect, record the trade-off and leave a path to change it.
    Push harder when it creates a security, reliability, or data-loss risk.

## AI, ML, and MLOps

### 6. Does an AI infrastructure architect need to code ML algorithms?

??? note "Model answer"

    Usually not from scratch. The architect should know the main problem
    families—classification, regression, forecasting, ranking, clustering,
    recommendation, and generative models—and when each fits.

    They should be able to read a training loop, understand features, labels,
    loss, metrics, and overfitting, and discuss the infrastructure
    consequences of the model choice. Practical Python, APIs, SQL, containers,
    and infrastructure as code are more likely to matter than deriving a new
    optimization algorithm.

    A coding or ML debugging round is still possible. "I do not need it for
    the job" is not a reason to be unable to read it.

### 7. When would you use conventional ML instead of an LLM?

??? note "Model answer"

    Use the simplest approach that meets the requirement.

    Conventional ML is often better for structured data, stable labels,
    forecasting, ranking, anomaly detection, or high-volume decisions with
    tight latency and cost limits. An LLM fits language-heavy work, flexible
    extraction, generation, summarization, and tool use.

    Compare both with a simple baseline. The decision should include quality,
    latency, cost, explainability, data availability, and how errors affect
    users.

### 8. Explain one model-training step

??? note "Model answer"

    1. The **forward pass** uses the current weights to make predictions.
    2. The **loss** turns the prediction error into a number.
    3. The **backward pass** calculates how each weight contributed to that
       loss. Those values are the gradients.
    4. The **optimizer** changes the weights slightly in the direction
       expected to lower the loss.

    Real models repeat this for millions or billions of weights and many
    batches. Validation data checks whether lower training loss also means
    better results on unseen examples.

### 9. How do you know whether a model is good enough for production?

??? note "Model answer"

    Start with the business failure modes, then choose metrics that represent
    them. Use a held-out test set with normal, difficult, and unsafe cases.
    Compare against the current system or a simple baseline.

    Offline quality is only one gate. Also test latency, cost, throughput,
    reliability, security, and human escalation. For an LLM system, evaluate
    the whole path: retrieval, prompts, tools, model output, and user outcome.

### 10. What is MLOps?

??? note "Model answer"

    MLOps is the system that makes model changes repeatable and safe. It
    covers:

    - Versioned data, code, configuration, and models.
    - Reproducible training and evaluation pipelines.
    - Experiment tracking and a model registry.
    - Approval, deployment, canary or shadow testing, and rollback.
    - Production monitoring for quality, drift, latency, and cost.

    CI/CD deploys software. MLOps adds data and model behavior, which can
    change even when the application code does not.

### 11. What do you monitor after deploying a model?

??? note "Model answer"

    Monitor four layers:

    - Service: errors, latency, saturation, and availability.
    - Infrastructure: GPU memory, utilization, temperature, network, and I/O.
    - Data: schema changes, missing values, and input-distribution drift.
    - Model: task quality, refusals, unsafe output, and business outcomes.

    Define who responds and what action each signal can trigger. A drift chart
    without labels, an owner, or a retraining decision is only a dashboard.

## Distributed training

### 12. Explain data parallelism with an example

??? note "Model answer"

    Each GPU has the same model but receives a different part of the global
    batch.

    With eight GPUs and a global batch of 512 examples, each GPU processes 64
    examples. Each calculates gradients for the same model weights. An
    all-reduce operation combines those gradients so every GPU makes the same
    weight update.

    The GPUs exchange gradients, not their raw training examples.

### 13. How are tensor and pipeline parallelism different?

??? note "Model answer"

    **Tensor parallelism** splits the calculation inside a layer across
    GPUs. It communicates often, so it usually stays within a fast NVLink
    domain.

    **Pipeline parallelism** puts groups of layers on different GPUs. A
    microbatch moves through those stages like work on an assembly line. It
    communicates less frequently but can leave stages idle during pipeline
    bubbles.

    Large jobs combine data, tensor, and pipeline parallelism. The right mix
    depends on model size, memory, network topology, and batch size.

### 14. What problem do FSDP and ZeRO solve?

??? note "Model answer"

    Full training keeps model weights, gradients, and optimizer state. These
    can take roughly 16 bytes per parameter with mixed-precision Adam before
    activations.

    Fully Sharded Data Parallel and ZeRO divide some or all of that state
    across workers. Workers gather the parameters they need for computation,
    then reduce and shard gradients afterward. This lets a model train when a
    full copy of its state would not fit on each GPU. The trade-off is more
    communication and more complicated checkpointing.

### 15. What is a dense transformer, and how is a mixture-of-experts model different?

??? note "Model answer"

    A dense transformer uses the same full set of layers for every token. A
    mixture-of-experts model has several feed-forward expert blocks and sends
    each token to only a small subset.

    Mixture-of-experts models can have many parameters without using all of
    them for every token. They add routing, load-balancing, and all-to-all
    communication challenges.

### 16. GPU utilization is low during training. Where do you look?

??? note "Model answer"

    First split step time into compute, communication, and waiting for data.
    Run with synthetic in-memory data. If utilization improves, inspect file
    sizes, storage latency, data-loader workers, CPU decoding, and
    host-to-device copies.

    If data is not the limit, check batch size, kernel efficiency, pipeline
    bubbles, gradient synchronization, network bandwidth, topology, thermal
    throttling, and slow ranks. An average utilization number does not show
    where time went.

## Self-hosted inference

### 17. Explain prefill and decode

??? note "Model answer"

    **Prefill** processes the input prompt, usually many tokens in parallel.
    It builds the key-value cache and drives time to first token.

    **Decode** generates one new token per sequence at each step. It repeatedly
    reads the model weights and the key-value cache. Decode is often limited
    by memory bandwidth and drives the delay between output tokens.

### 18. Explain attention, query heads, and KV heads without equations

??? note "Model answer"

    Attention is a learned lookup:

    ```text
    Q asks → K matches → V supplies
    ```

    The current position creates a query. It compares that query with keys
    from positions it can see. The match scores decide how much of each value
    to combine.

    A query head is one searcher performing that lookup. A KV head is one set
    of searchable keys and values. A model runs several query heads in
    parallel, then combines their results.

### 19. What is the KV cache, and why can it become large?

??? note "Model answer"

    The cache stores the keys and values already calculated for prompt and
    generated tokens. Without it, the server would recalculate them at every
    decode step.

    A useful estimate is:

    ```text
    bytes per token =
      2 × attention layers × KV heads × head dimension × bytes per value
    ```

    For 32 layers, 8 KV heads, a head dimension of 128, and two-byte values:

    ```text
    2 × 32 × 8 × 128 × 2 = 128 KiB per token
    ```

    An 8,192-token request then needs about 1 GiB of cache. Multiply by active
    requests. Long context and concurrency can consume more memory than the
    weights.

### 20. What are MHA, GQA, and MQA?

??? note "Model answer"

    With multi-head attention, each query head has its own KV head. With
    grouped-query attention, several query heads share a KV head. With
    multi-query attention, all query heads share one KV head.

    Sharing reduces cache size and memory traffic. The model designer chooses
    this during training. A serving operator normally cannot change the head
    count, but can select cache precision, context limits, paging, and other
    serving settings.

### 21. How would you size a self-hosted model endpoint?

??? note "Model answer"

    Start with memory fit:

    ```text
    weight memory = parameters × bytes per parameter
    ```

    Add the KV cache, runtime overhead, and enough room for the target
    concurrency. Then benchmark tokens per second at the required time to
    first token and inter-token latency.

    Replica count comes from measured goodput at the SLO, not peak GPU FLOPs.
    Add traffic bursts, failure headroom, rollout capacity, and model-loading
    time. Compare the result with a managed API using cost per million tokens
    at realistic utilization.

### 22. Throughput improved after batching, but latency became worse. Why?

??? note "Model answer"

    Larger batches use the GPU more efficiently and increase total tokens per
    second. They can also make a request wait longer to join a batch and give
    each active sequence a smaller share of a decode step.

    Tune for the latency SLO, not maximum throughput. Use continuous batching,
    queue limits, admission control, and separate service classes when
    interactive and batch traffic have different goals.

## GPU infrastructure and operations

### 23. How do you choose a GPU for a workload?

??? note "Model answer"

    Check, in order:

    1. Does the model and runtime state fit in memory?
    2. Is the workload limited by compute, memory bandwidth, or communication?
    3. Which numeric precision and software stack are supported?
    4. Does it need fast links within or between nodes?
    5. What capacity is available, at what cost and commitment?

    A newer GPU is not automatically the best value. Benchmark the customer
    workload and compare time or tokens per dollar at the required SLO.

### 24. Explain NVLink, RDMA, and GPUDirect RDMA

??? note "Model answer"

    NVLink is a high-bandwidth GPU-to-GPU connection, usually within a node
    or rack-scale system.

    Remote Direct Memory Access moves data between hosts with little CPU and
    kernel involvement. GPUDirect RDMA lets the network adapter read or write
    GPU memory directly instead of copying through host memory.

    Tensor parallelism benefits from the fastest local links. Data-parallel
    and other cross-node communication use the cluster network. GPU-to-NIC
    topology matters because a poor path adds extra PCIe or CPU-socket hops.

### 25. InfiniBand or RoCE?

??? note "Model answer"

    Both can carry RDMA traffic.

    InfiniBand is an HPC-focused fabric with lossless flow control and mature
    support for large collective communication. RoCE carries RDMA over
    Ethernet and fits teams that want a common Ethernet operating model, but
    reliable performance depends on congestion and loss configuration.

    Choose from measured performance, scale, team skills, isolation needs,
    existing network standards, and cost. Do not reduce the decision to a
    protocol name.

### 26. All-reduce is fast on two nodes and slow on 64. How do you debug it?

??? note "Model answer"

    Reproduce with a collective benchmark at the same node count and message
    size. Then check:

    1. Whether the job is using RDMA rather than falling back to sockets.
    2. GPU, NIC, and NUMA topology.
    3. Link errors, speed, MTU, and one slow node or rank.
    4. Placement, oversubscription, and competing traffic.
    5. The selected collective algorithm and software versions.

    Bisect the node set to find a bad host or fabric segment. Compare
    algorithm bandwidth and bus bandwidth with an accepted baseline.

### 27. How would you design storage for a training cluster?

??? note "Model answer"

    Use tiers:

    - Object storage for durable datasets, checkpoints, and artifacts.
    - A shared or parallel filesystem for high-throughput active data.
    - Local NVMe for staging, cache, and temporary work.

    Pack small files into larger shards. Size aggregate read bandwidth so
    data loaders do not starve GPUs.

    Size checkpoint writes with:

    ```text
    required bandwidth = checkpoint size / acceptable pause
    ```

    Prefer sharded, asynchronous checkpoints and copy them to durable storage.

### 28. Kubernetes or Slurm?

??? note "Model answer"

    Slurm fits research and HPC teams that run large batch jobs and value
    mature queues, gang scheduling, and topology-aware allocation.

    Kubernetes fits platform teams operating containers, online inference,
    services, and shared cloud infrastructure. It needs queueing and gang
    scheduling for large distributed jobs.

    Ask about users, workload mix, existing skills, multi-tenancy, and
    operations. Some platforms run Slurm workloads on infrastructure managed
    by Kubernetes. The interface the research team sees and the control plane
    the platform team operates do not have to be the same.

### 29. A GPU pod is stuck in Pending. What do you check?

??? note "Model answer"

    Start with the scheduler event:

    - Enough free GPUs of the requested type?
    - Taints, tolerations, selectors, affinity, or topology constraints?
    - Namespace quota or queue admission?
    - Gang waiting for the rest of the job?
    - Persistent volume still unbound?
    - GPU device plugin or resource driver healthy?
    - Cluster autoscaler able to add a matching node?

    Do not restart components before reading the event. `Pending` is usually a
    scheduling constraint, not an application failure.

### 30. What makes a shared AI platform production-ready?

??? note "Model answer"

    It needs more than working GPUs:

    - Federated identity, least privilege, workload identities, and audit.
    - Network and tenant isolation, private paths, encryption, and secrets.
    - Quotas, queues, priorities, admission control, and cost attribution.
    - Versioned infrastructure as code and separate environments.
    - Health checks, automatic draining or repair, backups, and tested
      recovery.
    - Metrics across jobs, models, GPUs, network, storage, and user SLOs.
    - A documented upgrade and rollback process.

    Define platform ownership and support boundaries before production.

## Architecture cases

For each case, use this order:

    goal → questions → assumptions → rough math → architecture
         → risks → benchmark or proof of concept → next decision

### 31. Size a cluster to train a 70B dense model on 2T tokens in 30 days

??? note "Model answer"

    Clarify precision, sequence length, codebase, target quality, data
    readiness, checkpoint needs, and whether 30 days includes failures and
    evaluation.

    Start with:

    ```text
    training FLOPs ≈ 6 × parameters × training tokens
    time = FLOPs / (GPUs × peak FLOP/s × expected MFU)
    ```

    Give a range for model FLOPs utilization rather than assuming perfect
    hardware use. Add capacity for failures, restarts, evaluation, and
    scheduling.

    The design also needs a high-bandwidth compute fabric, sharded training
    state, topology-aware scheduling, a data path that feeds every GPU,
    asynchronous checkpoints, health automation, and an initial scaling test.
    The benchmark should verify efficiency before committing the full cluster.

### 32. Design an endpoint for a 70B model with interactive users

??? note "Model answer"

    Ask for prompt and output lengths, concurrent requests, peak traffic,
    time-to-first-token and inter-token SLOs, quality tolerance for
    quantization, regions, and availability.

    Calculate weight and KV-cache memory first. Pick tensor parallelism that
    keeps frequent communication on fast local links. Benchmark one replica
    using the real request-length distribution. Derive replicas from measured
    goodput at the SLO, then add failure and rollout capacity.

    Include routing, admission control, continuous batching, autoscaling,
    weight loading, observability, canary deployment, and overload behavior.

### 33. A regulated customer wants to fine-tune on sensitive data

??? note "Model answer"

    Begin with data classification, residency, retention, identity, audit,
    and isolation requirements. Do not treat "regulated" as one standard.

    Use a private network boundary, federated identity, workload identities,
    least privilege, customer-controlled encryption where required, dedicated
    capacity when shared tenancy is not acceptable, and restricted artifact
    stores. Keep training data, experiment logs, model artifacts, evaluation
    sets, and backups inside the approved boundary.

    Bring security and legal teams into discovery. Build a small threat model
    and prove controls before moving real data.

### 34. Plan a two-week proof of concept

??? note "Model answer"

    Agree on pass/fail criteria before provisioning:

    - Time to first successful workload.
    - Collective, storage, or serving performance at a named scale.
    - Model quality and latency at the target configuration.
    - Failure recovery and operational visibility.
    - Estimated production capacity and cost.

    Provision through infrastructure as code. Run platform baselines before
    the customer workload, then compare the results with the current system.
    End with a decision document, open risks, a production plan, and owners.

### 35. A customer says their workload is 30% slower on your platform

??? note "Model answer"

    Acknowledge the impact and make the comparison reproducible. Check that
    model code, batch size, precision, software versions, data, and topology
    are the same.

    Break step time into GPU compute, collective communication, and data
    waiting. Test one GPU, one node, then multiple nodes. Use synthetic data
    and collective benchmarks to isolate the storage and network paths. Look
    for a slow rank, fallback transport, poor affinity, throttling, or a
    different parallelism configuration.

    Fix the cause, rerun the baseline, and add a regression test. Keep the
    customer updated with evidence rather than guesses.

## Behavioral and communication

### 36. Tell me about a significant technical project

??? note "Model answer"

    Pick one production project with enough depth for follow-up questions.
    Cover:

    - The user or business problem and scale.
    - Your individual responsibility.
    - The architecture and hardest trade-off.
    - A failure, surprise, or decision change.
    - Deployment, operations, and measured result.
    - What you would do differently now.

    Be ready to draw the design and explain one incident. "We built" should
    not hide what you personally decided or implemented.

### 37. Tell me about an unhappy customer

??? note "Model answer"

    Use a case where you changed the outcome, not only managed communication.
    Explain how you acknowledged the impact, created a technical workstream,
    set an update rhythm, found the cause, and prevented recurrence.

    Include evidence: recovery time, performance improvement, avoided cost,
    reduced incidents, or a clear change in customer confidence.

### 38. Tell me about influencing without authority

??? note "Model answer"

    Name the teams, their competing incentives, and why an instruction would
    not have worked. Explain the shared evidence or mechanism you created:
    design review, prototype, metrics, operating goal, or customer signal.

    A strong result is a changed decision or adopted standard, not "everyone
    agreed with me."

### 39. Describe a technical decision you would make differently

??? note "Model answer"

    Choose a real decision with a meaningful consequence. State what you knew
    then, what assumption was wrong, how you detected it, and what you changed.

    Avoid a disguised strength such as "I cared too much." Good judgment
    includes updating your view when the evidence changes.

### 40. Explain a complex infrastructure trade-off to an executive

??? note "Model answer"

    Start with the decision, business impact, and recommendation. Use one or
    two numbers. Keep the technical mechanism available for questions.

    For example: "The lower-cost option misses the launch date because the
    model takes 45 days to train. The recommended option completes in about
    28 days and includes failure headroom. It costs more for one month but
    avoids delaying the launch. We will validate the estimate with a
    32-GPU scaling test before signing the full commitment."

## Practical drills

These may appear as live exercises rather than spoken questions.

### Python

Given per-node GPU metrics in JSON, write a small program that:

- Validates the input.
- Marks nodes unhealthy from explicit thresholds.
- Explains every reason for the decision.
- Returns a stable, machine-readable result.
- Includes tests for missing data and boundary values.

The interviewer is usually testing decomposition, naming, error handling, and
communication more than clever algorithms.

### Infrastructure as code

Sketch a module interface for development, proof-of-concept, and production
AI environments. Separate reusable modules from environment values. Explain
remote state, locking, identity, secrets, policy checks, version pinning, plan
review, and how a failed change is rolled back or repaired.

### Architecture whiteboard

Practice drawing only what affects the decision:

    users or jobs
          ↓
    entry point and identity
          ↓
    scheduler or serving control plane
          ↓
    GPU compute ↔ compute network
          ↕
    data, model, and checkpoint storage
          ↓
    metrics, logs, audit, and cost

Add trust boundaries, failure domains, and ownership. A large diagram is not
automatically a good design.

## Final review

Before the interview, make sure you can answer these without notes:

- What data is stored, moved, or shared at each training step?
- What limits this workload: memory, compute, network, storage, or scheduling?
- Which number is measured and which is only an assumption?
- What happens when a node, model, or deployment fails?
- How does the design move from proof of concept to production?
- What does the customer need to decide next?

## Go deeper

- [ML Algorithms for Architects](../05-ai-workloads/21-ml-algorithms-for-architects.md)
- [Distributed Training](../05-ai-workloads/22-distributed-training.md)
- [Self-Hosted Inference](../05-ai-workloads/23-self-hosted-inference.md)
- [MLOps Lifecycle](../05-ai-workloads/25-mlops-lifecycle.md)
- [AI Solution Architecture](../07-enterprise-architecture/32-ai-solution-architecture.md)
- [Workload Sizing and TCO workshop](../05-workshops/W13-workload-sizing-tco.md)
- [Enterprise AI Platform workshop](../05-workshops/W14-enterprise-ai-platform.md)
