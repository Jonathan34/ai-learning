# Workshop W8 — Cost Optimization

**Goal:** Take an existing LLM system (like your W4 or W5 agent) and systematically reduce its cost without meaningfully hurting quality. Learn the levers that actually move the needle.

**Time:** 3-4 hours

**Prerequisites:** The agent from W4 or W5 with observability from W6 (you need to be able to measure before and after)

---

## What you're doing

Every LLM system has fat on it. Some of it is unavoidable — the cost of doing the work. Some of it is waste — tokens spent on nothing useful, calls made that didn't need to happen, context repeated unnecessarily.

This workshop walks through the main cost-reduction levers in order of impact. You'll measure baseline cost, apply each optimization, measure again, and understand which ones actually matter for your system.

---

## Part 1: Measure the baseline (30 min)

Before optimizing, measure what you're actually spending. From your W6 logs, for a representative sample of requests (at least 20):

```python
def calculate_cost_per_request(log_file, pricing):
    """Calculate cost per trace."""
    traces = {}
    with open(log_file) as f:
        for line in f:
            entry = json.loads(line)
            trace_id = entry.get("trace_id")
            if not trace_id:
                continue
            
            if trace_id not in traces:
                traces[trace_id] = {
                    "input_tokens": 0, 
                    "output_tokens": 0, 
                    "cached_tokens": 0,
                    "llm_calls": 0,
                    "tool_calls": 0
                }
            
            if entry.get("type") == "tool_call":
                traces[trace_id]["tool_calls"] += 1
            else:
                usage = entry.get("usage", {})
                traces[trace_id]["input_tokens"] += usage.get("input_tokens", 0)
                traces[trace_id]["output_tokens"] += usage.get("output_tokens", 0)
                traces[trace_id]["cached_tokens"] += usage.get("cached_tokens", 0)
                traces[trace_id]["llm_calls"] += 1
    
    # Calculate cost per trace
    for trace_id, stats in traces.items():
        input_cost = (stats["input_tokens"] - stats["cached_tokens"]) * pricing["input"] / 1_000_000
        cached_cost = stats["cached_tokens"] * pricing["cached"] / 1_000_000
        output_cost = stats["output_tokens"] * pricing["output"] / 1_000_000
        stats["cost_usd"] = input_cost + cached_cost + output_cost
    
    return traces

# Example pricing for Claude Sonnet (check current rates)
pricing = {
    "input": 3.00,      # per million tokens
    "cached": 0.30,     # per million tokens (90% discount)
    "output": 15.00,    # per million tokens
}

traces = calculate_cost_per_request("llm_calls.jsonl", pricing)

# Report
costs = [t["cost_usd"] for t in traces.values()]
print(f"Requests: {len(traces)}")
print(f"Avg cost: ${sum(costs)/len(costs):.4f}")
print(f"P95 cost: ${sorted(costs)[int(len(costs)*0.95)]:.4f}")
print(f"Max cost: ${max(costs):.4f}")
print(f"Avg LLM calls per request: {sum(t['llm_calls'] for t in traces.values())/len(traces):.1f}")
print(f"Avg tool calls per request: {sum(t['tool_calls'] for t in traces.values())/len(traces):.1f}")
print(f"Avg input tokens: {sum(t['input_tokens'] for t in traces.values())/len(traces):.0f}")
print(f"Avg output tokens: {sum(t['output_tokens'] for t in traces.values())/len(traces):.0f}")
```

Write these numbers down. They're your baseline. Every optimization below will be measured against this.

---

## Part 2: Enable prompt caching (30 min)

This is almost always the highest-impact optimization. If your system has a stable system prompt and task spec, you're reprocessing them on every request for nothing.

**How caching works:** the provider stores the computation for a prefix of your prompt. On subsequent requests with the same prefix, you only pay full price for the tokens that differ. Cache hits are typically 10% the cost of fresh input tokens.

**Requirements:**

- Cached prefix must be byte-identical across requests

- Minimum prefix length varies (Anthropic: 1024 tokens, OpenAI: 1024 tokens, adjust per provider)

- Cache TTL is usually 5 minutes (your prefix needs to be used that often to stay warm)

**Restructure your prompt:**

