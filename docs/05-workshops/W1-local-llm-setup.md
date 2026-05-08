# Workshop W1 — Local LLM Setup

**Goal:** have an LLM running on your laptop, understand the stack well enough to reason about it, and compare at least 3 models hands-on.

**Time:** 2-3 hours the first time, faster after.

**You'll learn:** how local inference actually works, quantization in practice, the feel of different model sizes, and the limits of what a laptop can run.

---

## Why run models locally

You can read ten blog posts about Ollama vs llama.cpp vs vLLM and still have no intuition for what local inference feels like. Running models yourself gives you something reading can't: you'll know which models are actually usable, how fast they are on your hardware, how big their files are, and how much memory they eat.

---

## Part 1: Install Ollama (30 min)

Ollama wraps llama.cpp (the most popular C++ inference engine for running models on consumer hardware) with model management and an HTTP API. It's the most approachable starting point.

```bash
# macOS
brew install ollama
# or download from ollama.com

# Start the server
ollama serve
```

Pull your first model. Start small:

```bash
ollama pull llama3.2:3b
```

This downloads a 2 GB quantized model. Quantization means the model's numbers have been compressed from their original precision (32-bit floating point) down to 4-bit integers — trading a small amount of quality for a massive reduction in file size and memory use. The original Llama 3.2 3B in full precision is closer to 6 GB.

Run it:

```bash
ollama run llama3.2:3b
```

You're now talking to a language model running entirely on your laptop. Ask it something. Note the generation speed.

What to observe:
- Time to first token (should be ~100ms on a fast CPU, longer on first load)
- Tokens per second (write it down — this is your baseline)
- Does it fit in memory? Check Activity Monitor / Task Manager.

---

## Part 2: Compare models across sizes (30 min)

Pull three more models covering a range:

```bash
ollama pull llama3.2:1b       # smaller
ollama pull llama3.1:8b       # medium
ollama pull qwen2.5:14b       # larger, different model family
```

The 14B model needs 8+ GB of memory. If you don't have that, skip it and try `mistral:7b` instead.

Ask each model the same prompts. Things to compare:

1. **Tokens per second** — smaller models are faster
2. **Reasoning** — e.g., "I have 3 apples. I eat 1 and give 2 to my friend. How many do I have?"
3. **Domain knowledge** — pick something from your work
4. **Code** — e.g., "Write a Python function that checks if a string is a palindrome, handling whitespace and case."

You'll notice patterns. Small models hallucinate more. Large models are slower but more reliable. Different model families have different "personalities." Instruction-following varies a lot.

Write down what you observed.

---

## Part 3: Inspect the quantization (20 min)

```bash
ollama show llama3.2:3b --modelfile
```

You'll see the parameter file. Notice the model is `Q4_K_M` or similar — that's 4-bit quantization with a specific compression scheme. The letters and numbers encode how aggressively the model was compressed and which algorithm was used.

Find where Ollama stored the actual model file:

```bash
# macOS
ls -lh ~/.ollama/models/blobs/
```

Look at the file sizes. These are GGUF files — llama.cpp's binary format for storing quantized models. This is what's actually loaded into memory at runtime.

Try a different quantization level:

```bash
ollama pull llama3.2:3b-instruct-q8_0    # 8-bit quantization
```

Compare quality and speed against the default (Q4). You'll see the quality-vs-size trade-off directly. The 8-bit version is larger and slightly better; the 4-bit version is smaller and slightly worse. For most tasks, the difference is subtle.

---

## Part 4: Use it from code (30 min)

Ollama exposes an OpenAI-compatible API. This means any code written for the OpenAI API works with your local model — just change the URL.

Start the server if it's not already running:

```bash
ollama serve
```

In Python:

```python
import openai

client = openai.OpenAI(
    base_url="http://localhost:11434/v1",
    api_key="ollama"  # required by the library but unused locally
)

response = client.chat.completions.create(
    model="llama3.2:3b",
    messages=[
        {"role": "system", "content": "You are concise."},
        {"role": "user", "content": "Explain RAG in two sentences."}
    ]
)
print(response.choices[0].message.content)
```

Now you have a local LLM reachable from code. Anything you can build against OpenAI or Anthropic, you can prototype locally for free.

---

## Part 5: Try llama.cpp directly (30 min, optional but recommended)

Ollama is a wrapper. Going one level down gives you real understanding of what's happening underneath.

```bash
# macOS via Homebrew
brew install llama.cpp
```

Download a GGUF model file directly from HuggingFace (a platform where people publish model files):

```bash
# Example: a small Llama 3.2 model
# Check huggingface.co for current URLs
wget https://huggingface.co/bartowski/Llama-3.2-3B-Instruct-GGUF/resolve/main/Llama-3.2-3B-Instruct-Q4_K_M.gguf
```

Run it:

```bash
llama-cli -m Llama-3.2-3B-Instruct-Q4_K_M.gguf -p "Explain RAG in two sentences." -n 200
```

Now you've seen the naked inference path. llama.cpp has knobs for everything — context length, temperature, sampling strategy, batching. Run `llama-cli --help` once just to see the scope.

---

## Part 6: Reflect (15 min)

Answer these for yourself:

1. What's the smallest model that's useful for your day-to-day work?
2. How does quality change as you go from 3B → 8B → larger?
3. How does your laptop's hardware limit what you can run? (GPU memory, system memory)
4. If you were designing a product with a "works offline" requirement, which model would you pick and why?
5. What did the 4-bit quantization cost you, quality-wise? Was it a meaningful loss for your use case?

Writing down answers — even brief ones — cements the learning.

---

## Gotchas

- **Ollama defaults to a short context window** (2048 tokens). If your prompt is long, the model silently truncates. Set `num_ctx` in the Modelfile or API call to increase it.
- **Quantized models lose capability unevenly.** Small models quantized aggressively can become borderline useless on reasoning tasks while still being fine for classification or summarization.
- **CPU vs GPU matters a lot.** M-series Macs and RTX cards both accelerate local inference, but through different paths. Check that your runtime is using the right backend.
- **Model licenses vary.** Llama has restrictions, Mistral has permissive licensing, some models are research-only. Check before using commercially.
- **Memory is often the real limit.** A 13B model at Q4 needs ~8 GB of memory. Add a few GB for context and overhead. A 16 GB laptop can't run 30B+ models comfortably.

---

## What you should have after this workshop

- Ollama installed with 3+ models pulled
- Understanding of quantization trade-offs from direct observation
- A Python script that calls a local model successfully
- A sense of what "works locally" actually means — and what doesn't

Next workshop builds on this. You can keep using your local model.

## Go deeper

- [Ollama documentation](https://ollama.com)
- [llama.cpp GitHub repository](https://github.com/ggerganov/llama.cpp)
- [HuggingFace model hub](https://huggingface.co/models) — where most GGUF files are published
- [TheBloke's quantization guide](https://huggingface.co/TheBloke) — explains the different quantization levels
