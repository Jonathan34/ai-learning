---
layout: default

title: "08 — MCP and Tool Interfaces"
nav_order: 3
parent: "Agents"
---
# 08 — MCP and Tool Interfaces

## The mental model

**Tools are how LLMs do things in the real world. An LLM alone generates text. An LLM with a "send email" tool sends email. An LLM with an "execute SQL" tool reads your database.**

The tool interface is one of the most consequential design decisions in an agent system. It determines what the agent can do, how reliably it does it, and what can go wrong. Good tool design makes an agent useful. Bad tool design makes it frustrating or dangerous.

**Model Context Protocol (MCP)** — Anthropic's open protocol released in late 2024 and gaining traction through 2025 — is emerging as the standard for connecting LLMs to tools and data sources in a portable way. The way USB-C standardized physical connections, MCP is attempting to standardize LLM-to-tool connections. This is a real shift worth understanding.

**The thing to remember:** tool descriptions are part of the prompt. The model doesn't read your code; it reads the natural-language description of what the tool does and when to use it. Tools are prose first, code second.

## Why this matters in production

Most "the agent is broken" complaints trace to tool issues:

- **Bad tool descriptions** → the model picks the wrong tool or doesn't recognize when to use one
- **Bad tool schemas** → the model hallucinates arguments
- **Too many tools** → the model gets confused, picks suboptimally, or uses them inefficiently
- **Too few tools** → the agent can't complete the task and dead-ends
- **Tools that fail loudly** → the model sees an error and gives up; better to return a graceful message
- **Tools that fail silently** → the model proceeds on bad data

Tool engineering is 40-60% of agent engineering. It's worth taking seriously.

## Tool design fundamentals

### Anatomy of a tool

Every tool has:
- **Name** — how the model refers to it
- **Description** — what it does, when to use it, critically important
- **Parameters** — schema (typically JSON Schema) of required and optional arguments
- **Return value** — what the model gets back

Example (in provider-neutral pseudocode):

```json
{
  "name": "search_customer_orders",
  "description": "Look up a customer's order history. Returns up to 20 recent orders with order_id, date, status, and total. Use this when the user asks about their past purchases, order status, or when you need to reference a specific order.",
  "parameters": {
    "type": "object",
    "properties": {
      "customer_id": {
        "type": "string",
        "description": "The customer's unique ID, usually extracted from authentication context."
      },
      "status_filter": {
        "type": "string",
        "enum": ["all", "open", "completed", "cancelled"],
        "description": "Filter by order status. Use 'all' if the user didn't specify."
      }
    },
    "required": ["customer_id"]
  }
}
```

Good tools follow a few principles:

**1. Specific names.** `search_customer_orders` not `search`. The name alone should give the model a strong hint about when to use the tool.

**2. Detailed descriptions.** Tell the model what the tool does, what it returns, and critically, *when to use it*. "Use this when..." is the single most valuable phrase in a tool description.

**3. Typed parameters with descriptions.** Every parameter gets a description. `status_filter: "Filter by order status"` is weaker than the version above that enumerates options and tells the model what to do when unspecified.

**4. Structured return values.** Give the model parseable output — JSON, tables, structured text. Don't return free-form prose that the model then has to re-parse.

**5. Stable schemas.** Tool schemas are part of the contract. Changing them subtly changes model behavior. Treat them like any API contract: versioned, tested, reviewed.

### Name and description as prompt

The model never sees your implementation. It sees the name and description. So:

- **Name a tool for what it does, not how it works.** `check_refund_eligibility` not `call_stripe_api`.
- **Describe side effects explicitly.** If a tool sends email, the description should say "Sends an email to the customer." If it's read-only, say so.
- **Include usage constraints.** "Only call this for orders less than 30 days old." "Do not call this more than once per session."
- **Anti-examples help.** "Do not use this for general product questions; use `search_product_catalog` instead."

Time spent writing tool descriptions is prompt engineering. Write them carefully, test them, iterate.

### Return values: what the model can do next

The return format shapes what the model can do with the result. Good return values:

- **Contain everything the model might need.** If orders have status, date, and total, return all three even if the user asked about status. Fetching once is cheaper than fetching twice.
- **Are structured enough to reason over.** JSON or a table beats prose.
- **Handle "nothing to return" gracefully.** An empty result should say "No orders found for customer X" not return `[]`, which the model may misinterpret.
- **Include metadata when useful.** Source, timestamp, reliability.
- **Truncate sensibly.** Returning 10MB of data to the model is a waste and blows the context. Truncate with a note: "Returned first 20 of 150 results; use `next_page` to continue."

### Errors: the model needs to know what went wrong

When a tool call fails, you have choices about what to return to the model:

- **Bad choice:** raise an exception and crash the agent. The user sees a 500; the model never gets a chance to recover.
- **Bad choice:** return nothing. The model has no signal; it may assume success and proceed on empty data.
- **Good choice:** return a structured error with a clear message. `{"error": "customer_id not found", "recovery": "Ask the user to verify the ID, or use search_customer_by_email"}`.