```python
# BEFORE — system prompt + context + user input all mixed
messages = [
    {"role": "system", "content": f"You are a helpful assistant. {dynamic_context} {user_instruction}"},
    {"role": "user", "content": user_message}
]

# AFTER — stable content first, dynamic content last
messages = [
    {
        "role": "system",
        "content": [
            {
                "type": "text",
                "text": STABLE_SYSTEM_PROMPT,  # 2000 tokens of instructions and tool descriptions
                "cache_control": {"type": "ephemeral"}  # Anthropic syntax
            }
        ]
    },
    {
        "role": "user",
        "content": f"{dynamic_context}\n\nUser request: {user_message}"  # changes per request
    }
]
```

Run your 20 test requests again. Compare:

- What percentage of your input tokens now come from cache?

- How much did cost per request drop?

For systems with large stable prompts (agent tool descriptions, detailed instructions), caching can cut input costs by 70-90%.

---

## Part 3: Constrain output length (20 min)

Output tokens cost 3-5x more than input tokens. Generating 2000 tokens when 200 would do is pure waste.

**Approaches:**

**Explicit max_tokens limit:**
```python
response = client.chat.completions.create(
    messages=messages,
    max_tokens=500,  # hard cap
)
```

**Prompt-level constraint:**
```
Keep your response under 150 words. Be direct and concise.
```

**Structured output (forces terse format):**
```
Respond in JSON with only these fields: summary (one sentence), action (one word), confidence (0-1).
```

Measure the impact. If your agent was generating 800-token reasoning chains before a final 100-token answer, you may cut output costs in half with tighter constraints. Quality often stays the same — verbose models aren't better models.

---

## Part 4: Route simple requests to smaller models (45 min)

Not every request needs the biggest model. A classification or simple lookup works fine on a small, cheap model.

**Build a classifier:**

```python
def classify_complexity(user_request):
    """Classify request complexity using a small fast model."""
    classifier_prompt = f"""Classify this request as 'simple' or 'complex'.

Simple: single-turn questions, straightforward lookups, classification, extraction.
Complex: multi-step reasoning, ambiguous requests, requires judgment.

Request: {user_request}

Classification (one word):"""
    
    response = call_llm(
        messages=[{"role": "user", "content": classifier_prompt}],
        model="claude-haiku-4",  # small, fast, cheap
        max_tokens=10
    )
    return response.strip().lower()

def route_request(user_request):
    complexity = classify_complexity(user_request)
    if complexity == "simple":
        return run_agent(user_request, model="claude-haiku-4")
    else:
        return run_agent(user_request, model="claude-sonnet-4")
```

Measure:

- What percentage of requests route to the small model?

- How much does that save per request?

- Did quality drop on simple requests? (Use your W3 eval harness to check.)

Typical savings: 50-70% on traffic that's classified as simple, usually 40-60% of total traffic. Net savings: 20-40% of your total bill.

---

## Part 5: Prune context aggressively (30 min)

Agents accumulate context as they run. By step 10, you might be sending 15,000 tokens of history when 2,000 would do.

**Strategies:**

**Summarize old steps:**
```python
def prune_context_by_summarization(messages, keep_recent=4, summarize_model="claude-haiku-4"):
    if len(messages) < keep_recent + 2:
        return messages  # nothing to summarize
    
    system_msg = messages[0]
    old_messages = messages[1:-keep_recent]
    recent_messages = messages[-keep_recent:]
    
    # Summarize the old messages
    summary = call_llm(
        messages=[
            {"role": "system", "content": "Summarize the key facts and decisions from this conversation in 3-5 bullet points."},
            {"role": "user", "content": json.dumps([m for m in old_messages])}
        ],
        model=summarize_model,
        max_tokens=300
    )
    
    return [
        system_msg,
        {"role": "assistant", "content": f"[Previous conversation summary:]\n{summary}"},
        *recent_messages
    ]
```

**Drop old tool outputs:**
Tool outputs often take up massive context space. Once you've used a tool result, you often don't need its full content anymore.

```python
def prune_old_tool_outputs(messages, keep_last_n_tool_results=3):
    # Replace old tool_results with short summaries
    tool_result_indices = [
        i for i, m in enumerate(messages) 
        if m.get("role") == "tool"
    ]
    
    if len(tool_result_indices) <= keep_last_n_tool_results:
        return messages
    
    # Summarize all but the most recent
    for i in tool_result_indices[:-keep_last_n_tool_results]:
        content = messages[i].get("content", "")
        if len(content) > 200:
            messages[i]["content"] = f"[Previous tool result — {len(content)} chars, summarized]"
    
    return messages
```

