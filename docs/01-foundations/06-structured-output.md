# Structured Output and Type Safety

The moment you try to use an LLM in a real system — not a chatbot, but an actual backend component — you hit this wall: the model outputs free-form text, and the rest of your codebase expects typed data. Bridging this gap reliably is one of the most under-discussed skills in production AI engineering.

Every time your code does `json.loads(model_output)` and hopes for the best, you're writing fragile software. This chapter is about doing it properly.

## The problem

LLMs generate text. Your system needs structured data — JSON objects matching a schema, enum values from a fixed set, numbers within a range, lists of a specific type. The model doesn't naturally produce these. It produces tokens that are *probably* valid JSON, *probably* within your constraints, *probably* matching your schema.

"Probably" isn't good enough when the output feeds into a database write, an API call, or a user-facing UI component.

## Provider-level structured output

Most providers now support constraining the model's output at the API level:

**JSON mode** — guarantees the output is valid JSON (no markdown wrappers, no trailing text). Doesn't guarantee it matches your schema — just that it parses.

**Schema-constrained output** — you provide a JSON Schema, and the provider restricts which tokens the model can pick so the final answer matches it. Anthropic, OpenAI, and Google all offer this natively, with different API shapes ([Anthropic](https://platform.claude.com/docs/en/build-with-claude/structured-outputs), [OpenAI](https://developers.openai.com/api/docs/guides/structured-outputs), [Google](https://ai.google.dev/gemini-api/docs/structured-output), checked October 2026).

Don't confuse this with tool calling. A tool schema describes the arguments for an action you want your code to run — look up an order, send an email ([chapter 9](../02-agents/09-mcp-and-tools.md) covers that). A structured output schema describes the model's answer itself. All three providers document these as separate features. Whether you can combine them in a single request depends on the provider and model, so check the docs before you rely on it. Google, for example, limits that combination to Gemini 3 models and marks it as a preview ([Gemini docs](https://ai.google.dev/gemini-api/docs/structured-output), checked October 2026). For a while the standard trick with Claude was to define a fake tool and force the model to call it just to get typed data back. You don't need that anymore. If nothing is going to execute, don't declare a tool.

```python
# Anthropic (Claude) — native structured output via the Python SDK
from pydantic import BaseModel

class Person(BaseModel):
    name: str
    age: int

response = client.messages.parse(
    model="claude-sonnet-4-5",  # check the docs for which models support structured outputs
    max_tokens=1024,
    messages=[{"role": "user", "content": "Extract the person's name and age from: 'John is 32 years old'"}],
    output_format=Person,
)
person = response.parsed_output  # a Person instance
```

Without the SDK helper, you pass the schema yourself as `output_config={"format": {"type": "json_schema", "schema": {...}}}` and the JSON arrives as the response text ([Anthropic docs](https://platform.claude.com/docs/en/build-with-claude/structured-outputs)).

```python
# OpenAI — Responses API with a Pydantic model
response = client.responses.parse(
    model="gpt-4o",
    input=[{"role": "user", "content": "Extract the person's name and age from: 'John is 32 years old'"}],
    text_format=Person,
)
person = response.output_parsed
```

If you hand OpenAI a raw JSON Schema instead, set `"strict": true` and `"additionalProperties": false`. Without strict mode, you're back to output that usually matches the schema ([OpenAI docs](https://developers.openai.com/api/docs/guides/structured-outputs)).

Schema-constrained output is the right default for any extraction, classification, or structured generation task. Use it. Just know what it buys you: no more broken JSON, missing required fields, or invented keys. It doesn't buy you correct answers, and you can still get no usable answer at all. More on that below.

## Pydantic and validation layers

In Python, the standard pattern is defining your output types as Pydantic models. With the provider SDK helpers above, the model class does the whole job. The SDK generates the JSON Schema from it, sends it, and parses the response back into a typed object.

You still validate yourself when you're parsing text on your own: an older SDK, a local model, or a provider without native support:

```python
from pydantic import BaseModel, Field

class ExtractedPerson(BaseModel):
    name: str = Field(description="Full name of the person")
    age: int = Field(ge=0, le=150, description="Age in years")
    confidence: float = Field(ge=0.0, le=1.0, description="How confident the extraction is")

# Parse model output
person = ExtractedPerson.model_validate_json(model_response)
```

There's a subtlety even on the native path. Providers enforce only a subset of JSON Schema. Anthropic's Python SDK, for example, removes constraints it can't enforce, like the `ge=0, le=150` bounds above, from the schema it sends. It then checks the parsed result against your original model. A raw schema that includes unsupported constraints can be rejected with a 400 error instead ([Anthropic docs](https://platform.claude.com/docs/en/build-with-claude/structured-outputs), checked October 2026). So the bounds still hold, but the model wasn't forced to respect them while generating. A violation shows up as a validation error, so your code still needs to handle it.

Libraries like `instructor` (built on Pydantic) handle the full loop: schema generation, prompt injection, parsing, validation, and retry on failure.

The key insight: your Pydantic model IS your contract with the LLM. The field names, types, descriptions, and constraints all become part of the prompt (either explicitly or via the schema). Name your fields clearly — `order_status` not `status` — because the model reads those names.

## When structured output fails

Schema constraints prevent the ordinary syntax and shape errors. They don't prevent these:

**No answer at all.** A refusal, a response cut off at the token limit, a schema the provider won't accept, or a plain network failure. Anthropic and OpenAI both document refusals and incomplete responses as cases where the output may not match your schema ([Anthropic](https://platform.claude.com/docs/en/build-with-claude/structured-outputs), [OpenAI](https://developers.openai.com/api/docs/guides/structured-outputs)). Check the stop reason before trusting the result.

**Correct format, wrong content.** The model returns `{"name": "John", "age": 32}` when the text said "John's son is 32" — it extracted the wrong person. Schema validation passes. The data is wrong. Google's guide says it directly: validate the values in your application, because a schema-valid answer can still be semantically wrong ([Gemini docs](https://ai.google.dev/gemini-api/docs/structured-output)). You need content-level checks (eval) on top of format validation.

**Enum hallucination.** You constrain a field to `["approved", "denied", "pending"]` but the model sees ambiguous input and picks one anyway rather than expressing uncertainty. Consider adding an "uncertain" value to your enum, or a confidence field.

**Nested object failures.** Complex schemas with nested arrays of objects are harder for models to produce correctly. The deeper the nesting, the more likely something goes wrong. Flatten when possible.

**Overfit to schema.** The model fills every field even when some aren't applicable. If a field is optional, make it explicitly nullable and tell the model when to use null.

## Patterns that work in production

**Separate extraction from generation.** Don't ask the model to both think and produce structured output in one call. First ask it to reason (free text), then ask a second call to extract structure from its own reasoning. More tokens, more reliable.

**Retry with error feedback.** With native structured output you shouldn't need retries to repair JSON. You still need them when your own validation fails, either a business rule or a constraint the provider couldn't enforce in the schema. Send the error message back to the model: "Your output failed validation: age must be >= 0, got -1. Please fix". Most models correct themselves on the first retry.

**Graceful partial output.** Design your schemas with optional fields. If the model can't extract something, it's better to get `null` than a hallucinated value. Your downstream code should handle missing fields.

**Streaming structured output.** For large structured responses (lists of items), some providers support streaming partial JSON. This lets you process items as they arrive rather than waiting for the full response. Useful for long extractions.

## Classification: the most common use case

The single most common structured output task is classification — assigning one or more labels from a fixed set. This is where LLMs replaced hundreds of bespoke ML models.

```python
class TicketClassification(BaseModel):
    category: Literal["billing", "technical", "account", "other"]
    priority: Literal["low", "medium", "high", "critical"]
    requires_human: bool
    reasoning: str = Field(description="Brief explanation of classification")
```

The `reasoning` field is intentional. Including it improves accuracy (chain-of-thought effect) and gives you debuggability. In production, log the reasoning even if you don't show it to users.

## Things that trip people up

**Putting the schema in the prompt AND using schema-constrained mode.** Pick one. If you're using the provider's structured output feature, the schema is already communicated. Repeating it in the prompt wastes tokens and can create conflicts if they diverge.

**Not testing edge cases in your schema.** What happens with empty input? What about input in a language your model wasn't expecting? What about adversarial input designed to break your schema? Test these explicitly.

**Ignoring the `description` fields in your schema.** These aren't just documentation — the model reads them. A field called `score` with no description is ambiguous. A field called `score` with `description="1-5 rating of response quality, where 5 is perfect"` is clear.

**Using structured output for creative tasks.** If you want the model to write a poem, don't force it into JSON. Structured output is for extraction, classification, and structured generation. Use free text for creative work.

## Where this fits in the stack

Structured output is the bridge layer between the model and everything else:

```
User input → Model (free generation or structured) → Validation → Your typed codebase → DB/API/UI
```

Get this layer right and your LLM integration feels like calling a typed API. Get it wrong and you're debugging `json.loads()` failures at 3 AM.

## Go deeper

- [Anthropic's structured outputs docs](https://platform.claude.com/docs/en/build-with-claude/structured-outputs) — JSON outputs, strict tool use, and the supported schema subset
- [OpenAI's structured outputs guide](https://developers.openai.com/api/docs/guides/structured-outputs) — JSON Schema constrained generation, and when to use function calling instead
- [Gemini structured output guide](https://ai.google.dev/gemini-api/docs/structured-output) — Google's version, with the same split between response format and function calling
- [Instructor library](https://github.com/jxnl/instructor) — Pydantic-based structured output for multiple providers
- [Outlines](https://github.com/dottxt-ai/outlines) — constrained generation for local models
