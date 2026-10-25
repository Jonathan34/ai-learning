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

Add runtime overhead and the **KV cache**.

## What the KV cache stores

At every transformer layer, attention creates three vectors for each token:

- **Query:** what information this token is looking for
- **Key:** what information this token can be matched on
- **Value:** the information this token can contribute

During generation, the new token's query is compared with the keys from earlier tokens. The resulting scores decide how much of each earlier value to use:

```text
new token's query
        ↓ compare
cached keys from earlier tokens
        ↓ produce attention scores
cached values from earlier tokens
        ↓ weighted combination
attention result for the new token
```

The query for an earlier token is no longer needed. Its key and value are needed every time a later token attends to it, so the server keeps them in the KV cache.

Without the cache, generating each new token would require recalculating keys and values for the entire sequence. The cache trades GPU memory for much less repeated computation.

## Attention heads and KV heads

A model does not perform only one attention calculation per layer. It divides its hidden representation into several **query heads**, usually called attention heads. Each head can learn a different way to match information.

For example:

```text
hidden size: 4,096
query heads: 32
head dimension: 4,096 / 32 = 128 values per head
```

A **KV head** is one key projection and one value projection. The number of KV heads depends on the model's attention design:

| Design | Example with 32 query heads | Sharing | Relative KV-cache size |
|---|---:|---|---:|
| Multi-head attention, MHA | 32 KV heads | Each query head has its own key and value head | 1× |
| Grouped-query attention, GQA | 8 KV heads | Four query heads share each KV head | 1/4 |
| Multi-query attention, MQA | 1 KV head | All 32 query heads share one KV head | 1/32 |

GQA and MQA reduce cache memory and the amount of cache data read during decoding. The trade-off is that more query heads share the same key and value representation. The model designer chooses this structure during model development and training; a serving operator normally cannot change the KV-head count.

The **head dimension** is the number of values in one head's key or value vector. It is often:

```text
head dimension = hidden size / query heads
```

Do not assume that formula for every architecture. Some models store an explicit head dimension.

The model configuration commonly exposes fields such as:

```text
num_hidden_layers
hidden_size
num_attention_heads
num_key_value_heads
head_dim
torch_dtype
```

The exact names vary. Use the model configuration and serving engine rather than guessing from the model's parameter count.

## Worked KV-cache calculation

A useful estimate is:

```text
KV bytes per token =
  2 × layers × KV heads × head dimension × bytes per value
```

The first `2` means one key and one value.

Assume a model has:

- 32 layers
- 32 query heads
- 8 KV heads using GQA
- Head dimension of 128
- BF16 cache values at 2 bytes each

```text
KV bytes per token
  = 2 × 32 × 8 × 128 × 2
  = 131,072 bytes
  = 128 KiB, where 1 KiB = 1,024 bytes
```

One request holding 8,192 prompt and generated tokens needs:

```text
128 KiB × 8,192 tokens = 1 GiB
```

One GiB is `1,024³` bytes. One hundred requests at that length need about `100 GiB` of KV cache across the model replica, before allocator and runtime overhead.

If the same example used MHA with 32 KV heads, the cache would be four times larger:

```text
512 KiB per token
4 GiB per 8,192-token request
400 GiB for 100 such requests
```

For requests with different lengths, add their current sequence lengths:

```text
total KV cache =
  KV bytes per token ×
  sum of current tokens across active requests
```

Tensor parallelism may split that cache across the GPUs in one replica. It lowers the cache stored on each GPU but does not change the replica's total.

Long context and high concurrency can make the cache larger than the weights.

GQA or MQA, lower-precision cache values, shorter context limits, prefix sharing, paging, sliding-window attention, and offload can reduce the GPU footprint. Paging mainly reduces wasted space; offload saves GPU memory but adds data movement and may hurt latency.

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

[← Previous: Distributed Training](22-distributed-training.md) · [Next: Workload Sizing and Economics →](24-workload-sizing.md)
