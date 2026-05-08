# Structured Output and Type Safety

The moment you try to use an LLM in a real system — not a chatbot, but an actual backend component — you hit this wall: the model outputs free-form text, and the rest of your codebase expects typed data. Bridging this gap reliably is one of the most under-discussed skills in production AI engineering.

Every time your code does `json.loads(model_output)` and hopes for the best, you're writing fragile software. This chapter is about doing it properly.

## The problem

LLMs generate text. Your system needs structured data — JSON objects matching a schema, enum values from a fixed set, numbers within a range, lists of a specific type. The model doesn't naturally produce these. It produces tokens that are *probably* valid JSON, *probably* within your constraints, *probably* matching your schema.

"Probably" isn't good enough when the output feeds into a database write, an API call, or a user-facing UI component.

## Provider-level structured output

Most providers now support constraining the model's output at the API level:

**JSON mode** — guarantees the output is valid JSON (no markdown wrappers, no trailing text). Doesn't guarantee it matches your schema — just that it parses.

**Schema-constrained output** — you provide a JSON Schema, and the model is constrained to only produce outputs that validate against it. Anthropic, OpenAI, and Google all support this with slightly different APIs.

```python
# Anthropic (Claude) — tool use as structured output
response = client.messages.create(
    model="claude-sonnet-4-20250514",
    messages=[{"role": "user", "content": "Extract the person's name and age from: 'John is 32 years old'"}],
    tools=[{
        "name": "extract_person",
        "description": "Extract structured person data",
        "input_schema": {
            "type": "object",
            "properties": {
                "name": {"type": "string"},
                "age": {"type": "integer", "minimum": 0, "maximum": 150}
            },
            "required": ["name", "age"]
        }
    }],
    tool_choice={"type": "tool", "name": "extract_person"}
)
```

```python
# OpenAI — response_format with JSON Schema
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": "Extract the person's name and age from: 'John is 32 years old'"}],
    response_format={
        "type": "json_schema",
        "json_schema": {
            "name": "person",
            "schema": {
                "type": "object",
                "properties": {
                    "name": {"type": "string"},
                    "age": {"type": "integer"}
                },
                "required": ["name", "age"]
            }
        }
    }
)
```

Schema-constrained output is the right default for any extraction, classification, or structured generation task. Use it.

## Pydantic and validation layers

In Python, the standard pattern is defining your output types as Pydantic models and validating model output against them:

```python
from pydantic import BaseModel, Field

class ExtractedPerson(BaseModel):
    name: str = Field(description="Full name of the person")
    age: int = Field(ge=0, le=150, description="Age in years")
    confidence: float = Field(ge=0.0, le=1.0, description="How confident the extraction is")

# Parse model output
raw_json = json.loads(model_response)
person = ExtractedPerson.model_validate(raw_json)
```

Libraries like `instructor` (built on Pydantic) handle the full loop: schema generation, prompt injection, parsing, validation, and retry on failure.

The key insight: your Pydantic model IS your contract with the LLM. The field names, types, descriptions, and constraints all become part of the prompt (either explicitly or via the schema). Name your fields clearly — `order_status` not `status` — because the model reads those names.

## When structured output fails

Even with schema constraints, things break:

**Correct format, wrong content.** The model returns `{"name": "John", "age": 32}` when the text said "John's son is 32" — it extracted the wrong person. Schema validation passes. The data is wrong. You need content-level checks (eval) on top of format validation.

**Enum hallucination.** You constrain a field to `["approved", "denied", "pending"]` but the model sees ambiguous input and picks one anyway rather than expressing uncertainty. Consider adding an "uncertain" value to your enum, or a confidence field.

**Nested object failures.** Complex schemas with nested arrays of objects are harder for models to produce correctly. The deeper the nesting, the more likely something goes wrong. Flatten when possible.

**Overfit to schema.** The model fills every field even when some aren't applicable. If a field is optional, make it explicitly nullable and tell the model when to use null.

## Patterns that work in production

**Separate extraction from generation.** Don't ask the model to both think and produce structured output in one call. First ask it to reason (free text), then ask a second call to extract structure from its own reasoning. More tokens, more reliable.

**Retry with error feedback.** When validation fails, send the error message back to the model: "Your output failed validation: age must be >= 0, got -1. Please fix". Most models correct themselves on the first retry.

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

- [Anthropic's tool use docs](https://docs.anthropic.com/en/docs/build-with-claude/tool-use) — tool use as structured output
- [OpenAI's structured outputs guide](https://platform.openai.com/docs/guides/structured-outputs) — JSON Schema constrained generation
- [Instructor library](https://github.com/jxnl/instructor) — Pydantic-based structured output for multiple providers
- [Outlines](https://github.com/dottxt-ai/outlines) — constrained generation for local models
