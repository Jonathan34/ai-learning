# Workshop W9 — Agent Efficiency & KPIs

**Goal:** Instrument an agent to detect inefficient behavior — too many tool calls, too many steps, wasted context. Define KPIs that catch this in production and build dashboards to track them over time.

**Time:** 3-4 hours

**Prerequisites:** Agent with observability from W6. Cost optimization from W8 is useful but not required.

---

## What you're building

Agents fail in subtle, expensive ways. They complete their task but take 15 steps when 3 would have worked. They call the same tool 8 times. They produce the right answer after burning through 50,000 tokens. The user sees a correct response. Your bill sees the damage.

This workshop builds the instrumentation to detect these inefficiencies and the KPIs to track them.

---

## Part 1: Define efficiency metrics (30 min)

Efficiency isn't a single number. It's a profile across several dimensions:

**Step efficiency** — how many LLM calls per user request?
- Low (1-2 steps): simple request handled directly
- Medium (3-5 steps): typical multi-step task
- High (6+ steps): complex task or inefficient agent

**Tool efficiency** — how many tool calls per request, and are they diverse?
- One tool call per task step is typical
- The same tool called 5+ times is a warning sign (loop, retry, or the model not understanding the result)

**Token efficiency** — input tokens relative to output produced
- High input-to-output ratios mean the agent is reading a lot but producing little
- Context that grows unboundedly across steps is a sign of poor pruning

**Time efficiency** — wall-clock latency per request
- Each LLM call adds 500ms-3s
- Tool calls add variable time
- Total latency = sum of all sequential operations

**Task success** — did the agent actually complete the task?
- The whole point. No efficiency matters if the agent fails.

For each, you'll set thresholds that indicate "this run is inefficient" or "this pattern is worth investigating."

---

## Part 2: Compute efficiency metrics from traces (30 min)

From your W6 logs, compute these metrics per request:

```python
def compute_efficiency(trace_events):
    """Given all events for one trace, compute efficiency metrics."""
    llm_calls = [e for e in trace_events if e.get("type") != "tool_call"]
    tool_calls = [e for e in trace_events if e.get("type") == "tool_call"]
    
    # Step efficiency
    step_count = len(llm_calls)
    
    # Tool efficiency
    tool_count = len(tool_calls)
    tool_names = [tc["tool_name"] for tc in tool_calls]
    unique_tools = len(set(tool_names))
    repeated_calls = tool_count - unique_tools
    
    # Most-called tool
    from collections import Counter
    tool_counter = Counter(tool_names)
    most_common = tool_counter.most_common(1)
    most_common_tool = most_common[0] if most_common else ("none", 0)
    
    # Token efficiency
    total_input = sum(e.get("usage", {}).get("input_tokens", 0) for e in llm_calls)
    total_output = sum(e.get("usage", {}).get("output_tokens", 0) for e in llm_calls)
    input_to_output_ratio = total_input / max(total_output, 1)
    
    # Context growth (input tokens per step — how fast is context growing?)
    step_inputs = [e.get("usage", {}).get("input_tokens", 0) for e in llm_calls]
    context_growth = step_inputs[-1] - step_inputs[0] if len(step_inputs) > 1 else 0
    
    # Time efficiency
    total_duration_ms = sum(e.get("duration_ms", 0) for e in trace_events)
    
    return {
        "step_count": step_count,
        "tool_count": tool_count,
        "unique_tools": unique_tools,
        "repeated_tool_calls": repeated_calls,
        "most_common_tool": most_common_tool[0],
        "most_common_tool_count": most_common_tool[1],
        "total_input_tokens": total_input,
        "total_output_tokens": total_output,
        "input_to_output_ratio": input_to_output_ratio,
        "context_growth_tokens": context_growth,
        "total_duration_ms": total_duration_ms,
    }
```

Run this over your existing traces. Look at the distribution of each metric.

---

## Part 3: Define inefficiency signals (20 min)

Based on your data, define what "inefficient" means for your system:

