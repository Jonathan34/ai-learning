# Workshop W6 — Observability Setup

**Goal:** Instrument an agent end-to-end so you can answer "what did the model see when it produced that output?" for any past call.

**Time:** 2-3 hours

**Prerequisites:** The agent from W4 or W5. Python.

---

## What you're building

An observability layer that captures:
- Every prompt sent to the model (the full assembled text, not the template)
- Every response received
- Every tool call (name, arguments, result, duration)
- Timing and cost for each step
- Enough metadata to reconstruct any past interaction

---

## Part 1: Structured logging (30 min)

Start with the simplest useful thing: structured JSON logs for every LLM call.

```python
import time
import json
import uuid

def call_llm_instrumented(messages, tools=None, **kwargs):
    call_id = str(uuid.uuid4())[:8]
    start = time.time()
    
    response = client.chat.completions.create(
        messages=messages,
        tools=tools,
        **kwargs
    )
    
    duration = time.time() - start
    
    log_entry = {
        "call_id": call_id,
        "timestamp": time.time(),
        "model": kwargs.get("model", "default"),
        "messages": messages,  # THE FULL PROMPT
        "tools_available": [t["name"] for t in (tools or [])],
        "response": response.choices[0].message.content,
        "tool_calls": [
            {"name": tc.function.name, "args": tc.function.arguments}
            for tc in (response.choices[0].message.tool_calls or [])
        ],
        "usage": {
            "input_tokens": response.usage.prompt_tokens,
            "output_tokens": response.usage.completion_tokens,
        },
        "duration_ms": int(duration * 1000),
    }
    
    # Write to a log file (in production, send to your log aggregator)
    with open("llm_calls.jsonl", "a") as f:
        f.write(json.dumps(log_entry) + "\n")
    
    return response
```

Replace your existing `client.chat.completions.create` calls with this instrumented version. Now every call is logged with full context.

---

## Part 2: Request-level tracing (30 min)

A single user request may trigger multiple LLM calls (agent loop). Link them with a trace ID:

```python
import contextvars

trace_id_var = contextvars.ContextVar('trace_id', default=None)

def start_trace():
    trace_id = str(uuid.uuid4())[:12]
    trace_id_var.set(trace_id)
    return trace_id

def call_llm_instrumented(messages, tools=None, **kwargs):
    trace_id = trace_id_var.get()
    # ... same as before, but add trace_id to log_entry
    log_entry["trace_id"] = trace_id
    log_entry["step"] = get_and_increment_step()  # track step number within trace
    # ...
```

Now in your agent loop:

```python
def run_agent(user_message, **kwargs):
    trace_id = start_trace()
    print(f"Trace: {trace_id}")
    # ... rest of agent loop using call_llm_instrumented
```

Every LLM call within one user request shares a trace ID. You can reconstruct the full trajectory by filtering logs on that ID.

---

## Part 3: Tool call instrumentation (30 min)

Instrument tool execution the same way:

```python
def execute_tool_instrumented(name, arguments):
    start = time.time()
    
    try:
        result = execute_tool(name, arguments)
        error = None
    except Exception as e:
        result = None
        error = str(e)
    
    duration = time.time() - start
    
    log_entry = {
        "trace_id": trace_id_var.get(),
        "type": "tool_call",
        "timestamp": time.time(),
        "tool_name": name,
        "arguments": arguments,
        "result": result,
        "error": error,
        "duration_ms": int(duration * 1000),
    }
    
    with open("llm_calls.jsonl", "a") as f:
        f.write(json.dumps(log_entry) + "\n")
    
    if error:
        return {"error": error}
    return result
```

---

## Part 4: Build a trace viewer (30 min)

A simple script that reconstructs a trace from logs:

