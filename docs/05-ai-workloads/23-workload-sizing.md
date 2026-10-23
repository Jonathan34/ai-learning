# Workload Sizing and Economics

Sizing starts with a workload, not a GPU name.

The first answer should be a range with assumptions. The second answer should be a benchmark plan that replaces those assumptions with measurements.

## Start with the requirement

For training, ask:

- Model size and architecture: dense or mixture of experts
- Number of training tokens
- Target completion time
- Precision and framework
- Existing parallelism plan
- Dataset size and location
- Checkpoint size and recovery target
- Capacity model: reserved, on-demand, or interruptible

For inference, ask:

- Model and precision
- Prompt and output length distributions
- Peak and average concurrency
- TTFT and inter-token latency targets
- Availability target
- Traffic shape and growth
- Quality constraints on quantization
- Region and data-boundary requirements

If these are missing, the design is still discovery.

## Training: from tokens to GPU count

Use the dense-transformer estimate:

```text
training FLOPs ≈ 6 × parameters × tokens

GPU count ≈ training FLOPs /
            (deadline seconds × peak FLOP/s per GPU × expected MFU)
```

Example: a 70B model trained on 2T tokens needs roughly `8.4e23` FLOPs. At an assumed 40% MFU, the theoretical calculation may suggest hundreds of high-end GPUs for a one-month target.

Round up for:

- Evaluation and checkpoint time
- Failures and restarts
- Spare nodes
- Data-pipeline uncertainty
- A lower-than-expected MFU

Then test the customer's code on a smaller cluster. The PoC should measure MFU, scaling efficiency, collective performance, and checkpoint time.

## Inference: from traffic to replicas

Start with peak demand:

```text
peak output tokens/s =
  peak requests/s × average output tokens
```

Then benchmark one replica using the real request distribution:

```text
replicas =
  ceil(peak required tokens/s / goodput per replica)
  + availability headroom
```

Use **goodput**, not maximum tokens per second. Goodput counts only work that still meets the latency target.

Check memory separately:

```text
replica memory =
  weights + peak KV cache + runtime overhead
```

If the model fits on one GPU but misses latency, a wider tensor-parallel replica may help. If one replica meets latency, more replicas are usually the simpler path to throughput.

## Cost per unit of work

For self-hosted inference:

```text
cost per 1M output tokens =
  hourly replica cost /
  (goodput tokens/s × 3600) × 1,000,000
```

At 30% average use, the effective unit cost is much higher than the fully loaded benchmark. Include the real traffic curve.

Training cost begins with GPU-hours:

```text
GPU-hours = GPU count × run hours
```

Then add storage, data transfer, platform overhead, failed runs, engineering work, and unused reserved capacity.

## TCO is wider than the cloud bill

**Total cost of ownership (TCO)** includes:

- Compute, storage, and network
- Reservations that sit idle
- Data migration and egress
- Platform engineering and on-call work
- Time lost waiting for experiments
- Failed or repeated runs
- Security and compliance work
- The cost of changing providers later

A cheaper GPU-hour can produce a more expensive workload if the cluster is unreliable or hard to use.

## Capacity shapes the architecture

Use reserved capacity for steady baselines and long runs. Use on-demand capacity for bursts and experiments. Use interruptible capacity only for work that can restart cleanly.

For mixed workloads, keep the scarce fleet busy:

- Training during planned windows
- Batch inference in gaps
- Online inference on a protected baseline
- Development on smaller or partitioned GPUs

Utilization is often the largest economic lever.

## Show uncertainty

A useful estimate includes:

| Input | Low | Expected | High |
|---|---:|---:|---:|
| MFU or serving goodput | measured range | likely value | stretch result |
| Failure overhead | best case | observed baseline | stress case |
| Utilization | traffic floor | forecast | peak |
| Unit price | committed | blended | on-demand |

Run the cost model across all three columns. A single precise number hides more than it explains.

## The sizing deliverable

Keep the first document short:

1. Workload and success criteria
2. Assumptions
3. Capacity range
4. Proposed architecture
5. Cost range
6. Main risks
7. Benchmark plan
8. Decision gate

If the benchmark does not change the estimate, it was probably not testing the important assumptions.

## Things that trip people up

**Starting with the newest GPU.** Memory, bandwidth, availability, and price may point elsewhere.

**Using peak hardware throughput in the schedule.** Real workload efficiency belongs in the denominator.

**Forgetting spare capacity.** N+1 means something different when one replica uses eight GPUs.

**Comparing a fully loaded self-hosted benchmark with a managed API bill at average traffic.** Use the same demand curve and quality bar.

## Go deeper

- [Chinchilla scaling paper](https://arxiv.org/abs/2203.15556)
- [MLPerf Training results](https://mlcommons.org/benchmarks/training/)
- [MLPerf Inference results](https://mlcommons.org/benchmarks/inference-datacenter/)
- [Workshop W13 — Workload Sizing and TCO](../05-workshops/W13-workload-sizing-tco.md)
- [AI infrastructure video guide](../reference/ai-infrastructure-videos.md)

---

[← Previous: Self-Hosted Inference](22-self-hosted-inference.md) · [Next: GPU Systems →](../06-ai-infrastructure/24-gpu-systems.md)
