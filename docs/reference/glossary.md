# Glossary

Terms a PE should know and be able to use correctly. Organized roughly by category.

## Models and training

- **LLM (Large Language Model)** — a neural network trained on vast text corpora to predict the next token

- **Parameters** — the learned weights of a model; "70B parameters" means ~70 billion of them

- **Pretraining** — the initial, compute-heavy stage of training on broad text

- **Fine-tuning** — continuing to train on a specific dataset to specialize the model

- **SFT (Supervised Fine-Tuning)** — fine-tuning on input-output pairs, the first post-pretraining stage

- **RLHF (Reinforcement Learning from Human Feedback)** — training a model to prefer outputs humans rate higher

- **DPO (Direct Preference Optimization)** — a simpler alternative to RLHF

- **Distillation** — training a smaller model to imitate a larger one

- **Foundation model / Base model** — the pretrained model before instruction-tuning or RLHF

- **Instruction-tuned** — a model post-SFT, able to follow instructions

- **Chat-tuned / Assistant model** — a model further tuned for conversational interaction

- **Frontier model** — the current most-capable class (Claude 4, GPT-5, Gemini 2.5, etc.)

## Inference

- **Token** — sub-word unit the model operates on; a word can be 1-4 tokens typically

- **Tokenizer** — the algorithm that splits text into tokens; varies by model family

- **Context window** — the maximum number of tokens the model can attend to at once

- **Prefill** — the first pass of computation that processes the input prompt

- **Decoding / Generation** — generating output tokens one at a time

- **Temperature** — sampling parameter controlling randomness (0 = deterministic, >1 = more random)

- **Top-p / Nucleus sampling** — sample from the smallest set of tokens whose probabilities sum to p

- **Top-k sampling** — sample from only the top k most likely tokens

- **Greedy decoding** — always pick the most likely next token (temperature 0)

- **Streaming** — sending tokens to the client as they're generated

- **Prompt caching** — storing computation from a common prefix so it can be reused

- **Speculative decoding** — using a smaller model to guess ahead, verified by the large model

- **Batching** — processing multiple requests together for throughput

- **KV cache** — intermediate computation stored during generation, freed at the end

## Quantization

- **Quantization** — reducing the precision of model weights to save memory and compute

- **FP32 / FP16 / BF16** — full / half / brain-float precision

- **INT8 / INT4** — 8-bit / 4-bit integer quantization

- **GGUF** — file format used by llama.cpp for quantized models

- **AWQ, GPTQ** — specific quantization algorithms

- **Activation quantization** — quantizing intermediate values during inference, not just weights

## Prompting and context

- **Prompt** — the input text given to the model

- **System prompt** — the prompt that establishes role, constraints, and style

- **User prompt / User message** — the per-turn input from the user

- **Context** — everything in the input: system prompt + history + retrieved docs + tool output + user message

- **Few-shot / Few-shot prompting** — including examples in the prompt to show the model what you want

- **Zero-shot** — giving only instructions, no examples

- **Chain-of-thought (CoT)** — asking the model to reason step-by-step before answering

- **Self-consistency** — sampling multiple chains of thought and taking the majority answer

- **Structured output** — output constrained to a schema (JSON, function call format)

- **Tool use / Function calling** — model generates a structured request to call an external function

## RAG

- **RAG (Retrieval-Augmented Generation)** — retrieving relevant documents and injecting them into the prompt

- **Embedding** — a dense vector representation of text, used for similarity search

- **Vector store / Vector DB** — a database specialized for nearest-neighbor search on embeddings

- **Chunking** — splitting documents into smaller pieces for indexing

- **Re-ranking** — a second pass that reorders retrieved results with a more precise model

- **BM25** — a classical sparse retrieval algorithm (keyword-based)

- **Hybrid search** — combining dense (vector) and sparse (keyword) retrieval

- **Cross-encoder** — a model that takes two texts and scores their relevance together (used for re-ranking)

- **Bi-encoder** — a model that encodes texts independently (used for first-pass retrieval)

## Agents

- **Agent** — a system where the LLM decides actions to take, with tool use and loops

- **Workflow** — a system with predetermined steps, sometimes with LLM calls at nodes

- **ReAct** — Reasoning + Acting, an agent pattern alternating between reasoning and tool use

- **MCP (Model Context Protocol)** — an open protocol for connecting LLMs to tools and data

- **Tool** — a function the model can invoke, with a schema

- **Tool schema** — the structured description of a tool (name, args, return type)

- **Planner** — an agent component that decomposes goals into steps

- **Executor** — an agent component that carries out individual steps

- **Memory** — persistent state associated with an agent (short-term, long-term)

- **Multi-agent** — multiple LLM-driven components coordinating

## Evaluation

- **Eval / Evaluation** — measuring the quality of model outputs

- **Golden set / Test set** — human-labeled examples of correct outputs

- **LLM-as-judge** — using an LLM to evaluate another LLM's output

- **Groundedness** — whether an output is supported by provided sources

- **Faithfulness** — similar; whether the output accurately reflects the source

- **Hallucination** — model output that's confidently false

- **Benchmark** — a standardized test suite (MMLU, HumanEval, GSM8K, etc.)

- **Regression** — a quality drop introduced by a change

## Safety and security

- **Alignment** — making models pursue the goals their designers intend

- **Helpful, Harmless, Honest (HHH)** — an Anthropic-ish framing of model goals

- **Refusal** — when the model declines to respond (trained behavior)

- **Jailbreak** — a prompt that circumvents the model's trained refusals

- **Prompt injection** — an attack where untrusted input contains instructions that hijack the prompt

- **Indirect prompt injection** — injection via content the model retrieves or tools return

- **Red-teaming** — adversarial testing to find safety failures

- **Guardrails** — input/output filters and policies around the model

- **Constitutional AI** — Anthropic's approach to training with a list of principles

- **Sandbox** — a constrained environment where risky operations can be executed safely

## Infrastructure

- **Inference API** — the HTTP API for calling a model (OpenAI, Anthropic, etc.)

- **Hosted / Managed inference** — someone else runs the model, you call their API

- **Self-hosted** — you run the model on your own hardware

- **Open-weights / Open-source model** — a model whose weights are publicly available (licenses vary)

- **TPM (Tokens Per Minute)** — rate limit on provider APIs

- **RPM (Requests Per Minute)** — rate limit on provider APIs

- **Batch API** — a cheaper, slower path provided by some vendors for bulk processing

## The acronym cheat sheet

- AI — Artificial Intelligence

- AGI — Artificial General Intelligence (often marketing)

- CoT — Chain of Thought

- DPO — Direct Preference Optimization

- GGUF — GPT-Generated Unified Format (llama.cpp quantized format)

- HHH — Helpful, Harmless, Honest

- LLM — Large Language Model

- MCP — Model Context Protocol

- MoE — Mixture of Experts (model architecture)

- NLP — Natural Language Processing

- PII — Personally Identifiable Information

- RAG — Retrieval-Augmented Generation

- ReAct — Reasoning + Acting

- RLHF — Reinforcement Learning from Human Feedback

- SFT — Supervised Fine-Tuning

- SOTA — State of the Art

- TPM / RPM — Tokens / Requests Per Minute