```python
def view_trace(trace_id):
    events = []
    with open("llm_calls.jsonl") as f:
        for line in f:
            entry = json.loads(line)
            if entry.get("trace_id") == trace_id:
                events.append(entry)
    
    events.sort(key=lambda e: e["timestamp"])
    
    print(f"=== Trace {trace_id} ({len(events)} events) ===\n")
    
    total_tokens = 0
    total_cost = 0
    
    for event in events:
        if event.get("type") == "tool_call":
            print(f"  🔧 Tool: {event['tool_name']}({event['arguments']})")
            print(f"     Result: {str(event['result'])[:100]}...")
            print(f"     Duration: {event['duration_ms']}ms")
        else:
            tokens = event.get("usage", {})
            total_tokens += tokens.get("input_tokens", 0) + tokens.get("output_tokens", 0)
            print(f"  🤖 LLM call (step {event.get('step', '?')})")
            print(f"     Model: {event.get('model')}")
            print(f"     Input tokens: {tokens.get('input_tokens', '?')}")
            print(f"     Output tokens: {tokens.get('output_tokens', '?')}")
            print(f"     Duration: {event['duration_ms']}ms")
            if event.get("tool_calls"):
                print(f"     → Decided to call: {[tc['name'] for tc in event['tool_calls']]}")
            elif event.get("response"):
                print(f"     → Final response: {event['response'][:100]}...")
        print()
    
    print(f"Total tokens: {total_tokens}")
    print(f"Total events: {len(events)}")
```

Run your agent, then view the trace. You should be able to see exactly what happened at each step.

---

## Part 5: Simulate a production incident (30 min)

Now use your observability to debug a problem:

1. **Introduce a bug.** Make one of your tools return incorrect data silently (e.g., wrong order status).
2. **Run the agent.** It will produce a wrong answer based on the bad tool data.
3. **Debug from the trace alone.** Can you identify where things went wrong just by reading the trace? You should be able to see: the tool returned bad data → the model used that data → the final answer was wrong.

This is the test of whether your observability is sufficient. If you can reconstruct the failure from logs alone (without re-running the agent), you're in good shape.

---

## Part 6 (optional): Connect to Langfuse (30 min)

If you want to see what a real observability tool looks like, connect to Langfuse (free tier available):

```python
from langfuse import Langfuse

langfuse = Langfuse(
    public_key="pk-...",
    secret_key="sk-...",
    host="https://cloud.langfuse.com"
)

# Wrap your agent run in a trace
trace = langfuse.trace(name="user_request", input=user_message)

# Each LLM call becomes a "generation" span
generation = trace.generation(
    name="agent_step_1",
    model="claude-sonnet-4-20250514",
    input=messages,
    output=response.choices[0].message.content,
    usage={"input": response.usage.prompt_tokens, "output": response.usage.completion_tokens}
)
```

Langfuse gives you a web UI to browse traces, filter by time/model/cost, and spot patterns. It's what the DIY approach above evolves into at scale.

---

## Gotchas

- **Log the actual prompt, not the template.** The template says `{system_prompt} + {context}`. You need the fully assembled text. This is the #1 mistake teams make.
- **Full-context logging is expensive at scale.** A 10K-token prompt logged 1000 times/day is 10M tokens of log data per day. Sample in production (log 100% of errors, 5-10% of successes).
- **PII in logs.** User messages contain personal data. Redact or mask before storing. Set retention policies.
- **Correlation IDs are essential.** Without them, you can't link multiple LLM calls to a single user request. Add them from day one.
- **Timestamps need to be precise.** If you're measuring latency, use monotonic clocks, not wall clocks.

---

## What you should have after this workshop

- Instrumented LLM calls with full prompt, response, and metadata logging
- Trace IDs linking all calls within a single user request
- Tool call instrumentation with timing and error capture
- A trace viewer that reconstructs any past interaction
- The ability to debug a production issue from logs alone

## Go deeper

- [Langfuse documentation](https://langfuse.com/docs) — open-source observability for LLMs
- [OpenTelemetry for GenAI](https://opentelemetry.io/docs/specs/semconv/gen-ai/) — the emerging standard
- [Chapter 12 (Observability)](../03-production/12-observability.md) covers the theory and production patterns
