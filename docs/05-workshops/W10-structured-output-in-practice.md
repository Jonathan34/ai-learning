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

Use your provider's structured output feature:

```python
import anthropic
import json

client = anthropic.Anthropic()

def extract_ticket(text: str) -> dict:
    response = client.messages.create(
        model="claude-sonnet-4-20250514",
        max_tokens=1024,
        messages=[{"role": "user", "content": f"Extract structured data from this support ticket:\n\n{text}"}],
        tools=[{
            "name": "classify_ticket",
            "description": "Extract structured classification from a support ticket",
            "input_schema": SupportTicket.model_json_schema()
        }],
        tool_choice={"type": "tool", "name": "classify_ticket"}
    )

    # Extract the tool use result
    for block in response.content:
        if block.type == "tool_use":
            return block.input

    raise ValueError("No tool use in response")
```

Run it on your 5 examples. Check:

- Does it parse correctly every time?
- Are the classifications sensible?
- What happens with the ambiguous input?
- What happens with the adversarial input?

---

## Part 3: Add validation and retry (30 min)

Wrap extraction with proper error handling:

```python
from pydantic import ValidationError

def extract_with_retry(text: str, max_retries: int = 2) -> SupportTicket:
    last_error = None

    for attempt in range(max_retries + 1):
        try:
            raw = extract_ticket(text)
            return SupportTicket.model_validate(raw)
        except ValidationError as e:
            last_error = e
            if attempt < max_retries:
                # Feed the error back to the model
                error_msg = f"Previous extraction failed validation:\n{e}\n\nPlease fix and try again."
                raw = extract_ticket(f"{text}\n\n[SYSTEM NOTE: {error_msg}]")
            else:
                raise ValueError(f"Extraction failed after {max_retries + 1} attempts: {last_error}")
```

Test it — force failures by making your Pydantic model stricter (lower max_length, narrower ranges). See how often the retry fixes things.

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

1. **Schema-constrained** (what you built above)
2. **Free-form + parse** — ask the model to "return JSON" in the prompt, then parse
3. **Two-step** — first ask the model to reason about the ticket in free text, then extract from its own reasoning

```python
# Approach 3: Two-step (reason then extract)
def extract_two_step(text: str) -> SupportTicket:
    # Step 1: reason
    reasoning = client.messages.create(
        model="claude-sonnet-4-20250514",
        max_tokens=500,
        messages=[{"role": "user", "content": f"Analyze this support ticket. What category is it? How urgent? What's the customer's tone? What action is needed?\n\n{text}"}]
    ).content[0].text

    # Step 2: extract from reasoning
    raw = extract_ticket(f"Based on this analysis:\n{reasoning}\n\nExtract structured data.")
    return SupportTicket.model_validate(raw)
```

Which approach gives better accuracy? Which is cheaper? Which handles edge cases better?

---

## Gotchas

- **Schema drift.** If you change your Pydantic model, old cached/stored extractions won't match. Version your schemas.

- **The model fills every field even when it shouldn't.** Add explicit nullable fields or an "unknown" enum value for when information is genuinely missing.

- **Pydantic v1 vs v2.** The API changed significantly. Make sure your code matches your installed version (`model_validate` is v2, `parse_obj` is v1).

- **Provider differences in schema support.** Anthropic uses tool use for structured output. OpenAI has `response_format`. Google has `response_schema`. The concepts are the same but the API calls differ.

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
- [Anthropic's tool use docs](https://docs.anthropic.com/en/docs/build-with-claude/tool-use)
- [Workshop W3 (Real Evaluation)](W3-real-evaluation.md) — expanding the eval approach used here
