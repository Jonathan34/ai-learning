---
layout: default

title: "W7 — Red-Team Exercise"
nav_order: 7
parent: "Workshops"
---
# Workshop W7 — Red-Team Exercise (Break Your Own System)

> Status: **outline**. Will expand in full depth.

**Goal:** attack the agent you built in W4/W5 and document the attack surface. You'll either fix the issues or accept them with a threat model.

**Time:** 3-4 hours

**Planned contents:**

1. Start with the agent from W4 or W5 (with tools)
2. Try direct prompt injection: user messages that attempt to override system instructions
3. Try indirect prompt injection: plant instructions in the content tools retrieve (fake web pages, fake documents)
4. Try data exfiltration: get the agent to reveal its system prompt, API keys in env, private context
5. Try denial of wallet: craft inputs that maximize tokens consumed
6. Try tool misuse: get the agent to call tools with malicious arguments (file paths, SQL, shell)
7. Try jailbreaks: get the model to produce content it was trained to refuse
8. Document each successful attack, assess severity, propose mitigations
9. Implement 2-3 mitigations and re-test
10. Write up findings — this is your first AI red-team report

**Key gotchas demonstrated:**
- Direct prompt injection is easier than you think
- Indirect injection via tool outputs is the really insidious one
- "The model will refuse" is not a security control
- Sandboxing tools isn't optional
- Logging is critical for forensics
- Defense in depth is required; no single control is sufficient

**Reference:**
- OWASP LLM Top 10
- Anthropic's "AI Safety Red-Teaming" materials
- Simon Willison's prompt injection archive (practical examples)
