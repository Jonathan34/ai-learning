# Tools and Models Quick Reference

**Warning:** this ages fast. Verify anything specific before relying on it.

## Frontier hosted models (late 2025)

| Model | Provider | Good at | Notes |
|---|---|---|---|
| Claude 4 family (Opus, Sonnet, Haiku) | Anthropic | Reasoning, coding, long context, safety | Strong default for production |
| GPT-4o / o3 | OpenAI | Broad capability, tool use, reasoning (o3) | Widely available |
| Gemini 2.5 Pro / Flash | Google | Long context, multimodal | Strong at images, 1M+ context |
| Grok | xAI | Less aligned; use case specific | Less mature ecosystem |

## Open-weights models worth knowing

| Model family | Sizes | Notes |
|---|---|---|
| Llama 3.x | 1B-405B | Meta; most widely deployed open model |
| Qwen 2.5 / 3 | 0.5B-72B | Alibaba; strong multilingual |
| Mistral / Mixtral | 7B-8x22B | French lab; MoE variants |
| DeepSeek | Various | Strong reasoning models |
| Nemotron | Various | NVIDIA's family; heavily optimized for NVIDIA hardware |
| Gemma | 2B-27B | Google's open family |
| Phi | 3-14B | Microsoft; small and capable |

## Local runtimes

| Runtime | Best for | Notes |
|---|---|---|
| Ollama | Developer UX, easy model management | Wraps llama.cpp |
| llama.cpp | CPU + GPU flexibility, GGUF format | The foundation most others build on |
| vLLM | GPU server-side throughput | Best for production self-hosting |
| TensorRT-LLM | NVIDIA GPU perf | Best perf on NVIDIA hardware |
| MLC-LLM | Cross-platform, mobile | Broader hardware reach |
| MLX | Apple silicon | Apple-native, fast on M-series |
| ONNX Runtime | Cross-runtime, traditional ML shops | Less LLM-specific |

## Vector databases

| DB | Best for | Notes |
|---|---|---|
| Chroma | Local dev, simple cases | Easy to start |
| Qdrant | Production self-hosted | Rust; fast, feature-rich |
| Weaviate | Production hosted or self | Has hybrid search |
| Pinecone | Managed, hassle-free | Paid; good SLAs |
| pgvector | When you already have Postgres | Often the right answer |
| LanceDB | Embedded, multimodal | Interesting newer entrant |
| Milvus | Large scale self-hosted | Kubernetes-native |

## Embedding models

| Model | Notes |
|---|---|
| OpenAI text-embedding-3-small / large | Widely used, paid |
| Cohere embed-v3 | Strong multilingual |
| Voyage AI | Purpose-built embedding lab |
| BGE (open) | Strong open baseline |
| sentence-transformers (open) | Classic, many variants |

## Agent frameworks

| Framework | Best for | Notes |
|---|---|---|
| Claude Agent SDK | Claude-native agents | Well-integrated with tool use |
| LangGraph | Stateful graph-based agents | More principled than LangChain |
| LangChain | Rapid prototyping, broad ecosystem | Abstractions sometimes fight you |
| AutoGen | Multi-agent patterns | Microsoft; actor-model-ish |
| CrewAI | Role-based multi-agent | Easy to start |
| OpenAI Agents SDK | OpenAI-native | Ties to Assistants API |
| Mastra | Newer TypeScript-native | Gaining traction |
| DSPy | Compiled prompts, research-flavored | Different philosophy |

## Observability / LLMOps

| Tool | Notes |
|---|---|
| Langfuse | Open-source, widely used |
| Braintrust | Hosted, strong eval focus |
| Helicone | Logs and cost tracking |
| LangSmith | LangChain's companion |
| Weave (W&B) | Traditional ML observability + LLM |
| Arize Phoenix | Open-source, strong evals |

## Eval frameworks

| Tool | Notes |
|---|---|
| Promptfoo | CLI-first, simple, open-source |
| Braintrust (above) | Full-featured, hosted |
| OpenAI Evals | Reference implementation |
| Inspect (UK AI Safety Institute) | Research-grade evals |
| LangSmith evals | Tied to LangSmith |

## MCP servers worth knowing

| Server | Notes |
|---|---|
| filesystem | Reference MCP server for file access |
| fetch | HTTP fetching |
| github | Git repo operations |
| slack | Slack integration |
| postgres | SQL access |
| Many more emerging | Check modelcontextprotocol.io |

## Where to find updates

- HuggingFace (models, daily papers)

- modelcontextprotocol.io (MCP ecosystem)

- The lab blogs directly (Anthropic, OpenAI, Google DeepMind, Meta AI)

- GitHub trending for AI/ML repos

- arxiv-sanity, Papers With Code

## Cost ballpark (late 2025, USD)

Prices change. This is a rough order-of-magnitude for thinking.

- Frontier hosted output tokens: ~$1-$15 per million tokens

- Frontier hosted input tokens: ~$0.25-$5 per million tokens

- Open-weights self-hosted on GPUs: ~$0.05-$2 per million output tokens depending on model size and utilization

- Embedding: ~$0.02-$0.15 per million tokens

- Fine-tuning: varies widely; budget $100-$10k for a reasonable-scale fine-tune
