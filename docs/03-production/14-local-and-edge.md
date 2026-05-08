# Local and Edge Inference

Running LLMs on your own hardware — including consumer laptops — has become practical through quantization (compressing model weights) and efficient runtimes. It's a different design space from hosted inference, with real advantages and real constraints.

This chapter is especially relevant if you're interested in roles at companies like NVIDIA that focus on making AI run efficiently on local hardware.

## Why run locally

- **Privacy.** Data never leaves the device. No third-party provider sees your prompts.

- **Offline capability.** Works without internet. Useful for field work, air-gapped environments, or unreliable connectivity.

- **Cost at scale.** No per-token charges. Once you have the hardware, inference is "free" (minus electricity and maintenance).

- **Latency.** No network round trip. For small models, local inference can be faster than a hosted API call.

- **Customization.** You can fine-tune, quantize, and modify the model however you want.

## Why not run locally

- **Model quality.** The best open-weights models lag frontier hosted models (Claude, GPT-5) by 6-18 months in capability.

- **Hardware requirements.** Large models need expensive GPUs or lots of RAM.

- **Operational burden.** You manage updates, compatibility, hardware failures.

- **No prompt caching.** You don't get the provider-side optimizations that hosted APIs offer.

## The runtime landscape

These are the main tools for running models locally:

| Runtime | What it is | Best for |
|---|---|---|
| **Ollama** | Wraps llama.cpp with model management and an API | Developer experience, easy model switching |
| **llama.cpp** | C++ inference engine, runs on CPU and GPU | Maximum flexibility, broad hardware support |
| **vLLM** | Python-based, GPU-optimized for throughput | Server-side self-hosting at scale |
| **TensorRT-LLM** | NVIDIA's optimized runtime | Best performance on NVIDIA GPUs |
| **MLX** | Apple's framework for Apple silicon | Best performance on M-series Macs |
| **MLC-LLM** | Cross-platform including mobile | Broadest hardware reach |

If you're just getting started: use Ollama. It handles model downloads, quantization selection, and gives you an API with one command. [Workshop W1](../05-workshops/W1-local-llm-setup.md) walks through this.

If you need production performance on NVIDIA hardware: TensorRT-LLM. On Apple hardware: MLX. For server-side self-hosting: vLLM.

## Quantization: making models fit

Frontier models have billions of parameters (learned numerical weights). In full precision, each parameter is a 32-bit or 16-bit floating-point number. A 70-billion-parameter model at 16-bit precision needs ~140 GB of memory. That doesn't fit on any consumer hardware.

Quantization compresses these numbers. Instead of 16-bit floats, you store them as 8-bit integers, 4-bit integers, or even lower. This trades a small amount of quality for a massive reduction in size and memory use.

| Quantization | Size of 7B model | Quality impact | Memory needed |
|---|---|---|---|
| FP16 (full) | ~14 GB | None (baseline) | ~16 GB |
| INT8 (8-bit) | ~7 GB | Minimal | ~8 GB |
| Q4 (4-bit) | ~4 GB | Small but measurable | ~5 GB |
| Q2 (2-bit) | ~2 GB | Significant | ~3 GB |

The sweet spot for most use cases is 4-bit quantization (Q4). It's roughly 4x smaller than full precision with only a small quality loss on most tasks. 8-bit is better quality but larger. 2-bit is too lossy for most purposes.

**GGUF** is the file format used by llama.cpp (and therefore Ollama) for quantized models. When you see a model file like `Llama-3.2-3B-Instruct-Q4_K_M.gguf`, the `Q4_K_M` part tells you it's 4-bit quantization with a specific compression algorithm.

## Hardware: what actually limits you

**Memory is usually the bottleneck, not compute.** The entire model needs to fit in memory (GPU memory for GPU inference, system RAM for CPU inference). A 13B model at Q4 needs ~8 GB. A 70B model at Q4 needs ~40 GB. Most consumer laptops have 16-32 GB of RAM.

**GPU vs CPU:**

- GPU inference is much faster (10-50x) but requires the model to fit in GPU memory (VRAM)

- CPU inference works for smaller models (up to ~13B on most machines) but is slower

