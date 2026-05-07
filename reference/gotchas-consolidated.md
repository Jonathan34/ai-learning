---
layout: default

title: "Gotchas (Consolidated)"
nav_order: 4
parent: "Reference"
---
# Consolidated Gotchas

The things experienced AI engineers have learned the hard way, in one list. Cross-referenced to the chapter where each is explained in context.

## On how LLMs work

- **"The model knows X"** — it doesn't. It has statistical associations. Never rely on it as a factual database without retrieval. (Ch 01)
- **Token ≠ word** — token counts vary wildly by language and tokenizer. Budget with the actual tokenizer. (Ch 01)
- **Non-determinism by default** — even temperature 0 has some non-determinism in practice. Design for it. (Ch 01)
- **Context ≠ memory** — the model has no memory across calls. Memory is a system you build. (Ch 01)

## On prompting

- **Prompts are coupled to model versions** — pin versions, run eval on upgrades. (Ch 02)
- **System prompts are not sandboxes** — don't put secrets there; they can be extracted. (Ch 02)
- **Instruction-following is soft** — 99% success rate looks great and fails 1% in production. Eval or accept. (Ch 02)
- **Verbose prompts aren't always better** — too long and the model loses track. (Ch 02)
- **"Act as an expert" does little** on modern instruction-tuned models. (Ch 02)

## On context engineering

- **Naive RAG feels magical in demos, fails in production** — chunking, re-ranking, hybrid search matter more than they seem. (Ch 03)
- **Lost in the middle** — put critical info near the end of long contexts. (Ch 03)
- **Embedding drift** — changing models means re-embedding everything. (Ch 03)
- **PII in context flows to the model provider** — know your data governance. (Ch 03)
- **RAG without "say I don't know" guardrails** will happily fabricate. (Ch 03)
- **Context window != budget to dump everything in** — assemble deliberately. (Ch 03)

## On evaluation

- **"We'll eval it once we have users"** — no. Build eval before features. (Ch 04)
- **LLM-as-judge with the same model you're evaluating** — calibration failure. (Ch 04)
- **Public benchmarks are often in training data** — don't trust SOTA claims without caveats. (Ch 04)
- **The demo works ≠ the feature works** — distribution testing, not point testing. (Ch 04)
- **Eval sets drift** — your golden set ages. Refresh periodically. (Ch 04)

## On security and safety

- **Refusals are trained, not hard-coded** — they can be bypassed. Defense in depth. (Ch 05)
- **Indirect prompt injection via tool outputs** is the insidious one. A fetched web page can hijack your agent. (Ch 05, Ch 08)
- **"The model will refuse" is not a security control** — don't rely on it. (Ch 05)
- **Sandboxing tools isn't optional** — if your agent has tools, they need sandboxes. (Ch 05, Ch 08)
- **User input should never concatenate into instructions** — always treat as data with delimiters. (Ch 05)
- **Capability creep for agents** — start with least privilege. (Ch 05)

## On agents

- **"Agent" is overloaded** — classify where on the spectrum, argue for simpler when possible. (Ch 06)
- **Loop conditions matter** — agents can run forever without a stop. Set budgets. (Ch 06)
- **Hallucinated tool calls** — model invents tool names or arguments. Validate. (Ch 06, Ch 08)
- **Agent cost multiplies steps** — a 10-step agent costs ~10x a one-shot call. (Ch 06)
- **Plan drift** — the agent's plan and behavior diverge mid-run. Observe, don't trust. (Ch 06)

## On frameworks

- **Frameworks hide what the model sees** — always be able to log the actual prompt. (Ch 07)
- **Frameworks change fast** — pin versions, expect breakage. (Ch 07)
- **Framework abstractions aren't always clean** — what looks simple may be 5 LLM calls. (Ch 07)

## On MCP and tools

- **Tool descriptions ARE prompts** — write them as carefully as any other part of the prompt. (Ch 08)
- **Tool outputs too long blow context** — truncate or summarize. (Ch 08)
- **MCP has no default security** — add auth, rate limits, capability constraints yourself. (Ch 08)

## On multi-agent

- **Multi-agent multiplies cost 5-10x** — make sure the benefit is real. (Ch 09)
- **Agents don't converge, they loop** — "let them negotiate" is an anti-pattern. (Ch 09)
- **Debugging multi-agent requires serious observability** — from the start. (Ch 09)

## On state and memory

- **Unbounded memory storage** — inserting everything forever is a cost and privacy disaster. (Ch 10)
- **Memory poisoning** — early false facts persist and compound. (Ch 10)
- **Stale memory vs. fresh observation conflicts** — have a resolution strategy. (Ch 10)
- **PII in memory has retention implications** — GDPR, CCPA, etc. (Ch 10)

## On inference economics

- **Output tokens dominate cost** — prompt caching doesn't help the generation side. (Ch 11)
- **Tail latency kills UX** — p99 matters more than p50 for interactive AI. (Ch 11)
- **Capacity is finite** — provider rate limits are real at scale. (Ch 11)
- **Agent loops multiply cost** — plan and budget accordingly. (Ch 11)

## On observability

- **Logging "prompt template" ≠ logging "actual prompt"** — log what the model literally saw. (Ch 12)
- **Full-context logs have PII and cost** — sample and redact intelligently. (Ch 12)
- **Dashboards without alerts** are decorative. (Ch 12)

## On deployment

- **"Just use OpenAI" has real governance implications** at scale. (Ch 13)
- **Self-hosting is harder than the blog posts make it sound** — ops, capacity, versioning. (Ch 13)
- **Multi-provider fallback is quality risk** — models differ more than providers admit. (Ch 13)

## On local inference

- **Quantization is lossy** — evaluate, don't assume parity. (Ch 14)
- **Local models lag frontier by 6-18 months** — plan around this. (Ch 14)
- **Memory is the real limit** — not just compute. (Ch 14)
- **Windows local inference has quirks** — more than Linux. (Ch 14)

## On platform

- **Building a platform too early** — solves problems no one has. (Ch 15)
- **No platform at all** — every team rebuilds, inconsistently. (Ch 15)
- **Ignoring cost attribution** — surprise six-figure bills. (Ch 15)

## On product

- **Shipping a chatbot by default** — often the wrong UX. (Ch 17)
- **Not designing for failure** — fragile launches. (Ch 17)
- **Over-promising** via model responses → trust loss. (Ch 17)

## On org

- **Over-hiring AI specialists** when applied engineers would do. (Ch 18)
- **Under-investing in platform** — "just build features" doesn't scale. (Ch 18)
- **Letting research disconnect from product** — papers not products. (Ch 18)

## On learning

- **Firehose anxiety** — you can't keep up with everything. Filter aggressively. (Ch 19)
- **Hype amplification on social** — researchers are better signals than influencers. (Ch 19)
- **Benchmark drama** — new SOTA is rarely actionable. (Ch 19)
