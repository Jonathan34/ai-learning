# Deployment Patterns

How you deploy an AI system determines its latency, cost, compliance posture, and reliability. "We'll just use the OpenAI API" is a deployment decision with real consequences that most teams don't think through until they hit a wall.

## The main options

### Hosted API (OpenAI, Anthropic, Google directly)

You call their API. They run the model. Simplest to start.

| | |
|---|---|
| Good for | Getting started fast, access to frontier models, no infrastructure to manage |
| Watch out for | Data leaves your perimeter, rate limits at scale, vendor lock-in, outages you can't control |

### Cloud provider managed (Amazon Bedrock, Google Vertex AI, Azure OpenAI)

Same models, but accessed through your cloud provider. The provider handles the relationship with the model vendor.

| | |
|---|---|
| Good for | Easier compliance (data stays in your cloud account), consolidated billing, sometimes better rate limits, VPC integration |
| Watch out for | Slightly different feature sets than calling the model vendor directly, sometimes lagging on newest model versions |

### Self-hosted open-weights models

You run the model yourself on your own hardware (or rented GPUs). Models like Llama, Mistral, Qwen, Nemotron.

| | |
|---|---|
| Good for | Full data control, no per-token cost (just infrastructure), offline/air-gapped requirements, customization (fine-tuning) |
| Watch out for | Significant ops burden, GPU procurement, model quality lags frontier by 6-18 months, you own uptime |

### Hybrid

Different models for different use cases. Common pattern: frontier hosted model for complex tasks, self-hosted small model for simple/high-volume tasks, cloud-managed for anything with compliance requirements.

## Decision factors

| Factor | Points toward hosted | Points toward self-hosted |
|---|---|---|
| Data sensitivity | Cloud-managed (stays in your account) | Self-hosted (never leaves your infra) |
| Latency | Hosted (optimized infrastructure) | Self-hosted (no network hop to provider) |
| Scale | Hosted at low volume | Self-hosted at very high volume |
| Compliance | Cloud-managed (certifications) | Self-hosted (full control) |
| Model quality | Hosted (frontier models) | Self-hosted lags 6-18 months |
| Offline requirement | — | Self-hosted (only option) |
| Budget | Hosted (no upfront cost) | Self-hosted (lower marginal cost at scale) |

Most teams start with hosted APIs and only move to self-hosted when they have a specific reason (cost at scale, data sensitivity, offline requirement, or need for fine-tuning).

## Operational patterns

### Multi-region

If your users are global, latency to a single region matters. Options:

- Deploy in multiple regions (if self-hosted)

- Use a provider with global endpoints

- Accept the latency for non-interactive workloads

### Fallback across providers

When your primary provider has an outage (it happens), fall back to another. Sounds simple. In practice:

- Different models behave differently — your prompts may need adjustment

- Different rate limits and pricing

- Different feature support (tool use formats, streaming behavior)

- Testing the fallback path is essential — don't discover it's broken during an outage

### Request queuing

When you hit rate limits or capacity constraints, queue requests rather than failing. Important for batch workloads and traffic spikes.

### Caching

Three levels:

- **Prompt caching** (provider-side) — caches computation for repeated prompt prefixes. Automatic with most providers.

- **Response caching** (your side) — if the exact same input produces the same output, cache it. Works for deterministic queries (temperature 0, same input).

- **Semantic caching** — cache responses for queries that are similar (not identical) to previous ones. More complex, higher risk of serving stale/wrong answers.

### Circuit breakers and graceful degradation

When the AI system is down or degraded:

- Fall back to a simpler model

- Fall back to a rule-based system

- Show a "temporarily unavailable" message

- Queue the request for later processing

Don't let an AI outage take down your entire application. The AI feature should degrade gracefully, not catastrophically.

## Model lifecycle

### Version pinning

Pin your model version in production. Don't use "latest" — model updates can change behavior in ways that break your prompts.

```
# Bad
model = "claude-sonnet"  # could change any time

# Good  
model = "claude-sonnet-4-20250514"  # specific version
```

### Upgrade process

When you want to upgrade to a newer model:

1. Run your eval harness on the new model with your existing prompts

2. Check for regressions

3. If regressions exist, adjust prompts and re-eval

4. Canary deploy (small % of traffic) and monitor

5. Full rollout once confident

### Deprecation planning

Providers deprecate old models. You'll get notice (usually months), but you need a plan:

- Track which models you're using where

- Have your eval harness ready to test replacements

- Budget time for prompt adjustments

## Things that trip people up

**"Just use OpenAI" without thinking about governance.** Fine for a prototype. In production, someone will ask: "Where does our customer data go? What's their data retention policy? Are they SOC 2 compliant? What happens during an outage?" Have answers.

**Self-hosting is harder than blog posts make it sound.** GPU procurement, driver management, model serving infrastructure, load balancing, monitoring, on-call, capacity planning. It's a real ops commitment.

**Multi-provider fallback is a quality risk.** Claude and GPT-4 behave differently. A prompt tuned for one may produce worse results on the other. Test your fallback path with your actual prompts.

**Not planning for model deprecation.** The model you're using today will be deprecated eventually. If your system is tightly coupled to one specific model version with no eval harness, the deprecation notice becomes a fire drill.

**Ignoring cold start for self-hosted.** Loading a large model into GPU memory takes seconds to minutes. If your system scales to zero and back, users experience that delay. Keep models warm or accept the cold start cost.

## Where things stand

For most teams in 2025: start with a hosted API (directly or through your cloud provider). Move to self-hosted only when you have a compelling reason. The operational burden of self-hosting is real and often underestimated.

The hybrid pattern (hosted frontier for complex tasks, self-hosted small model for simple/high-volume tasks) is increasingly common and often the best balance of cost, quality, and operational complexity.

## Go deeper

- [Amazon Bedrock documentation](https://docs.aws.amazon.com/bedrock/) — cloud-managed multi-model access

- [vLLM documentation](https://docs.vllm.ai/) — the standard for self-hosted GPU inference

- [LiteLLM](https://github.com/BerriAI/litellm) — unified interface across providers with fallback support

- [Chapter 14](14-local-and-edge.md) covers local/edge inference specifically