- Apple M-series chips use unified memory — the GPU and CPU share the same RAM pool, which makes them unusually good for local inference

- NVIDIA RTX cards have 8-24 GB of VRAM depending on the model

**Practical limits on a typical machine:**

| Hardware | Comfortable model size |
|---|---|
| 8 GB RAM laptop | 3B-7B at Q4 |
| 16 GB RAM laptop | 7B-13B at Q4 |
| 32 GB RAM (M-series Mac) | 13B-30B at Q4 |
| RTX 4090 (24 GB VRAM) | 13B-30B at Q4 |
| 64 GB RAM (M-series Mac) | 70B at Q4 |

## Model distribution

Model files are large. A 7B model at Q4 is ~4 GB. A 70B model is ~40 GB. Distributing these to end users is a real engineering problem:

- Download time on slow connections

- Storage space on user devices

- Updates when new model versions release

- Multiple models for different tasks

Ollama handles this with a Docker-like pull model (`ollama pull llama3.2:3b`). For custom applications, you'll need to solve distribution yourself — CDN hosting, delta updates, or bundling with the application.

## Cold start

Loading a model into memory takes time:

- Small models (3B-7B): 1-5 seconds

- Medium models (13B-30B): 5-15 seconds

- Large models (70B+): 30-60+ seconds

For always-on applications, keep the model loaded in memory. For on-demand applications, users will experience the cold start delay on first use. Some systems pre-load models on boot or keep them warm with periodic dummy requests.

## The local agent use case

Local models powering agents on your device:

- Privacy-preserving personal assistants

- IDE coding assistants (Copilot alternatives)

- Offline-capable tools

- OS-level AI features (what NVIDIA and Apple are building toward)

The tension: local models are less capable than frontier hosted models. An agent that works well with Claude Sonnet might fail with a local 7B model. You often need to simplify the agent (fewer tools, simpler tasks, more constrained prompts) to work reliably with smaller models.

## Things that trip people up

**"Works on my laptop" doesn't generalize.** Hardware varies enormously. A model that runs smoothly on your 32 GB M3 Max won't run at all on a user's 8 GB Windows laptop. Test on representative hardware.

**Quantization is lossy.** A 4-bit quantized 13B model is not as good as the full-precision 13B. The loss is usually small for straightforward tasks but can be significant for reasoning, math, or following complex instructions. Always evaluate quantized models on your specific task.

**Windows local inference has more quirks than Linux/Mac.** CUDA on Windows, driver compatibility, path issues. If you're targeting Windows users, test thoroughly on Windows.

**Battery impact on laptops.** Running inference continuously drains battery fast. For mobile/laptop applications, consider when to run locally vs. when to fall back to a hosted API.

**Model licenses vary.** Llama has a license that restricts some commercial uses. Mistral models are more permissive. Some models are research-only. Check the license before shipping a product with an embedded model.

**Cold start latency.** Users don't expect to wait 10 seconds before an AI feature responds. Either keep models warm or set expectations in the UI.

## Where things stand

Local inference is practical today for models up to ~30B parameters on high-end consumer hardware, and up to ~13B on typical laptops. Quality is good enough for many tasks (summarization, classification, simple Q&A, code completion) but still lags frontier models on complex reasoning.

The gap is closing. New models are more efficient. Hardware is getting better. Quantization techniques are improving. In 2-3 years, running a model locally that matches today's frontier quality will likely be routine.

For now: use local inference when privacy, offline capability, or cost at scale demands it. Use hosted inference when you need the best quality or can't guarantee hardware.

## Go deeper

- [Workshop W1 — Local LLM Setup (hands-on with Ollama and llama](../05-workshops/W1-local-llm-setup.md).cpp)

- [Ollama](https://ollama.com) — easiest way to get started

- [llama.cpp](https://github.com/ggerganov/llama.cpp) — the foundation most local runtimes build on

- [HuggingFace model hub](https://huggingface.co/models) — where quantized models are published

- [TensorRT-LLM](https://github.com/NVIDIA/TensorRT-LLM) — NVIDIA's optimized runtime