The error message is a prompt to the model. Write it to help the model recover.

### Idempotency, retries, rate limits

Agent loops retry. Tools need to handle that.

- **Idempotent writes:** if the agent calls `update_customer_email` twice with the same arguments, the second call should be a no-op or at least safe. Use idempotency keys when possible.
- **Rate limits:** if a tool can be rate-limited, return a structured rate-limit response. Tell the model to wait or try later. Don't just 429.
- **Retries inside the tool:** for transient failures, retry internally with backoff before returning to the model. Don't burden the model's loop with retry logic it may not handle well.

## Tool sets: how many, which ones

Too few tools → the agent can't accomplish tasks. Too many → the model gets confused.

A rough heuristic: **5-15 tools is the sweet spot for most agents.** Below 5 and you're often asking the model to improvise around missing capability. Above 15-20 and the model increasingly picks wrong tools or misses relevant ones.

Some strategies when you have too many tools:

- **Hierarchical tool sets.** A "meta-tool" the agent calls first that returns a relevant subset.
- **Dynamic tool selection.** Use embeddings to retrieve the most relevant tools for the current query before giving them to the agent.
- **Multi-agent routing.** Specialist agents, each with their own tool subset; a router picks which specialist handles the query.
- **Just cut some tools.** Often the right answer. Many tools were added "in case" and aren't pulling their weight.

## Model Context Protocol (MCP)

### What MCP is

MCP is an open protocol specification that defines how AI applications (clients) connect to servers that expose tools, resources, and prompt templates. Published by Anthropic in November 2024, picked up by OpenAI, Google, and much of the ecosystem through 2025.

The pitch: instead of every AI app reimplementing connections to Gmail, GitHub, Slack, Postgres, filesystems, etc., there's a standard protocol. Anyone can write an MCP server that exposes their system; anyone can write an MCP client that consumes those servers.

Think of it as the AI equivalent of LSP (Language Server Protocol, which standardized editor/language integrations). Before LSP, every editor had custom integrations for every language. After LSP, one server talks to any compliant editor.

### What an MCP server exposes

- **Tools** — functions the LLM can call (same concept as provider-native tool use, but portable)
- **Resources** — data the LLM can read (files, database rows, API responses)
- **Prompts** — templates the client can surface to the user ("Summarize this document," "Review this PR")

A single server can expose any combination. A filesystem MCP server exposes tools for reading/writing files and resources for the file tree. A database MCP server exposes tools for querying and resources for schema introspection.

### How it works (briefly)

- **Transport:** stdio (subprocess), HTTP/SSE, or newer streamable HTTP. Stdio is common for local dev; HTTP for remote servers.
- **Capabilities negotiation:** client and server exchange what they support on connect.
- **Tool invocation:** client sends a tool call; server executes and returns the result.
- **Resource fetch:** client requests a resource; server returns content.

The protocol handles discovery ("what tools do you have?") and invocation, with metadata for schemas, descriptions, and pagination.

### The ecosystem

As of late 2025:

- **Reference MCP servers** exist for filesystem, fetch, GitHub, Slack, Google Drive, Postgres, SQLite, Puppeteer, and more
- **Client support** in Claude Desktop, several IDE assistants, Cursor, Cline, Warp, and growing
- **Third-party servers** have appeared for most popular SaaS tools (Notion, Linear, Jira, etc.)
- **OpenAI added MCP support** to their Agents SDK in 2025
- **Community momentum** is significant; the modelcontextprotocol.io directory lists hundreds of servers

### Why this matters

- **Portability.** An MCP server works with any compliant client. You're not tying your integrations to one AI provider.
- **Ecosystem leverage.** You can use dozens of existing servers without writing them yourself.
- **Standardized security model.** MCP has evolving conventions for auth, capability scoping, and user consent.
- **Local-first friendly.** Many MCP servers run locally, keeping your data on your machine.

### Building an MCP server

If you have a tool or data source you want to expose to agents, writing an MCP server is straightforward:

- Python SDK: `mcp` package
- TypeScript SDK: `@modelcontextprotocol/sdk`
- Define your tools, resources, and prompts
- Pick a transport (stdio for local, HTTP for remote)
- Expose it

Example (minimal Python MCP server):

```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("my-server")

@mcp.tool()
def add(a: int, b: int) -> int:
    """Add two numbers."""
    return a + b

if __name__ == "__main__":
    mcp.run()
```

That's a working MCP server. Plugin it into Claude Desktop and Claude can now call `add`.

### Security implications of MCP

MCP lowers the bar to connecting tools to agents, which is good and dangerous. Key considerations:

- **MCP has no default authentication or authorization.** Servers you run locally trust the client; servers over HTTP need you to add auth.
- **Once connected, MCP tools are privileged.** The protocol doesn't enforce capability constraints; that's the application's job.
- **Untrusted MCP servers are a supply chain risk.** Running a random MCP server from the community is like running a random script from the internet. Audit before adopting.
- **Indirect prompt injection via MCP resources.** Content returned by an MCP server can contain injection payloads. Same defenses as with any tool.

