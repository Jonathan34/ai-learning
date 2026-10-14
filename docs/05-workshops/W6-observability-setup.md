# Workshop W6 — Observability Setup

**Goal:** Instrument an agent end-to-end so you can reconstruct what happened on a past request, and, when you deliberately turn content capture on, answer "what did the model see when it produced that output?"

**Time:** 2-3 hours

**Prerequisites:** The agent from W4 or W5. Python.

---

## What you're building

An observability layer that always captures:

- Every LLM call's model, token counts, finish reason, and timing

- Every tool call's name, duration, and whether it errored

- Trace IDs and step numbers, so you can reconstruct the shape of any past interaction

And, only when you switch content capture on:

- The prompt sent to the model (the full assembled text, not the template)

- The response received

- Tool arguments and results

Content is off by default because prompts, responses, and tool data can contain personal information, secrets, and proprietary context. That matches the OpenTelemetry GenAI conventions, which treat message content as opt-in and likely sensitive ([GenAI events conventions](https://github.com/open-telemetry/semantic-conventions-genai/blob/main/docs/gen-ai/gen-ai-events.md), checked 2026-10-12). See [Chapter 13](../03-production/13-observability.md) for the reasoning.

---

## Part 1: Structured logging (30 min)

Start with the simplest useful thing: structured JSON logs for every LLM call.

```python
import os
import time
import json
import uuid

# Off unless someone deliberately turns it on
CAPTURE_CONTENT = os.environ.get("LLM_CAPTURE_CONTENT") == "true"

def call_llm_instrumented(messages, tools=None, **kwargs):
    call_id = str(uuid.uuid4())[:8]
    start = time.time()

    response = client.chat.completions.create(
        messages=messages,
        tools=tools,
        **kwargs
    )

    duration = time.time() - start

    message = response.choices[0].message
    log_entry = {
        "call_id": call_id,
        "timestamp": time.time(),
        "model": kwargs.get("model", "default"),
        "tools_available": [t["name"] for t in (tools or [])],
        "tool_calls": [
            {"name": tc.function.name} for tc in (message.tool_calls or [])
        ],
        "finish_reason": response.choices[0].finish_reason,
        "usage": {
            "input_tokens": response.usage.prompt_tokens,
            "output_tokens": response.usage.completion_tokens,
        },
        "duration_ms": int(duration * 1000),
    }

    if CAPTURE_CONTENT:
        log_entry["messages"] = messages  # the full prompt
        log_entry["response"] = message.content
        log_entry["tool_calls"] = [
            {"name": tc.function.name, "args": tc.function.arguments}
            for tc in (message.tool_calls or [])
        ]

    # Write to a log file (in production, send to your log aggregator)
    with open("llm_calls.jsonl", "a") as f:
        f.write(json.dumps(log_entry) + "\n")

    return response

```

Replace your existing `client.chat.completions.create` calls with this instrumented version. Now every call is logged with its metadata. Run once with `LLM_CAPTURE_CONTENT=true` on your own test data and compare the two log entries, so you can see exactly what the flag adds.

In a real system, that flag deserves more ceremony than an environment variable: scope it to one tenant, flow, or incident, make it expire, and redact before writing. This workshop keeps it simple.

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
        "error": error,
        "duration_ms": int(duration * 1000),
    }

    if CAPTURE_CONTENT:
        log_entry["arguments"] = arguments
        log_entry["result"] = result

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
            print(f"  🔧 Tool: {event['tool_name']}({event.get('arguments', '<not captured>')})")
            if "result" in event:
                print(f"     Result: {str(event['result'])[:100]}...")
            if event.get("error"):
                print(f"     Error: {event['error']}")
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

Run your agent, then view the trace. With content capture off, you should still see the sequence of steps, which tools ran, what failed, and where the time and tokens went. With it on, you also see what the model was given and what it said.

---

## Part 5: Simulate a production incident (30 min)

Now use your observability to debug a problem:

1. **Introduce a bug.** Make one of your tools return incorrect data silently (e.g., wrong order status).

2. **Run the agent with content capture off.** It will produce a wrong answer based on the bad tool data. Look at the trace. How far can you narrow it down from metadata alone? Usually to the step and the tool, but not to the bad value.

3. **Turn content capture on for this incident and reproduce.** Set `LLM_CAPTURE_CONTENT=true`, run the same request on your test data, and read the trace. You should be able to see: the tool returned bad data → the model used that data → the final answer was wrong.

4. **Turn it off again.** Then delete the content entries you no longer need.

This is the test of whether your observability is sufficient. If metadata gets you to the right place and targeted content capture gets you the rest, you're in good shape.

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
trace = langfuse.trace(
    name="user_request",
    input=user_message if CAPTURE_CONTENT else None,
)

# Each LLM call becomes a "generation" span
generation = trace.generation(
    name="agent_step_1",
    model="claude-sonnet-4-20250514",
    input=messages if CAPTURE_CONTENT else None,
    output=response.choices[0].message.content if CAPTURE_CONTENT else None,
    usage={"input": response.usage.prompt_tokens, "output": response.usage.completion_tokens}
)

```

Sending content to a hosted tool means it now lives with a third party too. Keep the same flag in front of it.

Langfuse gives you a web UI to browse traces, filter by time/model/cost, and spot patterns. It's what the DIY approach above evolves into at scale.

---

## Gotchas

- **Capture the actual prompt, not the template.** When content capture is on, the template `{system_prompt} + {context}` is useless. You need the fully assembled text.

- **Full-context logging is expensive at scale.** A 10K-token prompt logged 1000 times/day is 10M tokens of log data per day. One more reason to capture content for incidents and approved eval samples rather than for every call.

- **PII, secrets, and proprietary data in logs.** User messages, retrieved documents, and tool results can contain all three. Keep content capture off by default. When it's on, redact before storing, restrict who can read it, and set a short retention period your privacy policy approves.

- **Debug flags outlive debugging.** Content capture switched on for an incident tends to stay on. Make it expire.

- **Correlation IDs are essential.** Without them, you can't link multiple LLM calls to a single user request. Add them from day one.

- **Timestamps need to be precise.** If you're measuring latency, use monotonic clocks, not wall clocks.

---

## What you should have after this workshop

- Instrumented LLM calls with metadata logging by default

- An opt-in switch that adds prompts, responses, tool arguments, and tool results

- Trace IDs linking all calls within a single user request

- Tool call instrumentation with timing and error capture

- A trace viewer that works with or without captured content

- The ability to narrow down a production issue from metadata, then confirm it with targeted content capture

## Go deeper

- [Langfuse documentation](https://langfuse.com/docs) — open-source observability for LLMs

- [OpenTelemetry GenAI events conventions](https://github.com/open-telemetry/semantic-conventions-genai/blob/main/docs/gen-ai/gen-ai-events.md) — the emerging standard, including which content fields are opt-in (checked 2026-10-12)

- [Chapter 13 (Observability)](../03-production/13-observability.md) covers the theory and production patterns
