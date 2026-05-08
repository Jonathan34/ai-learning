# State, Memory, and Long-Running Agents

An LLM has no memory. Every time you call it, it starts fresh. Any "memory" in an agent system is something you build around the model — state stored externally and fed back into the context on the next call.

For agents that run across multiple turns or multiple sessions, state management becomes the central engineering problem. It's often bigger than the model itself.

## Kinds of state

**Conversation history** — what was said so far in this session. Usually kept in the context directly, trimmed or summarized as it grows too long.

**Working memory** — the agent's current task, its plan, what it's tried so far, what it's learned from tool calls. This is the "scratchpad" that helps the agent stay on track across loop iterations.

**Long-term memory** — facts about the user, preferences learned over time, outcomes of prior conversations. Persists across sessions. Usually stored in a database or vector store and retrieved when relevant.

**Tool state** — results from tools the agent has called. Relevant for the current task but usually not worth persisting long-term.

## Short-term: managing conversation history

The simplest form of memory: keep the full conversation in the context. Works fine for short conversations. Breaks down when:

- The conversation exceeds the context window

- Old messages contain instructions that conflict with current ones

- The cost of processing the full history on every turn becomes too high

Solutions, from simplest to most complex:

**Truncation.** Drop the oldest messages when the context gets too long. Simple but lossy — you might drop something the user referenced earlier.

**Sliding window.** Keep the last N messages. Better than truncation but still lossy.

**Summarization.** Periodically summarize older messages into a shorter form and replace them. Preserves key information while reducing token count. Costs an extra LLM call to generate the summary.

**Selective retrieval.** Store all messages in a database. On each turn, retrieve only the messages relevant to the current query (using vector similarity or keyword matching). More complex but handles very long conversations well.

Most production systems use a combination: keep recent messages in full, summarize older ones, and retrieve from history when the user references something specific.

## Long-term: memory across sessions

When a user comes back tomorrow, what should the agent remember?

**User facts.** "Prefers concise responses". "Works in healthcare". "Has a dog named Max". These are extracted from conversations and stored as structured data.

**Prior interactions.** Relevant past conversations, retrieved when they're useful for the current one.

**Learned preferences.** How the user likes things formatted, what topics they care about, what they've explicitly corrected.

Architecturally, long-term memory is usually:

- A structured database for explicit facts (key-value pairs, user profiles)

- A vector store for semantic retrieval of past conversations

- Sometimes both, queried at the start of each session or each turn

## The durability problem

Agents that run for hours or days (background tasks, long research jobs, persistent assistants) need crash recovery. If the process dies mid-task, you need to resume where you left off.

**Event logging.** Record every action the agent took as an event. On restart, replay the log to reconstruct state. This is the same pattern as event sourcing in distributed systems.

**Checkpointing.** Save the full agent state at key points. On restart, load the last checkpoint and continue. LangGraph has built-in support for this.

**Idempotent tool calls.** If the agent retries a tool call after a crash, the result should be the same (or at least safe). Use idempotency keys for write operations. Design tools so that calling them twice with the same arguments doesn't cause problems.

## Memory quality

Memory systems decay over time. This is the part most teams skip and then regret.

**Stale facts.** The user changed jobs six months ago. The memory still says they work at the old company. Without a mechanism to update or expire facts, memory becomes misleading.

**Contradictions.** The user said "I prefer detailed responses" in January and "keep it short" in March. Which one wins? You need a conflict resolution strategy — usually "most recent wins" or "ask the user."

**Memory poisoning.** A user (or an attacker via prompt injection) plants a false fact early in a conversation. The agent stores it as a memory. Every future session now operates on false information. This is a real security concern.

**Unbounded growth.** If you store everything forever, storage costs grow, retrieval gets noisier, and irrelevant old memories pollute the context. You need pruning — either time-based (expire after N days), relevance-based (drop memories that are never retrieved), or explicit (let users delete memories).

## Privacy and retention

Memory means storing user data. That triggers real obligations:

- What data are you storing? (Conversations, extracted facts, preferences)

- How long do you keep it? (Retention policy)

- Can the user see what's stored? (Transparency)

- Can the user delete it? (Right to deletion — required by GDPR, CCPA)

- Where is it stored? (Data residency)

- Who can access it? (Access controls)

If you're building memory into a product, these aren't optional considerations. Plan for them from the start.

## Frameworks for memory

**LangGraph checkpointing** — built-in state persistence for graph-based agents. Handles short-term state well.

**Mem0** — a managed memory layer that handles extraction, storage, and retrieval of user facts. Abstracts the complexity but adds a dependency.

**Zep** — similar to Mem0, focused on conversation memory with automatic summarization.

**DIY with Postgres / Redis / S3** — often the right answer for teams that want full control. Store conversation history in Postgres, user facts in a structured table, semantic memories in a vector store (pgvector or similar).

## Things that trip people up

**Storing full conversation history forever.** Unbounded storage cost, noisy retrieval, privacy liability. Set retention limits.

**"Chat history" as the only memory.** Works for 5 turns. Breaks at 50. You need summarization or selective retrieval for longer interactions.

**Non-deterministic memory retrieval.** The same query retrieves different memories on different runs (because vector similarity has ties and ordering varies). This makes agent behavior inconsistent and hard to debug.

**Cold-start cost.** Rehydrating long memory at session start takes time. If you're loading 50 memories into context on every first message, that's latency the user feels.

**Memory without a way to correct it.** Users will notice when the agent "remembers" something wrong. If there's no way to correct it, trust erodes fast.

**Testing memory systems.** Hard — like testing distributed systems. You need to simulate multi-session interactions, verify that the right things are remembered and the wrong things are forgotten. Most teams under-test this.

## Where things stand

Memory is one of the less mature parts of the agent stack. Short-term memory (conversation history management) is well-understood. Long-term memory (cross-session persistence, fact extraction, quality maintenance) is still being figured out.

The tools are improving — Mem0, Zep, LangGraph checkpointing — but best practices are still emerging. If you're building memory into a system today, expect to iterate on the design as the field matures.

Memory is not a feature you bolt on. It's an architectural decision that affects privacy, cost, reliability, and user trust. Treat it with the same seriousness as your database schema.

## Go deeper

- [Mem0 documentation](https://docs.mem0.ai/) — managed memory layer

- [LangGraph persistence docs](https://langchain-ai.github.io/langgraph/concepts/persistence/) — checkpointing and state management

- [Letta (formerly MemGPT)](https://www.letta.com/) — research on memory-augmented agents

- [Workshop W4](../05-workshops/W4-tool-using-agent.md) and W5 both involve state management in practice
