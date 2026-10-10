# Workshop W10 — Structured Output in Practice

**Goal:** Build a production-quality extraction pipeline that takes messy real-world input and produces validated, typed data. Understand where structured output breaks and how to make it robust.

**Time:** 2-3 hours

**Prerequisites:** Python, an LLM API (Claude or OpenAI), familiarity with Pydantic

---

## What you're building

A pipeline that:

1. Takes unstructured text (emails, support tickets, meeting notes)
2. Extracts structured data using an LLM with schema constraints
3. Validates and retries on failure
4. Handles edge cases gracefully
5. Measures extraction accuracy

This is the most common production LLM pattern that isn't a chatbot — and the one most teams get wrong.

---

## Part 1: Define your extraction target (20 min)

Pick a real extraction task. Good options:

- **Support tickets** → category, priority, customer sentiment, action items
- **Meeting notes** → attendees, decisions made, action items with owners, follow-up date
- **Product reviews** → sentiment, feature mentions, specific complaints, would-recommend

Define your Pydantic model:

```python
from pydantic import BaseModel, Field
from typing import Literal, Optional
from datetime import date

class SupportTicket(BaseModel):
    category: Literal["billing", "technical", "account", "shipping", "other"]
    priority: Literal["low", "medium", "high", "critical"]
    sentiment: Literal["frustrated", "neutral", "positive"]
    summary: str = Field(max_length=200, description="One-sentence summary of the issue")
    action_required: str = Field(description="What needs to happen next")
    requires_human: bool = Field(description="True if this can't be resolved automatically")
    confidence: float = Field(ge=0.0, le=1.0, description="How confident the extraction is, 0 to 1")
```

Write 5 example inputs by hand — vary the difficulty. Include one that's ambiguous, one that's missing information, and one that's adversarial (prompt injection attempt disguised as a ticket).

---

## Part 2: Basic extraction with schema constraints (30 min)

Use your provider's native structured output feature. That constrains the model's final answer to your schema. Don't wrap the schema in a fake tool and force the model to call it. Tool schemas are for actions your code runs, and Claude now supports schema-constrained answers directly ([Anthropic structured outputs docs](https://platform.claude.com/docs/en/build-with-claude/structured-outputs), checked October 2026).

```python
import anthropic

client = anthropic.Anthropic()
MODEL = "claude-sonnet-4-5"  # check the docs for which models support structured outputs

# The schema Claude generates against. It mirrors SupportTicket but leaves out
# the length and numeric bounds; SupportTicket checks those after the response arrives.
TICKET_SCHEMA = {
    "type": "object",
    "properties": {
        "category": {"type": "string", "enum": ["billing", "technical", "account", "shipping", "other"]},
        "priority": {"type": "string", "enum": ["low", "medium", "high", "critical"]},
        "sentiment": {"type": "string", "enum": ["frustrated", "neutral", "positive"]},
        "summary": {"type": "string", "description": "One-sentence summary of the issue"},
        "action_required": {"type": "string", "description": "What needs to happen next"},
        "requires_human": {"type": "boolean", "description": "True if this can't be resolved automatically"},
        "confidence": {"type": "number", "description": "How confident the extraction is, 0 to 1"},
    },
    "required": ["category", "priority", "sentiment", "summary", "action_required", "requires_human", "confidence"],
    "additionalProperties": False,
}

class ExtractionError(Exception):
    pass

def extract_ticket(text: str, note: str = "", max_tokens: int = 1024) -> SupportTicket:
    prompt = f"Extract structured data from this support ticket:\n\n{text}"
    if note:
        prompt += f"\n\n{note}"

    response = client.messages.create(
        model=MODEL,
        max_tokens=max_tokens,
        messages=[{"role": "user", "content": prompt}],
        output_config={"format": {"type": "json_schema", "schema": TICKET_SCHEMA}},
    )

    # Refusals and truncation are the cases where the output may not match the schema,
    # so check before parsing anything
    if response.stop_reason in ("refusal", "max_tokens"):
        raise ExtractionError(f"No usable output (stop_reason={response.stop_reason})")

    # Raises pydantic.ValidationError if a bound the schema left out is violated
    return SupportTicket.model_validate_json(response.content[0].text)
```

