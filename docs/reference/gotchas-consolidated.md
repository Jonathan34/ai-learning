# Consolidated Gotchas

The things experienced AI engineers have learned the hard way, in one list. Cross-referenced to the chapter where each is explained in context.

## On how LLMs work

- **"The model knows X"** — it doesn't. It has statistical associations. Never rely on it as a factual database without retrieval. ([Ch 01](../01-foundations/01-llm-mental-model.md))

- **Token ≠ word** — token counts vary wildly by language and tokenizer. Budget with the actual tokenizer. ([Ch 01](../01-foundations/01-llm-mental-model.md))

- **Non-determinism by default** — even temperature 0 has some non-determinism in practice. Design for it. ([Ch 01](../01-foundations/01-llm-mental-model.md))

- **Context ≠ memory** — the model has no memory across calls. Memory is a system you build. ([Ch 01](../01-foundations/01-llm-mental-model.md))

## On prompting

- **Prompts are coupled to model versions** — pin versions, run eval on upgrades. ([Ch 02](../01-foundations/02-prompting-as-programming.md))

- **System prompts are not sandboxes** — don't put secrets there; they can be extracted. ([Ch 02](../01-foundations/02-prompting-as-programming.md))

- **Instruction-following is soft** — 99% success rate looks great and fails 1% in production. Eval or accept. ([Ch 02](../01-foundations/02-prompting-as-programming.md))

- **Verbose prompts aren't always better** — too long and the model loses track. ([Ch 02](../01-foundations/02-prompting-as-programming.md))

- **"Act as an expert" does little** on modern instruction-tuned models. ([Ch 02](../01-foundations/02-prompting-as-programming.md))

## On context engineering

- **Naive RAG feels magical in demos, fails in production** — chunking, re-ranking, hybrid search matter more than they seem. ([Ch 03](../01-foundations/03-context-engineering.md))

- **Lost in the middle** — put critical info near the end of long contexts. ([Ch 03](../01-foundations/03-context-engineering.md))

- **Embedding drift** — changing models means re-embedding everything. ([Ch 03](../01-foundations/03-context-engineering.md))

- **PII in context flows to the model provider** — know your data governance. ([Ch 03](../01-foundations/03-context-engineering.md))

- **RAG without "say I don't know" guardrails** will happily fabricate. ([Ch 03](../01-foundations/03-context-engineering.md))

- **Context window != budget to dump everything in** — assemble deliberately. ([Ch 03](../01-foundations/03-context-engineering.md))

## On evaluation

- **"We'll eval it once we have users"** — no. Build eval before features. ([Ch 04](../01-foundations/04-evaluation.md))

- **LLM-as-judge with the same model you're evaluating** — calibration failure. ([Ch 04](../01-foundations/04-evaluation.md))

- **Public benchmarks are often in training data** — don't trust SOTA claims without caveats. ([Ch 04](../01-foundations/04-evaluation.md))

- **The demo works ≠ the feature works** — distribution testing, not point testing. ([Ch 04](../01-foundations/04-evaluation.md))

- **Eval sets drift** — your golden set ages. Refresh periodically. ([Ch 04](../01-foundations/04-evaluation.md))

## On security and safety

- **Refusals are trained, not hard-coded** — they can be bypassed. Defense in depth. ([Ch 05](../01-foundations/05-security-and-safety.md))

- **Indirect prompt injection via tool outputs** is the insidious one. A fetched web page can hijack your agent. ([Ch 05](../01-foundations/05-security-and-safety.md), [Ch 09](../02-agents/09-mcp-and-tools.md))

- **"The model will refuse" is not a security control** — don't rely on it. ([Ch 05](../01-foundations/05-security-and-safety.md))

- **Sandboxing tools isn't optional** — if your agent has tools, they need sandboxes. ([Ch 05](../01-foundations/05-security-and-safety.md), [Ch 09](../02-agents/09-mcp-and-tools.md))

- **User input should never concatenate into instructions** — always treat as data with delimiters. ([Ch 05](../01-foundations/05-security-and-safety.md))

- **Capability creep for agents** — start with least privilege. ([Ch 05](../01-foundations/05-security-and-safety.md))

## On agents

- **"Agent" is overloaded** — classify where on the spectrum, argue for simpler when possible. ([Ch 07](../02-agents/07-what-is-an-agent.md))

- **Loop conditions matter** — agents can run forever without a stop. Set budgets. ([Ch 07](../02-agents/07-what-is-an-agent.md))

- **Hallucinated tool calls** — model invents tool names or arguments. Validate. ([Ch 07](../02-agents/07-what-is-an-agent.md), [Ch 09](../02-agents/09-mcp-and-tools.md))

- **Agent cost multiplies steps** — a 10-step agent costs ~10x a one-shot call. ([Ch 07](../02-agents/07-what-is-an-agent.md))

- **Plan drift** — the agent's plan and behavior diverge mid-run. Observe, don't trust. ([Ch 07](../02-agents/07-what-is-an-agent.md))

## On frameworks

- **Frameworks hide what the model sees** — always be able to log the actual prompt. ([Ch 08](../02-agents/08-framework-landscape.md))