Measure the impact on context size and cost. A 10-step agent with good context pruning might use half the tokens of the same agent without pruning.

---

## Part 6: Limit tool calls (30 min)

Agents loop. Sometimes they loop too much. Each loop iteration is at least one LLM call plus potentially tool calls.

**Strategies:**

**Hard step limit (you probably have this already):**
```python
max_steps = 10
for step in range(max_steps):
    # agent loop
```

**Per-tool call limit:**
```python
tool_call_counts = {}

def execute_tool_limited(name, arguments, max_per_session=5):
    tool_call_counts[name] = tool_call_counts.get(name, 0) + 1
    if tool_call_counts[name] > max_per_session:
        return {"error": f"Maximum {max_per_session} calls to {name} per session. Use what you have."}
    return execute_tool(name, arguments)
```

**Detect repeated calls:**
```python
recent_calls = []

def execute_tool_dedup(name, arguments):
    call_key = (name, json.dumps(arguments, sort_keys=True))
    if call_key in recent_calls[-3:]:
        return {"error": "You just called this with the same arguments. Try something different or conclude."}
    recent_calls.append(call_key)
    return execute_tool(name, arguments)
```

**Enforce a budget:**
```python
class TokenBudget:
    def __init__(self, max_tokens):
        self.max_tokens = max_tokens
        self.used = 0
    
    def consume(self, tokens):
        self.used += tokens
        if self.used > self.max_tokens:
            raise BudgetExceeded(f"Used {self.used} of {self.max_tokens} tokens")

budget = TokenBudget(50_000)  # per-request budget

def call_llm_with_budget(messages, **kwargs):
    # estimate before calling
    estimated = estimate_tokens(messages)
    if budget.used + estimated > budget.max_tokens:
        # Force a final response
        raise BudgetExceeded()
    
    response = call_llm(messages, **kwargs)
    budget.consume(response.usage.prompt_tokens + response.usage.completion_tokens)
    return response
```

---

## Part 7: Measure the final impact (20 min)

Run your 20 test requests again with all optimizations enabled:

| Metric | Baseline | After optimization | % change |
|---|---|---|---|
| Avg cost per request | $0.XX | $0.XX | -XX% |
| P95 cost per request | $0.XX | $0.XX | -XX% |
| Avg LLM calls | X.X | X.X | -XX% |
| Avg input tokens | X,XXX | X,XXX | -XX% |
| Avg output tokens | XXX | XXX | -XX% |
| Quality score (from W3) | X% | X% | change |

If you did the exercises thoughtfully, you should see cost drops of 40-70% with minimal quality impact. If quality dropped significantly, you were too aggressive on one of the levers — roll back the one that hurt the most.

---

## Things that trip people up

**Optimizing before measuring.** You'll guess wrong about where your cost comes from. Measure first.

**Cache invalidation from small changes.** A single extra space in your system prompt breaks caching. Your prompt construction needs to produce byte-identical output every time.

**Model routing that doesn't actually route.** If your classifier always returns "complex," you're paying for the classifier AND the big model. Verify the actual routing distribution.

**Context pruning that drops critical information.** Aggressive summarization can lose facts the agent needed later. Test with your eval harness after every pruning change.

**Tool limits that cripple the agent.** If you set max_steps too low, the agent gives up on legitimate multi-step tasks. Find the right balance by looking at your successful traces — what's the 90th percentile step count for successful runs?

**Forgetting about user experience.** Lower cost is good. Slower responses because you're summarizing context with another LLM call is bad. Latency matters too.

---

## What you should have after this workshop

- A measured baseline cost for your system

- Applied 4-5 cost optimization techniques

- Measured impact of each technique individually

- A cost profile reduced by 40-70% vs baseline

- Understanding of which levers matter most for your specific workload

## Go deeper

- [Anthropic prompt caching docs](https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching) — the highest-impact optimization

- [OpenAI prompt caching docs](https://platform.openai.com/docs/guides/prompt-caching) — OpenAI's implementation

- [LiteLLM cost tracking](https://docs.litellm.ai/docs/proxy/cost_tracking) — multi-provider cost tracking

- [Chapter 11 (Inference Economics)](../03-production/11-inference-economics.md) covers the theory behind these optimizations
