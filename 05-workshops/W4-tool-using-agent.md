---
layout: default

title: "W4 — Tool-Using Agent"
nav_order: 4
parent: "Workshops"
---
# Workshop W4 — Tool-Using Agent (From Scratch)

> Status: **outline**. Will expand in full depth.

**Goal:** build an agent that uses tools, without a framework, so you understand the loop.

**Time:** 3-4 hours

**Planned contents:**

1. Pick a scenario: customer support triage, code review helper, document Q&A with tool calls, etc.
2. Define 3-5 tools with clear schemas (e.g., search_docs, fetch_url, run_query, write_file with sandbox, send_message stub)
3. Build the agent loop in Python, no framework:
   - System prompt with tool descriptions
   - User message
   - Model call, parse tool calls
   - Execute tools, capture results
   - Feed results back into context
   - Loop with max steps and stop conditions
4. Handle errors gracefully (tool fails, model hallucinates a tool name, infinite loop)
5. Add logging that captures every prompt, response, tool call, and result
6. Run 10 scenarios; see what breaks
7. Port to Claude Agent SDK or LangGraph; compare what changed
8. Reflect: what did the framework do for you, what did it hide

**Key gotchas demonstrated:**
- Tool name hallucinations
- Argument hallucinations
- Loops that don't terminate
- The cost of an agent loop vs. a single shot
- What "agent" really means in practice
