---
title: "14 — Local and Edge Inference"
nav_order: 4
parent: "Production"
---
# 14 — Local and Edge Inference

> Status: **outline**. Will expand in full depth.

## The mental model

Local inference — running LLMs on your own hardware, including consumer laptops — has become practical through quantization and efficient runtimes. It's a different design space from hosted inference, with real advantages (privacy, offline, cost at scale) and real constraints (model size, latency on cold start, hardware heterogeneity).

This is the area where the NVIDIA PE job you looked at focuses. If you want to build credibility here, this chapter is the one to take most seriously.

## Planned contents

### The runtime landscape
- **llama.cpp** — CPU-first, GGUF format, broad hardware support, the most widely used local runtime
- **Ollama** — wrapper around llama.cpp with model management, great developer UX
- **vLLM** — GPU-first, throughput-optimized, popular for server-side deployment
- **TensorRT-LLM** — NVIDIA's optimized runtime for their GPUs, best perf on NVIDIA hardware
- **MLC-LLM** — cross-platform, including mobile
- **Apple MLX** — Apple silicon native
- **ONNX Runtime** — more traditional ML runtime, has LLM support

### Quantization
- Why models need to shrink for local inference
- Formats: GGUF (llama.cpp ecosystem), AWQ, GPTQ, INT8, INT4
- Quality vs size trade-offs
- Hardware-specific quantization (which formats run on which hardware)

### Hardware
- CPU inference: surprisingly usable for 7B-13B models on modern machines
- Consumer GPUs: RTX cards, Apple M-series, AMD
- Dedicated inference hardware: Groq, Cerebras, etc. (not "local" but relevant)
- Memory is often the bottleneck, not compute

### Deployment patterns
- Model distribution (weights are 4-80 GB — this is a real problem)
- Cold start (loading weights into memory)
- Concurrency on a single GPU
- Serverless GPU inference

### The local agent use case
- Privacy-preserving assistants
- Offline capability
- Embedded in apps, IDEs, OS
- Tension with frontier model capabilities

## Key gotchas

- Quantization is lossy — a 4-bit quantized 13B model is not as good as the full-precision 13B
- GGUF model files are large; distribution is a real problem
- "Works on my laptop" doesn't generalize — hardware heterogeneity is worse than in the cloud
- Local models lag frontier by 6-18 months in capability
- Windows local inference is less polished than Linux; CUDA on Windows has quirks
- Cold-start latency: loading a 7B model takes seconds, 70B takes a minute
- Battery impact on laptops is non-trivial

## What a PE needs to be credible on

- Can evaluate whether local inference is viable for a given use case
- Can compare runtimes and recommend one
- Knows the quantization landscape and its trade-offs
- Can design a hybrid architecture (local + cloud) that's robust to each's failure modes
- Understands the hardware constraints well enough to have opinions about GPU specs