```python
INEFFICIENCY_RULES = [
    {
        "name": "too_many_steps",
        "check": lambda m: m["step_count"] > 8,
        "severity": "warning",
        "description": "Agent took more than 8 LLM calls"
    },
    {
        "name": "same_tool_overused",
        "check": lambda m: m["most_common_tool_count"] > 4,
        "severity": "warning",
        "description": "Same tool called more than 4 times — possible loop"
    },
    {
        "name": "repeated_tool_calls",
        "check": lambda m: m["repeated_tool_calls"] > 3,
        "severity": "info",
        "description": "Multiple repeated tool calls"
    },
    {
        "name": "bloated_context",
        "check": lambda m: m["context_growth_tokens"] > 10_000,
        "severity": "warning",
        "description": "Context grew by 10k+ tokens — consider pruning"
    },
    {
        "name": "input_heavy",
        "check": lambda m: m["input_to_output_ratio"] > 50,
        "severity": "info",
        "description": "Very high input-to-output ratio — mostly reading, not producing"
    },
    {
        "name": "slow",
        "check": lambda m: m["total_duration_ms"] > 30_000,
        "severity": "warning",
        "description": "Total duration over 30 seconds"
    },
    {
        "name": "token_budget_blown",
        "check": lambda m: m["total_input_tokens"] + m["total_output_tokens"] > 100_000,
        "severity": "critical",
        "description": "Total tokens over 100k — very expensive run"
    },
]

def find_issues(metrics):
    return [
        rule for rule in INEFFICIENCY_RULES 
        if rule["check"](metrics)
    ]
```

Tune the thresholds for your system. If your typical request takes 6 steps, setting the threshold at 5 is too aggressive. If typical is 2, setting it at 10 is too lax.

---

## Part 4: Find your worst offenders (30 min)

Run the analysis over all your traces and find the worst cases:

```python
def analyze_all_traces(log_file):
    traces = group_events_by_trace(log_file)  # from W6
    
    results = []
    for trace_id, events in traces.items():
        metrics = compute_efficiency(events)
        issues = find_issues(metrics)
        
        results.append({
            "trace_id": trace_id,
            "metrics": metrics,
            "issues": [i["name"] for i in issues],
            "severity": max((i["severity"] for i in issues), default="ok", 
                          key=lambda s: ["ok", "info", "warning", "critical"].index(s))
        })
    
    return results

results = analyze_all_traces("llm_calls.jsonl")

# Find the worst by various dimensions
worst_by_steps = sorted(results, key=lambda r: -r["metrics"]["step_count"])[:5]
worst_by_tokens = sorted(results, key=lambda r: -(r["metrics"]["total_input_tokens"] + r["metrics"]["total_output_tokens"]))[:5]
worst_by_repeated_calls = sorted(results, key=lambda r: -r["metrics"]["repeated_tool_calls"])[:5]

print("Top 5 by step count:")
for r in worst_by_steps:
    print(f"  {r['trace_id']}: {r['metrics']['step_count']} steps, issues: {r['issues']}")
```

Pick the 3 worst traces. Open each one and read through it. What actually went wrong? Common patterns:

- **Loop on the same tool:** model keeps calling `search_X` with slightly different queries because it's not understanding the results
- **Redundant calls:** model calls `get_customer`, gets data, then calls `get_customer` again in a later step because it "forgot" the earlier result
- **Getting stuck:** model tries the same approach 3 times with minor variations, never pivoting to a different strategy
- **Over-reading:** model retrieves documents it doesn't need, then has to parse through them
- **Not stopping:** model has enough information but keeps making tool calls anyway

---

## Part 5: Implement targeted fixes (45 min)

For each pattern you found, implement a fix.

### Fix: Loop on the same tool

If the model keeps calling `search_X` with similar queries, tell it:

```python
# In the system prompt or after detecting a loop
"""
If you call the same tool 3 times without making progress, stop and explain to the user what you found and what you couldn't find. Do not keep retrying.
"""
```

Or enforce it in code:

```python
def execute_tool_with_loop_detection(name, arguments, history):
    # Count how many times this tool was called
    recent_calls = [h for h in history[-5:] if h["name"] == name]
    if len(recent_calls) >= 3:
        return {
            "error": f"Already called {name} {len(recent_calls)} times recently. Please use what you have or try a different approach."
        }
    return execute_tool(name, arguments)
```

### Fix: Redundant calls

Cache results within a single trace:

```python
trace_tool_cache = {}

def execute_tool_cached(name, arguments, trace_id):
    cache_key = (trace_id, name, json.dumps(arguments, sort_keys=True))
    if cache_key in trace_tool_cache:
        return {
            **trace_tool_cache[cache_key],
            "_cache_note": "Result from earlier in this session"
        }
    
    result = execute_tool(name, arguments)
    trace_tool_cache[cache_key] = result
    return result
```

