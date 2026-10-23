# Workshop W13 — Workload Sizing and TCO

**Goal:** Turn a workload description into a capacity range, cost range, and benchmark plan.

**Time:** 3-4 hours

**Prerequisites:** Chapters 21-29

---

## Choose one workload

### Training

A team wants to train a 70B dense model on 2T tokens within 30 days.

Assume:

- BF16 training
- A modern high-end GPU
- Expected MFU range of 35-50%
- Checkpoint every 30 minutes
- Five percent capacity headroom

### Inference

A service produces 50 million output tokens per day with a 3:1 peak-to-average ratio.

Assume:

- A 70B model
- FP8 weights
- Interactive latency targets
- One-region deployment with N+1 capacity
- Average utilization between 30% and 60%

Replace these with a real workload if you have one.

## Part 1: Write the assumptions

Create a machine-readable input file:

```yaml
workload: training
parameters: 70e9
tokens: 2e12
deadline_days: 30
mfu:
  low: 0.35
  expected: 0.42
  high: 0.50
```

Keep facts and assumptions separate. Add a source or owner for every fact.

## Part 2: Build the calculator

For training, calculate:

- Total FLOPs
- GPU count for each MFU case
- GPU-hours
- Checkpoint size estimate
- Checkpoint write bandwidth
- Capacity with headroom

For inference, calculate:

- Average and peak tokens per second
- Weight memory
- KV-cache scenarios
- Replica count from measured goodput
- N+1 capacity
- Cost per million tokens at several utilization levels

Do not invent serving goodput. Make it a required benchmark input.

## Part 3: Add a cost range

Include:

- GPU hourly cost
- Storage
- Data transfer
- Platform overhead
- Spare capacity
- Failure or restart overhead
- Engineering cost as a separate line

Run low, expected, and high cases. Show which assumption moves the result most.

## Part 4: Design the platform

Draw a small architecture:

- GPU node shape and count
- Compute fabric
- Scheduler
- Storage tiers
- Observability
- Identity and network boundary

Explain why each component follows from the workload.

## Part 5: Write the benchmark plan

Choose the assumptions that need measurement:

- MFU at a smaller scale
- Scaling efficiency
- Collective bandwidth
- Checkpoint time
- Inference goodput at the latency target
- Model quality after quantization
- Cold-start time

Define pass/fail criteria and the decision each test supports.

## Deliverable

Produce a two-page decision note:

1. Workload and target
2. Assumptions
3. Capacity and cost range
4. Architecture
5. Risks
6. Benchmark plan

The spreadsheet or code belongs in the same repository as the note.

## Review questions

- What happens if MFU is ten points lower?
- What happens if traffic is twice as bursty?
- Which capacity is reserved?
- Which work can be interrupted?
- How much does one failed node affect the service?
- What evidence would make you pick a different GPU?

## Go deeper

- [Chapter 24 — Workload Sizing and Economics](../05-ai-workloads/24-workload-sizing.md)
- [MLPerf results](https://mlcommons.org/benchmarks/)