This uses the raw `output_config.format` request so you can look at `stop_reason` before anything tries to parse. The SDK also has a `client.messages.parse(..., output_format=SupportTicket)` helper that builds the schema from your Pydantic model and returns `response.parsed_output` ([Anthropic docs](https://platform.claude.com/docs/en/build-with-claude/structured-outputs), checked October 2026). It's shorter, but here you want to see refusals and truncation as their own cases, so you need the stop reason first.

Claude only enforces a subset of JSON Schema, and a raw schema with unsupported constraints can be rejected with a 400 error. That's why `TICKET_SCHEMA` leaves out the `max_length` on `summary` and the `ge`/`le` bounds on `confidence`. `SupportTicket` still checks them, but only after the response arrives. The model wasn't forced to respect them while it was generating. Keeping the two definitions in step is the cost of the raw path, so treat drift between them as a bug.

Run it on your 5 examples. Check:

- Does any call fail outright? Malformed JSON should be rare now. When a call fails, check whether it was a refusal, truncation, or a `ValidationError` on one of the bounds.
- Are the classifications sensible? This is where the real errors live. A schema-valid answer can still be wrong.
- What happens with the ambiguous input?
- What happens with the adversarial input?

---

## Part 3: Add validation and retry (30 min)

With native structured output, you shouldn't need retries to repair broken JSON. Retries still earn their place for truncation, where you try again with a bigger token budget, for bounds the schema left out, for business rules the schema can't express, and for transient API errors. A refusal usually isn't worth retrying with the same input.

```python
from pydantic import ValidationError

def business_problems(ticket: SupportTicket) -> list[str]:
    problems = []
    if ticket.priority == "critical" and not ticket.requires_human:
        problems.append("priority is 'critical' but requires_human is False")
    # Add the rules your own pipeline cares about
    return problems

def extract_with_retry(text: str, max_retries: int = 2, max_tokens: int = 1024) -> SupportTicket:
    note = ""
    last_error = None

    for attempt in range(max_retries + 1):
        try:
            ticket = extract_ticket(text, note=note, max_tokens=max_tokens)
        except ExtractionError as e:
            last_error = e
            if "max_tokens" not in str(e):
                raise  # a refusal won't change on a blind retry
            max_tokens *= 2
            continue
        except ValidationError as e:
            # A bound the schema couldn't enforce, like confidence > 1
            last_error = e
            note = f"A previous extraction failed validation: {e}. Fix the fields it names."
            continue

        problems = business_problems(ticket)
        if not problems:
            return ticket

        # Feed the problems back into the next attempt
        last_error = "; ".join(problems)
        note = f"A previous extraction was inconsistent: {last_error}. Re-read the ticket and correct whichever field is wrong."

    raise ExtractionError(f"Extraction failed after {max_retries + 1} attempts: {last_error}")
```

Test it. Write a ticket that tempts the model into an inconsistent answer, like a full outage described in a calm, polite tone. Then call it with a very low `max_tokens` to trigger truncation. See how often the retry fixes things.

---

## Part 4: Handle the hard cases (30 min)

Real data has edge cases. Build explicit handling:

```python
class ExtractionResult(BaseModel):
    ticket: Optional[SupportTicket] = None
    extraction_failed: bool = False
    failure_reason: Optional[str] = None
    raw_text_length: int
    processing_time_ms: float

def extract_safe(text: str) -> ExtractionResult:
    import time
    start = time.time()

    # Guard: empty or too-short input
    if len(text.strip()) < 10:
        return ExtractionResult(
            extraction_failed=True,
            failure_reason="Input too short for meaningful extraction",
            raw_text_length=len(text),
            processing_time_ms=(time.time() - start) * 1000
        )

    # Guard: input too long (would blow context)
    if len(text) > 10000:
        text = text[:10000]  # truncate with a note

    try:
        ticket = extract_with_retry(text)
        return ExtractionResult(
            ticket=ticket,
            raw_text_length=len(text),
            processing_time_ms=(time.time() - start) * 1000
        )
    except Exception as e:
        return ExtractionResult(
            extraction_failed=True,
            failure_reason=str(e),
            raw_text_length=len(text),
            processing_time_ms=(time.time() - start) * 1000
        )
```

Now your extraction never crashes your pipeline. It returns typed results — either the extraction succeeded or it failed with a reason. Your downstream code can handle both cases.

---

## Part 5: Build an eval for extraction quality (30 min)

Create a golden set — inputs where you know the correct output:

```python
test_cases = [
    {
        "input": "My order hasn't arrived and it's been 2 weeks. Order #12345. I'm really frustrated.",
        "expected": {
            "category": "shipping",
            "priority": "high",
            "sentiment": "frustrated",
            "requires_human": False
        }
    },
    {
        "input": "Hey just wondering if you have a student discount?",
        "expected": {
            "category": "billing",
            "priority": "low",
            "sentiment": "neutral",
            "requires_human": False
        }
    },
    # Add 8-10 more covering edge cases
]

def evaluate_extraction(test_cases):
    results = {"total": len(test_cases), "correct": 0, "field_accuracy": {}}

    for case in test_cases:
        result = extract_safe(case["input"])
        if result.ticket is None:
            continue

        all_correct = True
        for field, expected_value in case["expected"].items():
            actual = getattr(result.ticket, field)
            correct = actual == expected_value
            if field not in results["field_accuracy"]:
                results["field_accuracy"][field] = {"correct": 0, "total": 0}
            results["field_accuracy"][field]["total"] += 1
            if correct:
                results["field_accuracy"][field]["correct"] += 1
            else:
                all_correct = False
                print(f"  MISMATCH on '{field}': expected {expected_value}, got {actual}")

        if all_correct:
            results["correct"] += 1

    return results
```

Run it. Which fields does the model get wrong most often? That tells you where to improve your schema descriptions or add examples.

---

## Part 6: Compare approaches (30 min)

Try the same extraction three ways and compare:

1. **Schema-constrained** (native structured output, what you built above)
2. **Free-form + parse** — ask the model to "return JSON" in the prompt, then parse
3. **Two-step** — first ask the model to reason about the ticket in free text, then extract from its own reasoning

```python
# Approach 3: Two-step (reason then extract)
def extract_two_step(text: str) -> SupportTicket:
    # Step 1: reason
    reasoning = client.messages.create(
        model=MODEL,
        max_tokens=500,
        messages=[{"role": "user", "content": f"Analyze this support ticket. What category is it? How urgent? What's the customer's tone? What action is needed?\n\n{text}"}]
    ).content[0].text

    # Step 2: extract from reasoning with native structured output
    return extract_ticket(text, note=f"Use this analysis of the ticket:\n{reasoning}")
```

Which approach gives better accuracy? Which is cheaper? Which handles edge cases better?

---

## Gotchas

- **Schema drift.** If you change your Pydantic model, old cached/stored extractions won't match. Version your schemas.

- **The model fills every field even when it shouldn't.** Add explicit nullable fields or an "unknown" enum value for when information is genuinely missing.

- **Pydantic v1 vs v2.** The API changed significantly. Make sure your code matches your installed version (`model_validate` is v2, `parse_obj` is v1).

- **Provider differences in API shape.** Anthropic, OpenAI, and Google all offer native schema-constrained responses. The parameter names and SDK helpers differ, and so do the parts of JSON Schema each one supports ([Anthropic](https://platform.claude.com/docs/en/build-with-claude/structured-outputs), [OpenAI](https://developers.openai.com/api/docs/guides/structured-outputs), [Google](https://ai.google.dev/gemini-api/docs/structured-output), checked October 2026). All three treat tool or function calling as a separate contract, used when the model asks your code to take an action. Older tutorials that force a tool call to get typed output predate native support with Claude.

- **Schema-valid isn't the same as correct.** Constrained output stops malformed JSON. It doesn't stop a wrong category or a made-up summary. That's what Part 5 is for.

---

## What you should have after this workshop

- A production-quality extraction pipeline with validation and retry
- Understanding of where structured output breaks (and how to detect it)
- A golden set eval for measuring extraction accuracy
- An informed comparison of schema-constrained vs free-form vs two-step approaches

This pattern — extract, validate, handle failure, measure — is the backbone of every non-chatbot LLM integration.

## Go deeper

- [Structured Output and Type Safety](../01-foundations/06-structured-output.md) — the theory behind what you just built
- [Instructor library](https://github.com/jxnl/instructor) — production-grade structured output with retries
- [Anthropic's structured outputs docs](https://platform.claude.com/docs/en/build-with-claude/structured-outputs) — `messages.parse`, `output_config.format`, and the supported schema subset
- [Workshop W3 (Real Evaluation)](W3-real-evaluation.md) — expanding the eval approach used here
