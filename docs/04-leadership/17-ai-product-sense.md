# AI Product Sense

A production AI feature is not a model. It's a model wrapped in UX decisions that determine whether users trust it, use it, and benefit from it. Good AI product sense is a specific skill — adjacent to general product sense but with its own traps.

## What good AI UX looks like

**Streaming over spinners.** When the model takes 3-5 seconds to respond, showing tokens as they arrive feels fast. A spinner for 5 seconds feels broken. Streaming is almost always the right choice for interactive AI.

**Visible uncertainty.** When the model isn't sure, the UI should show it. "Based on the available documents, it appears that..." is better than stating uncertain things as fact. Users trust systems that admit limitations.

**Citations and verification paths.** If the AI makes a claim, show where it came from. "According to [Document X, page 3]..." lets users verify. This is especially important for RAG systems where the answer is grounded in specific sources.

**Graceful degradation.** When the model fails (refuses, produces garbage, times out), the user should see something useful — not a blank screen or a generic error. "I couldn't find an answer to that. Here are some related topics..." is better than nothing.

**Editable outputs.** AI-generated drafts should be easy to edit. The user should be able to accept, modify, or reject. The AI is a starting point, not a final answer.

**Appropriate disclosure.** Users should know when they're interacting with AI. Not every response needs "I am an AI" — but the overall experience should be transparent about what's automated.

## Interaction patterns

Not everything needs to be a chatbot. Common patterns:

| Pattern | When to use | Example |
|---|---|---|
| **Chat** | Open-ended exploration, multi-turn tasks | Customer support, research assistant |
| **Single-turn Q&A** | Specific questions with specific answers | Search, FAQ, documentation lookup |
| **Draft generation** | User needs a starting point to edit | Email drafts, report templates, code suggestions |
| **Inline suggestions** | User is working; AI assists in context | Autocomplete, grammar correction, code completion |
| **Background processing** | AI works asynchronously on a batch | Summarizing meeting notes, classifying tickets |
| **Invisible AI** | AI powers a feature without the user knowing | Smart search ranking, content recommendations |

The chatbot is overused. Many tasks are better served by a single-turn interface, inline suggestions, or background processing. Ask: does the user actually want a conversation, or do they want a result?

## Latency budget

| Interaction type | Acceptable latency | Notes |
|---|---|---|
| Inline suggestions | < 200ms | Must feel instant |
| Chat (streaming) | < 1s to first token | Streaming makes the rest tolerable |
| Chat (no streaming) | < 3s total | Beyond this, users abandon |
| Background processing | Minutes to hours | User doesn't wait; notify when done |
| Batch/async | Hours | Acceptable for bulk operations |

If your system can't meet the latency budget for the interaction type you've chosen, either optimize (smaller model, shorter prompts, caching) or change the interaction pattern (move from real-time to async).

## Trust engineering

Users trust AI systems that are:

- **Calibrated.** They admit uncertainty when uncertain and are confident when confident.

- **Consistent.** Same question gets similar answers across sessions.

- **Transparent.** They show their sources, explain their reasoning when asked, and don't pretend to be human.

- **Correctable.** When wrong, the user can fix it and the system learns (or at least doesn't repeat the mistake in the same session).

Users distrust AI systems that are:

- **Overconfident.** Stating wrong things as fact.

- **Inconsistent.** Different answers to the same question on different days.

- **Opaque.** No way to understand why it said what it said.

- **Uncorrectable.** User says "that's wrong" and the system ignores it or repeats the error.

## Designing for failure

Every AI feature will produce bad outputs sometimes. The question is: what does the user experience when that happens?

**For low-stakes features** (suggestions, drafts): make it easy to dismiss or edit. The cost of a bad suggestion is one click to ignore it.

**For medium-stakes features** (customer-facing responses, reports): add a review step. Show the AI output to a human before it reaches the end user.

**For high-stakes features** (medical, legal, financial): the AI should be advisory only. A human makes the final decision. The UI should make this clear.

The worst outcome is a high-stakes AI feature that looks authoritative and is sometimes wrong. Users will trust it, act on it, and get hurt.

## Things that trip people up

**Shipping a chatbot by default.** Chat is the most complex interaction pattern (multi-turn state, context management, open-ended inputs). It's often not what the user needs. A simple form with an AI-powered response is often better.

**Not designing for the failure case.** The happy path demo looks great. What happens when the model refuses? When it hallucinates? When it times out? When the user asks something out of scope? Design these paths explicitly.

**Over-promising via model responses.** If the model says "I've scheduled your appointment for Tuesday" but it actually can't schedule anything, trust is destroyed. Constrain what the model claims to do to what it can actually do.

**Inconsistent behavior across sessions.** Users build mental models of how the AI works. If it behaves differently each time (because of temperature, context differences, or model updates), users can't predict it and stop trusting it.

**Ignoring accessibility.** Streaming text, dynamic content, and conversational interfaces all have accessibility implications. Screen readers, keyboard navigation, and reduced-motion preferences all need consideration.

## Where things stand

AI product sense is still developing as a discipline. Most AI features today are chatbots or copilot-style suggestions. The design space is much larger — background processing, invisible AI, structured workflows with AI at specific nodes.

The teams building the best AI products are the ones that start with the user's task, not with the model's capabilities. "What does the user need to accomplish?" first. "How can AI help?" second.

## Go deeper

- [Anthropic's design guidelines for Claude](https://docs.anthropic.com/en/docs/build-with-claude) — practical UX patterns

- [Nielsen Norman Group on AI UX](https://www.nngroup.com/topic/artificial-intelligence/) — research-backed design guidance

- [Apple's Human Interface Guidelines for AI](https://developer.apple.com/design/human-interface-guidelines/machine-learning) — platform-specific but principles transfer