**Senior move:** treat MCP servers like browser extensions. You wouldn't install a random browser extension; don't run a random MCP server. Check the source, understand the scope of what it can do, run with minimum privileges.

### MCP vs provider-native tool use

You can use Claude, GPT-5, Gemini, etc. with either:
- Their native tool use APIs (define tools in-code, pass to the API)
- MCP (tools come from MCP servers, client handles the bridging)

Native tool use is simpler for contained use cases where you're writing and deploying all the tools yourself. MCP wins when you want to reuse external servers, maintain portability, or build something that others will consume.

For many production systems, both coexist: some tools defined natively (custom business logic), some via MCP (standard integrations).

## Design patterns worth knowing

### Read tools vs. write tools

Separate them mentally and often physically. Read tools are safe to call speculatively. Write tools need guardrails.

- Read: `get_customer`, `search_products`, `list_tickets`
- Write: `update_customer`, `cancel_order`, `send_email`

Give the model read tools freely. Gate write tools with confirmations, budgets, or human checks depending on stakes.

### Tool composition

An agent can use tool output from one call as input to the next. This is powerful and occasionally error-prone.

Design considerations:
- **IDs and references.** If `search_customers` returns a list, each should have a stable ID the agent can pass to `get_customer_details`.
- **Linked pagination.** Return a cursor the model can use to fetch more, not just "there's more."
- **Avoid over-chaining.** If a common flow requires 4 tool calls, consider a higher-level tool that does all 4.

### Dynamic tool discovery

Some systems discover tools at runtime rather than hard-coding them. MCP supports this: the client asks the server what tools exist. Useful for:
- Development environments where tools change
- User-extensible systems where users bring their own integrations
- Multi-tenant agents where different users have different tool sets

Trade-off: dynamic tools are harder to test and eval — you don't know all the cases at design time.

### Tools that return data for rendering

Some tools exist less to inform the agent and more to surface data to the user. "Show a map of this location," "render a chart of these numbers." These are worth calling out as a distinct pattern: the return value is consumed by the UI, not the next reasoning step.

## Gotchas

**Gotcha: Tool descriptions that read like code comments.**

"Returns customer data" is not a description. "Retrieves the customer's profile including contact info, account status, and recent activity. Use this when answering questions about a specific customer or before performing customer-related operations" is a description. Budget time for description-writing.

**Gotcha: Argument hallucinations.**

The model invents arguments. A tool expects `order_id` as a string like "ORD-12345"; the model passes `42`. Defensive validation: parse, type-check, return a clear error if the argument is wrong, so the model can retry correctly.

**Gotcha: Tool name conflicts.**

If two tools have similar names, the model will conflate them. `search_orders` and `search_customer_orders` are different; the model may mix them up under pressure. Name distinctively.

**Gotcha: Return values that blow the context window.**

A `list_files` tool that returns 10,000 files in a single call destroys the agent's budget. Default to reasonable page sizes. Return counts and truncation metadata.

**Gotcha: Indirect prompt injection via tool output.**

A `fetch_url` tool returns a web page. The page has hidden instructions. The model follows them. This is covered more in the Security chapter, but worth noting here: any tool that returns untrusted content is an injection vector.

**Gotcha: Tools that make assumptions about user context.**

A tool that "sends a message" needs to know who to send it to. If it infers from context, it may get this wrong in edge cases. Better: require the recipient explicitly as a parameter.

**Gotcha: Rate limits invisible to the model.**

External APIs rate-limit. If your tool wraps one, handle limits internally (queue, backoff, structured error response). Don't let rate-limit errors reach the model without context; the model can't reason about "try again in 60 seconds" reliably.

**Gotcha: MCP servers you didn't write.**

Treat as untrusted. Audit the source. Run with least privilege. Don't give MCP servers access to credentials, networks, or data they don't need.

**Gotcha: Tool schemas drifting from implementation.**

The schema says `status` is an enum of four values; the implementation returns a fifth value in some cases. The model now sees output it can't parse. Keep schema and implementation in sync; ideally generate one from the other.

## Honest status

Tool design is mature as a discipline — the principles above are well-established, and tool-using LLMs work well in production today. MCP is younger but has strong momentum; it's reasonable to bet on it for new systems.

Watch for:
- MCP security practices to formalize (auth, capability scoping)
- Tool schema standards to stabilize across providers
- Tool discovery/marketplace patterns to mature
- Evaluation tooling for tool-using agents to improve

## What to read next

- **Workshop W4 — Tool-Using Agent.** Build an agent with proper tool design, see the failure modes directly.
- **09 — Multi-Agent Patterns.** When does tool use extend into multi-agent territory?
- **modelcontextprotocol.io** — the official MCP spec and server directory.
- **Anthropic's "Tool use with Claude" docs** — practical patterns that apply to any LLM.
- **The MCP source code on GitHub** — reading the SDK source is one of the best ways to understand the protocol.