- **Frameworks change fast** — pin versions, expect breakage. ([Ch 08](../02-agents/08-framework-landscape.md))

- **Framework abstractions aren't always clean** — what looks simple may be 5 LLM calls. ([Ch 08](../02-agents/08-framework-landscape.md))

## On MCP and tools

- **Tool descriptions ARE prompts** — write them as carefully as any other part of the prompt. ([Ch 09](../02-agents/09-mcp-and-tools.md))

- **Tool outputs too long blow context** — truncate or summarize. ([Ch 09](../02-agents/09-mcp-and-tools.md))

- **MCP has no default security** — add auth, rate limits, capability constraints yourself. ([Ch 09](../02-agents/09-mcp-and-tools.md))

## On multi-agent

- **Multi-agent multiplies cost 5-10x** — make sure the benefit is real. ([Ch 10](../02-agents/10-multi-agent-patterns.md))

- **Agents don't converge, they loop** — "let them negotiate" is an anti-pattern. ([Ch 10](../02-agents/10-multi-agent-patterns.md))

- **Debugging multi-agent requires serious observability** — from the start. ([Ch 10](../02-agents/10-multi-agent-patterns.md))

## On state and memory

- **Unbounded memory storage** — inserting everything forever is a cost and privacy disaster. ([Ch 11](../02-agents/11-state-memory-durability.md))

- **Memory poisoning** — early false facts persist and compound. ([Ch 11](../02-agents/11-state-memory-durability.md))

- **Stale memory vs. fresh observation conflicts** — have a resolution strategy. ([Ch 11](../02-agents/11-state-memory-durability.md))

- **PII in memory has retention implications** — GDPR, CCPA, etc. ([Ch 11](../02-agents/11-state-memory-durability.md))

## On inference economics

- **Output tokens dominate cost for generation-heavy workloads** — but for agents with large contexts, input tokens often dominate because you re-send the full context each turn. Know which applies to your system. ([Ch 12](../03-production/12-inference-economics.md))

- **Tail latency kills UX** — p99 matters more than p50 for interactive AI. ([Ch 12](../03-production/12-inference-economics.md))

- **Capacity is finite** — provider rate limits are real at scale. ([Ch 12](../03-production/12-inference-economics.md))

- **Agent loops multiply cost** — plan and budget accordingly. ([Ch 12](../03-production/12-inference-economics.md))

## On observability

- **Logging "prompt template" ≠ logging "actual prompt"** — log what the model literally saw. ([Ch 13](../03-production/13-observability.md))

- **Full-context logs have PII and cost** — sample and redact intelligently. ([Ch 13](../03-production/13-observability.md))

- **Dashboards without alerts** are decorative. ([Ch 13](../03-production/13-observability.md))

## On deployment

- **"Just use OpenAI" has real governance implications** at scale. ([Ch 14](../03-production/14-deployment-patterns.md))

- **Self-hosting is harder than the blog posts make it sound** — ops, capacity, versioning. ([Ch 14](../03-production/14-deployment-patterns.md))

- **Multi-provider fallback is quality risk** — models differ more than providers admit. ([Ch 14](../03-production/14-deployment-patterns.md))

## On local inference

- **Quantization is lossy** — evaluate, don't assume parity. ([Ch 15](../03-production/15-local-and-edge.md))

- **Local models still lag frontier** — gap narrowing (3-6 months on many tasks as of 2025) but real for complex reasoning. ([Ch 15](../03-production/15-local-and-edge.md))

- **Memory is the real limit** — not just compute. ([Ch 15](../03-production/15-local-and-edge.md))

- **Windows local inference has quirks** — more than Linux. ([Ch 15](../03-production/15-local-and-edge.md))

## On platform

- **Building a platform too early** — solves problems no one has. ([Ch 16](../03-production/16-ai-platform-engineering.md))

- **No platform at all** — every team rebuilds, inconsistently. ([Ch 16](../03-production/16-ai-platform-engineering.md))

- **Ignoring cost attribution** — surprise six-figure bills. ([Ch 16](../03-production/16-ai-platform-engineering.md))

## On product

- **Shipping a chatbot by default** — often the wrong UX. ([Ch 18](../04-leadership/18-ai-product-sense.md))

- **Not designing for failure** — fragile launches. ([Ch 18](../04-leadership/18-ai-product-sense.md))

- **Over-promising** via model responses → trust loss. ([Ch 18](../04-leadership/18-ai-product-sense.md))

## On org

- **Over-hiring AI specialists** when applied engineers would do. ([Ch 19](../04-leadership/19-team-and-org-patterns.md))

- **Under-investing in platform** — "just build features" doesn't scale. ([Ch 19](../04-leadership/19-team-and-org-patterns.md))

- **Letting research disconnect from product** — papers not products. ([Ch 19](../04-leadership/19-team-and-org-patterns.md))

## On learning

- **Firehose anxiety** — you can't keep up with everything. Filter aggressively. ([Ch 20](../04-leadership/20-staying-current.md))

- **Hype amplification on social** — researchers are better signals than influencers. ([Ch 20](../04-leadership/20-staying-current.md))

- **Benchmark drama** — new SOTA is rarely actionable. ([Ch 20](../04-leadership/20-staying-current.md))