### Fix: Not stopping

Add an explicit "are you done?" check in the system prompt:

```
Before each tool call, ask yourself: "Do I have enough information to answer the user's question?" If yes, stop calling tools and provide the answer. Do not use tools speculatively.
```

### Fix: Context bloat

Add periodic context pruning (from W8):

```python
if step > 5 and context_size(messages) > 10_000:
    messages = prune_context_by_summarization(messages)
```

---

## Part 6: Build a KPI dashboard (30 min)

Set up a simple dashboard. Even just a script that prints metrics is a start:

```python
def generate_kpi_report(log_file, lookback_days=7):
    """Generate weekly KPI report."""
    traces = analyze_all_traces(log_file)
    
    # Aggregate metrics
    total_requests = len(traces)
    avg_steps = sum(t["metrics"]["step_count"] for t in traces) / max(total_requests, 1)
    p95_steps = sorted(t["metrics"]["step_count"] for t in traces)[int(total_requests * 0.95)]
    avg_cost = sum(t["metrics"]["total_input_tokens"] * 3/1_000_000 + t["metrics"]["total_output_tokens"] * 15/1_000_000 for t in traces) / max(total_requests, 1)
    
    # Issue rates
    issue_counts = {}
    for t in traces:
        for issue in t["issues"]:
            issue_counts[issue] = issue_counts.get(issue, 0) + 1
    
    print(f"=== KPI Report (last {lookback_days} days) ===")
    print(f"Total requests: {total_requests}")
    print(f"Avg steps per request: {avg_steps:.2f}")
    print(f"P95 steps per request: {p95_steps}")
    print(f"Avg cost per request: ${avg_cost:.4f}")
    print()
    print("Issue frequency:")
    for issue, count in sorted(issue_counts.items(), key=lambda x: -x[1]):
        print(f"  {issue}: {count} ({100*count/total_requests:.1f}% of requests)")
```

At scale, you'd push these metrics to Grafana, Datadog, or your existing monitoring stack. Set alerts:

| KPI | Alert threshold |
|---|---|
| P95 steps per request | > 10 |
| Avg cost per request | 50% increase week-over-week |
| Tool loop rate | > 5% of requests |
| Task success rate (from W3 eval) | drops by > 5% |
| Tail latency (p99) | > 60s |

---

## Part 7: Compare before and after (20 min)

Run your test requests with the inefficiency fixes enabled. Compare to baseline:

| Metric | Before | After |
|---|---|---|
| Avg steps per request | | |
| Requests with tool loops | | |
| Avg tokens per request | | |
| Tail (p95) cost | | |
| Task success rate | | |

The goal: reduce waste without hurting task success. If success rate dropped, back off on the fixes that were too aggressive.

---

## Things that trip people up

**Optimizing for the average, ignoring the tail.** Most of your requests might be efficient. A small percentage of runaway requests consume most of your budget. Focus on P95/P99, not just averages.

**Setting thresholds before you have data.** Arbitrary thresholds produce either too many alerts (noise) or too few (misses). Look at your actual distribution first, then set thresholds at meaningful points (e.g., top 5% of runs).

**Fixing symptoms, not causes.** If the model keeps calling the same tool, adding a rate limit stops the symptom. The cause might be that the tool returns confusing output or the description is misleading. Look at the trace and fix the root cause when you can.

**Alert fatigue.** If every other request triggers an alert, people stop paying attention. Set thresholds where action is genuinely warranted.

**Forgetting that inefficiency is sometimes correct.** A genuinely complex request might need 10 steps. Not every long trace is a problem. Check quality before concluding something is broken.

---

## What you should have after this workshop

- A set of efficiency metrics computed from your agent traces
- Defined inefficiency signals with severity levels
- Identification of your actual worst-case runs
- Targeted fixes for the specific problems you found
- A KPI dashboard or report showing trends over time
- Threshold-based alerts for regression detection

## Go deeper

- [Langfuse metrics documentation](https://langfuse.com/docs/analytics) — built-in dashboards for agent metrics
- [Arize Phoenix](https://docs.arize.com/phoenix) — open-source observability with agent-specific metrics
- Chapter 12 (Observability) covers the monitoring theory
- Chapter 11 (Inference Economics) covers the cost dimensions
