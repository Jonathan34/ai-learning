# Self-Hosted Inference

Calling a hosted model API hides most serving decisions. Running the model yourself exposes them: memory fit, batch policy, cache size, replica count, cold starts, and the latency-throughput trade-off.

## Prefill and decode

An autoregressive model serves a request in two different phases.

**Prefill** processes the input prompt in parallel and builds the key-value cache. It is usually compute-heavy and drives time to first token.

**Decode** generates one token at a time. Each step reads the model weights and the cache. It is often limited by memory bandwidth and drives time per output token.

This distinction matters because one GPU shape or tuning choice may not be best for both.

## What must fit in GPU memory

A first estimate for model weights is:

```text
weight memory = parameter count × bytes per parameter
```

| Format | Approximate bytes per parameter | 70B model |
|---|---:|---:|
| BF16 | 2 | 140 GB |
| FP8 | 1 | 70 GB |
| INT4 / FP4 | 0.5 | 35 GB |

Add runtime overhead and the **KV cache**, which stores attention keys and values for tokens already processed.

A useful cache estimate is:

```text
KV bytes per token =
  2 × layers × KV heads × head dimension × bytes per value
```

Then multiply by tokens in flight across all active requests. Long context and high concurrency can make the cache larger than the weights.

Grouped-query attention, lower-precision cache values, shorter context limits, paging, and offload can reduce the footprint.

## The serving loop

Production model servers do more than call `model.generate()`:

1. Accept and tokenize requests.
2. Schedule prompt work and decode work.
3. Allocate KV-cache blocks.
4. Form batches from active sequences.
5. Run the model.
6. Stream tokens back.
7. Free cache blocks when a sequence ends.

**Continuous batching** changes the batch at each decode step. Finished requests leave and new requests join. This avoids waiting for the longest sequence in a fixed batch.

**Paged attention** stores the KV cache in blocks rather than one large contiguous region per request. This cuts fragmentation and lets the server use more of the available memory.

## Common serving engines

| Engine | Why teams choose it | Trade-off |
|---|---|---|
| vLLM | Fast adoption, continuous batching, paged KV cache, broad model support | Peak performance may need tuning |
| SGLang | Strong prefix reuse and structured or agent-heavy workloads | Smaller operational ecosystem |
| TensorRT-LLM | Highly optimized NVIDIA kernels and low-precision support | More build and engine-management work |
| Triton Inference Server | A general serving layer for several backends and models | It does not remove backend-specific tuning |

The engine is not the architecture. You still need routing, admission control, autoscaling, observability, rollout, and failure handling around it.

## The metrics that matter

- **TTFT:** time to first token
- **TPOT / ITL:** time per output token or inter-token latency
- **End-to-end latency**
- **Throughput:** tokens per second
- **Goodput:** throughput that still meets the latency target
- **Queue depth and queue time**
- **KV-cache use**
- **Cost per million tokens**

Always report percentiles. An acceptable average can hide a bad user experience at p95 or p99.

## Parallelism and replicas

Use the fewest GPUs that fit the model and meet the latency target.

**Tensor parallelism** splits one model replica across GPUs. It can reduce latency and make a large model fit, but it adds communication.

**Replicas** increase throughput and availability. Once one replica meets the latency goal, adding replicas is often simpler than making the replica wider.

For models that need more than one server, schedule the workers as one unit. Otherwise a partial replica can hold GPUs while waiting for the rest.

## Scaling and cold starts

GPU autoscaling is slow compared with ordinary web services. A new node may need to boot, load drivers, pull an image, and copy tens or hundreds of gigabytes of weights.

Useful controls include:

- A warm minimum
- Pre-pulled images
- Weights cached on local NVMe
- Admission control during scale-out
- Queue-depth or in-flight-request metrics
- A smaller fallback model

Scaling on CPU use usually tells you very little about model-server pressure.

## Quality is part of infrastructure

Quantization can change model behavior. A faster or cheaper serving path is not an improvement if it breaks the task.

Keep a task-specific evaluation set. Run it when changing:

- Weight precision
- KV-cache precision
- Model version
- Serving engine
- Parallelism
- Sampling settings

This connects software evaluation to infrastructure work.

## Things to watch for

**Sizing from a vendor benchmark.** Prompt length, output length, concurrency, and latency targets can change the result sharply.

**Optimizing total throughput while missing the latency SLO.** Large batches can make the benchmark look good and the product feel bad.

**Ignoring the cache.** Weight memory is only the start for long-context services.

**Scaling to zero for an interactive service.** Model load time may be measured in minutes.

## Go deeper

- [vLLM documentation](https://docs.vllm.ai/)
- [PagedAttention paper](https://arxiv.org/abs/2309.06180)
- [NVIDIA TensorRT-LLM documentation](https://nvidia.github.io/TensorRT-LLM/)
- [Triton Inference Server documentation](https://docs.nvidia.com/deeplearning/triton-inference-server/)
- [AI infrastructure video guide](../reference/ai-infrastructure-videos.md#inference-serving)

---

[← Previous: Distributed Training](21-distributed-training.md) · [Next: Workload Sizing and Economics →](23-workload-sizing.md)
