# Tools and Models Quick Reference

**Warning:** this ages fast. Verify anything specific before relying on it.

## Frontier hosted models (October 2026)

| Model | Provider | Good at | Notes |
|---|---|---|---|
| GPT-6 family (Astra, 6.1 Sol, Luna) | OpenAI | Coding, tool use, knowledge work, multimodal tasks | OpenAI positions Astra as its most capable model; 6.1 Sol balances capability and cost; Luna is the fastest and cheapest ([changelog](https://developers.openai.com/api/docs/changelog), [latest-model guide](https://developers.openai.com/api/docs/guides/latest-model/gpt-5.2), checked 2026-10-05) |
| Claude 5 family (Fable 5.1, Opus 5, Sonnet 5; Haiku 4.5) | Anthropic | Agentic coding, enterprise work, long context, balanced production workloads | Anthropic recommends Opus 5 for most workloads and Fable 5.1 for the hardest reasoning and long-horizon agent work; Fable, Opus, and Sonnet have 1M-token context, Haiku 4.5 has 200K ([model catalog](https://platform.claude.com/docs/en/models/overview), checked 2026-10-05) |
| Gemini 3 family (3.8 Flash stable; 3.1 Pro preview) | Google | Multimodal work, coding, agents, low-latency workloads | Production status varies by model; Google recommends 3.8 Flash or 3.5 Flash-Lite over 2.5 for new projects ([model catalog](https://ai.google.dev/gemini-api/docs/models), checked 2026-10-05) |
| Grok 4.7 | xAI | Coding, tool use, general-purpose reasoning | Current general-purpose flagship; real-time information requires search tools ([model catalog](https://docs.x.ai/docs/models?cluster=us-west-1), checked 2026-10-03) |

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
| OpenAI Agents SDK | OpenAI-native | Higher-level runtime over the Responses API; tools, handoffs, guardrails, tracing, sessions ([docs](https://openai.github.io/openai-agents-python/), checked 2026-09-28) |
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
