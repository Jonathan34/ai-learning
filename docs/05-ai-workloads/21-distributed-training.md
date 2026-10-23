# Distributed Training Systems

Training a model is a distributed systems problem with unusually expensive workers.

Every GPU runs the same sequence: forward pass, loss, backward pass, optimizer step. At scale, the GPUs must exchange data between those steps. One slow GPU, link, or data loader can hold up the whole job.

## What training needs

| Resource | What uses it | Common failure |
|---|---|---|
| GPU compute | Matrix multiplication in forward and backward passes | Kernels, batch size, or precision leave the GPU underused |
| GPU memory | Weights, gradients, optimizer state, activations | The model or batch does not fit |
| Network | Gradients, parameters, activations, or tokens moving between GPUs | Communication takes over the step time |
| Storage | Dataset reads and checkpoint writes | GPUs wait for data or checkpoint I/O |
| Scheduler | Allocating all workers together | Part of a job starts and holds scarce GPUs |

This is why a healthy training platform is more than a set of GPU virtual machines.

## The memory problem

Inference mainly stores model weights and a working cache. Full training stores more:

- Model weights
- Gradients
- Optimizer state
- Activations saved for the backward pass

For mixed-precision training with Adam, a useful first estimate is about **16 bytes per parameter**, before activations. A 70-billion-parameter model is therefore around 1.12 TB of model and optimizer state.

That estimate is not a capacity plan. Framework choices change it. It is a quick way to prove that the job needs sharding.

Activation checkpointing reduces memory by discarding some activations and recomputing them later. It trades extra compute for lower memory use.

## The parallelism toolbox

There is no single "distributed training" switch. Different kinds of parallelism split different things.

| Method | What is split | Main communication | Good default boundary |
|---|---|---|---|
| Data parallelism | The batch | Gradient all-reduce | Across replicas |
| FSDP / ZeRO | Parameters, gradients, optimizer state | All-gather and reduce-scatter | Across a fast cluster |
| Tensor parallelism | Matrices inside each layer | Collective operations every layer | Inside a fast GPU interconnect domain |
| Pipeline parallelism | Groups of layers | Activations between stages | Across nodes when the model needs it |
| Expert parallelism | Mixture-of-experts layers | All-to-all token exchange | Where the network can sustain it |

**FSDP** is Fully Sharded Data Parallel. It gives each worker only part of the model state, then gathers each layer when needed.

**Tensor parallelism** is much more sensitive to latency because GPUs communicate within every layer. Keep it on the fastest available interconnect. On a conventional eight-GPU server, that usually means inside the server.

Large runs combine these methods. For example: tensor parallel inside a server, pipeline parallel across a few servers, then sharded data parallel across the rest.

## Collectives are the hidden API

Distributed frameworks call collective communication operations:

- **All-reduce:** combine gradients and return the result to every GPU.
- **All-gather:** collect shards so every worker has the full value.
- **Reduce-scatter:** combine values, then give each worker one shard.
- **All-to-all:** every worker sends different data to every other worker.

NCCL, the NVIDIA Collective Communications Library, maps those operations onto NVLink, PCIe, InfiniBand, RoCE, or sockets.

When a training job is slow, "the network is fine" is not enough. The useful question is whether the collective used by this job reaches the expected bandwidth at this node count and message size.

## Performance: useful work, not peak FLOPS

**Model FLOPs Utilization (MFU)** estimates how much of the GPU's theoretical math rate becomes useful model work:

```text
MFU = achieved model FLOP/s / peak hardware FLOP/s
```

Peak numbers are not a promise. Communication, data loading, recomputation, pipeline bubbles, and failures all reduce useful throughput.

Track **goodput** too: the share of wall-clock time that produces useful training progress. A job can have decent MFU while it is running and still have poor goodput because it restarts often.

## Training time estimate

A common estimate for dense transformer training is:

```text
training FLOPs ≈ 6 × parameters × training tokens

time ≈ training FLOPs /
       (GPU count × peak FLOP/s per GPU × expected MFU)
```

Treat the result as a range. MFU is an assumption until the real code, data path, and parallelism run on the target cluster.

## Failure is part of the design

At hundreds or thousands of GPUs, hardware and network failures are normal.

A long-running job needs:

- Sharded checkpoints that do not funnel through one process
- Asynchronous writes where the framework supports them
- Automatic node drain and replacement
- A restart or requeue path
- Spare capacity
- Health checks that catch slow nodes, not only dead nodes

Checkpoint frequency is a trade-off. Write too often and the job spends time on storage. Write too rarely and each failure destroys more work.

## A useful debug order

When scaling efficiency is poor:

1. Prove single-GPU compute.
2. Prove one server across its GPU interconnect.
3. Run collective benchmarks across servers.
4. Check placement and topology.
5. Replace the real dataset with synthetic data.
6. Compare per-rank step times to find stragglers.
7. Profile compute and communication overlap.

This order separates application, fabric, storage, and hardware problems.

## Things to watch for

**Adding GPUs before fixing the bottleneck.** More workers can make communication or storage pressure worse.

**Using average step time.** One slow rank controls a synchronous job. Look at the distribution.

**Assuming FSDP removes the network problem.** It saves memory by communicating more.

**Checkpointing to one shared file.** It creates a bottleneck and a large failure domain.

## Go deeper

- [PyTorch FSDP documentation](https://pytorch.org/docs/stable/fsdp.html)
- [ZeRO paper](https://arxiv.org/abs/1910.02054)
- [Megatron-LM parallelism paper](https://arxiv.org/abs/2104.04473)
- [NCCL documentation](https://docs.nvidia.com/deeplearning/nccl/user-guide/docs/)
- [AI infrastructure video guide](../reference/ai-infrastructure-videos.md#distributed-training)

---

[Next: Self-Hosted Inference →](22-self-hosted-inference.md)
