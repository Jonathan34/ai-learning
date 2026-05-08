# We've Built the Prefrontal Cortex. Now Build the Rest.

*A neurology-inspired take on AI agent safety — why the brain's architecture might be the best blueprint we have for building agents that don't do catastrophic things.*

---

## The short version

Human brains aren't reliable. They work because they're wrapped in fast internal checks, learned habits, and other people watching. AI agents might need the same layered approach — not a single perfect model, but multiple systems at different speeds, shaped by experience, with external checks that assume any single part will sometimes be wrong.

This isn't about making models smarter. It's about building the *rest of the system* around them.

## Why this matters for architects

If you're building production agent systems, you're already making decisions this essay is about:

- **When should an agent pause vs. proceed?** The brain has a built-in threat detector (amygdala) that fires before conscious reasoning. Most agents don't have an equivalent — they plan and act with the same system.

- **How should stakes affect behavior?** Editing a scratch file and running a query against production should trigger different levels of caution. Most agent setups treat them the same.

- **Is self-checking enough?** A model re-checking its own output shares the same blind spots as the original plan. The brain uses *separate systems* — and society adds external checks on top of those.

- **Where does "learned caution" come from?** Humans develop impulse control through years of practice in low-stakes situations. Agents are trained mostly on "did the task get done" — not on "did you stop when you should have."

## The five directions

The essay proposes five architectural ideas borrowed from neuroscience:

1. **A second system, not a smarter first system.** A fast, cheap, separate model whose only job is to fire an alarm when an action looks dangerous. Different from the same model double-checking itself.

2. **Anticipation as a mandatory step.** Before any action, produce a concrete prediction of the resulting state — as a checkable artifact, not just internal reasoning.

3. **Stakes-proportional caution.** How carefully the agent thinks should scale with how bad it would be to be wrong.

4. **Memory weighted by consequence.** Persistent memory tagged by how bad the outcome was, not just what happened. Biases future plans the way scars bias careful engineers.

5. **Training to pause.** Deliberately building "did you stop when you should have" into training, the way we teach children impulse control through small, safe moments.

## Where the industry is

This isn't purely speculative — pieces are already being built:

- Anthropic's [constitutional classifiers](https://alignment.anthropic.com/2025/cheap-monitors/) — cheap, separate safety checks
- OpenAI's [deliberative alignment](https://openai.com/index/deliberative-alignment/) — model reasoning about safety before acting
- Permission systems and tool-level gates around agent actions
- Responsible Scaling Policies that tie safeguards to capability thresholds

The essay argues these are all good — and that the full picture needs all of them working together, not any single approach winning.

## Read the full essay

The complete piece with neuroscience context, worked examples, and connections to existing safety research:

**[Read on Medium →](https://medium.com/@jonathan.delfour/weve-built-the-prefrontal-cortex-now-build-the-rest-f1eedee9f986)**

---

*This perspective informs how this curriculum approaches agent safety (Chapter 05), agent architecture (Chapter 07), and the "Choosing the right level" section in Chapter 17. If you're designing agent systems with real-world tool access, the layered-safety mindset is the one that works.*
