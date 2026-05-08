# Deciding What to Build with AI

Most organizations are making two opposite mistakes right now: shipping AI features where AI isn't the right tool, and dismissing AI where it is. A Principal Engineer's job is to help the organization make better decisions about where AI creates real value.

## The viability filter

Before building, ask:

1. **Is there a genuine task being automated or augmented?** "Add AI" is not a task. "Classify incoming support tickets by urgency" is.
2. **Is this something LLMs are actually good at?** LLMs excel at language tasks: summarization, classification, extraction, generation, translation, conversation. They're bad at: precise arithmetic, real-time data lookup (without tools), guaranteed-correct outputs, tasks requiring perfect consistency.
3. **What's the cost of being wrong?** If the model produces a bad output, what happens? User sees something weird (low stakes)? Wrong medical advice (high stakes)? Money moves incorrectly (very high stakes)?
4. **Is the user better off with AI or with a simpler tool?** Sometimes a dropdown menu, a search bar, or a rule-based system is better than an LLM. Cheaper, faster, more reliable.

## Good fits for AI

These are task shapes where LLMs consistently add value:

- **Summarization.** Condensing long documents, conversations, or data into shorter forms. LLMs are genuinely good at this.
- **Classification and triage.** Sorting items into categories where the rules are fuzzy or hard to write explicitly. Support tickets, content moderation, intent detection.
- **Draft generation.** Producing a first draft that a human reviews and edits. Emails, reports, documentation, code.
- **Extraction from unstructured text.** Pulling structured data (names, dates, amounts, entities) from free-form text.
- **Conversational interface to a deterministic backend.** "Show me my orders from last month" → SQL query → formatted response. The LLM handles the natural language; the backend handles the data.
- **Translation and adaptation.** Between languages, between formats, between audiences.

## Bad fits for AI

- **Precise calculation.** Use a calculator. LLMs make arithmetic errors.
- **Tasks requiring legal/regulatory compliance on exact output.** If the output must be exactly right every time, LLMs aren't reliable enough.
- **High-volume, low-latency lookup.** A database query is faster and cheaper.
- **Tasks where a simple rule works.** If you can write an `if/else` that handles 95% of cases, do that. Use AI for the remaining 5% if needed.
- **Tasks where one error is catastrophic.** Unless you have human review in the loop.

## The cost-benefit model

For any proposed AI feature, estimate:

**Costs:**
- Engineering time to build, evaluate, and maintain
- Ongoing inference cost (tokens × volume)
- Quality cost (errors reaching users, support burden)
- Opportunity cost (what you're not building instead)

**Benefits:**
- Revenue (new capability, better conversion)
- Efficiency (time saved, headcount avoided)
- Quality (better outputs than the current approach)
- Capability (something that wasn't possible before)

If the benefits don't clearly outweigh the costs, don't build it. "It would be cool" is not a business case.

## Helping the organization say no

The hardest part of this role is often pushing back. "Can we add AI to X?" is a question you'll hear constantly. The right first response is questions, not a yes or no:

- What problem does this solve for the user?
- What happens when the AI is wrong? (It will be, sometimes.)
- How will we know if it's working? (What's the eval?)
- What's the simpler alternative? Have we tried it?
- What's the ongoing cost at expected volume?

Sometimes the answer after these questions is "yes, let's build it." Sometimes it's "actually, a rule-based system would be better." Sometimes it's "the cost of errors is too high without human review." All of these are good outcomes.

The political reality: "AI feature" sells better than "rule engine" in budget discussions, even when the rule engine is the right answer. Part of the PE role is being honest about this tension without being dismissive of organizational incentives.

## The demo trap

A great demo doesn't mean a shippable product. The gap between "works on 5 examples" and "works reliably on the distribution of real inputs" is enormous. Teams that ship based on demos discover this the hard way.

The reverse is also true: dismissing a capability because the first prototype fails is equally wrong. LLM features often need iteration — better prompts, better context, better eval — before they work well. The question is whether the iteration path is plausible, not whether the first attempt succeeds.

## Maintenance cost

AI features rot in ways traditional features don't:
- Model providers deprecate versions
- Model updates change behavior subtly
- User patterns drift away from your eval set
- Prompts accumulate tweaks without regression testing
- The competitive landscape moves (what was impressive last year is table stakes this year)

Budget ongoing maintenance time. An AI feature isn't "done" when it ships — it needs continuous attention to stay good.

## Things that trip people up

**Building AI because it's in the strategy deck.** If the only reason to use AI is "leadership wants AI features," push for specificity. Which features? For which users? Solving which problems?

**Not accounting for eval cost.** Building the feature is half the work. Building the eval harness, maintaining the test set, running continuous quality checks — that's the other half. Budget for it.

**"Let's just try it."** Fine for a prototype. Dangerous as a shipping strategy. "Trying it" without eval means you don't know if it works, and now you have to support it.

**Over-indexing on cost savings.** AI features that save $10K/month in labor but cost $8K/month in inference and $5K/month in engineering time are net negative. Do the math.

## Where things stand

The organizations that are getting the most value from AI are the ones that are specific about where they apply it, rigorous about evaluation, and honest about where it doesn't fit. The ones struggling are the ones that sprinkled AI everywhere without discipline.

As a PE or architect, your job is to bring that discipline. Not to be the person who says no to everything — but to be the person who asks the right questions and helps the organization invest where AI actually helps.

## Go deeper

- [Anthropic's "When to use AI" guidance](https://docs.anthropic.com/en/docs/overview) — practical framing
- Chapter 17 covers AI product sense — what good AI UX looks like once you've decided to build
