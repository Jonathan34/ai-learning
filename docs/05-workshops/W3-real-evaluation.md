# Workshop W3 — Real Evaluation

**Goal:** Build an evaluation harness for an LLM task, including an LLM-as-judge component and a regression test suite.

**Time:** 3-4 hours

**Prerequisites:** Python, an LLM API (hosted or local), ideally the RAG system from W2

---

## What you're building

An eval harness that can answer: "Did this change make things better or worse?" You'll build:
1. A golden test set with expected properties
2. Automated scorers (structural checks + LLM-as-judge)
3. A regression runner that compares two versions

---

## Part 1: Pick a task and build a test set (45 min)

Choose one of:
- Your RAG system from W2 (evaluate answer quality)
- A classification task (evaluate accuracy)
- A summarization task (evaluate completeness and conciseness)

Build 20-30 test cases. Each case needs:
- An input (the question or text to process)
- Expected properties (not necessarily an exact answer — properties you can check)

```python
test_cases = [
    {
        "input": "What is the return policy for electronics?",
        "expected_properties": {
            "mentions_timeframe": True,       # should mention "30 days" or similar
            "mentions_condition": True,        # should mention item condition requirement
            "is_json": False,                  # not expecting JSON output
            "max_length_words": 200,           # shouldn't be a novel
            "should_cite_source": True,        # should reference a document
        },
        "reference_answer": "Electronics can be returned within 30 days if unopened and in original packaging.",
        "difficulty": "easy"
    },
    # ... 19-29 more cases covering easy, medium, and hard
]
```

Include cases where the correct answer is "I don't know" — these test whether your system hallucinates.

---

## Part 2: Structural scorers (30 min)

These are cheap, deterministic checks. Run them first on every output.

```python
def score_structural(output, expected_properties):
    results = {}
    
    # Length check
    word_count = len(output.split())
    if "max_length_words" in expected_properties:
        results["length_ok"] = word_count <= expected_properties["max_length_words"]
    
    # Format check
    if expected_properties.get("is_json"):
        try:
            json.loads(output)
            results["valid_json"] = True
        except:
            results["valid_json"] = False
    
    # Contains expected content
    if expected_properties.get("mentions_timeframe"):
        results["mentions_timeframe"] = any(
            term in output.lower() 
            for term in ["30 days", "thirty days", "one month", "within a month"]
        )
    
    # Doesn't contain forbidden content
    if expected_properties.get("should_not_contain"):
        for forbidden in expected_properties["should_not_contain"]:
            results[f"no_{forbidden}"] = forbidden.lower() not in output.lower()
    
    return results
```

These catch obvious failures fast and free. A response that isn't valid JSON when it should be, or that's 2000 words when it should be 200, is clearly broken regardless of content quality.

---

## Part 3: LLM-as-judge (45 min)

For quality dimensions that can't be checked structurally (helpfulness, accuracy, tone), use another LLM as a judge.

```python
def judge_response(question, response, reference_answer, judge_model="claude-sonnet-4-20250514"):
    judge_prompt = f"""You are evaluating an AI assistant's response. 
    
Question asked: {question}

Reference answer (what a correct response should convey): {reference_answer}

Actual response to evaluate: {response}

Evaluate on these dimensions. For each, answer Yes or No and give a one-sentence reason.

1. ACCURATE: Does the response convey the same key facts as the reference answer?
2. COMPLETE: Does it cover the main points without major omissions?
3. GROUNDED: Does it only state things supported by the reference, without making things up?
4. CONCISE: Is it appropriately brief without unnecessary filler?

Respond in JSON format:
{{"accurate": {{"answer": "Yes/No", "reason": "..."}}, "complete": ...}}"""

    # Call the judge model
    result = client.chat.completions.create(
        model=judge_model,
        messages=[{"role": "user", "content": judge_prompt}]
    )
    return json.loads(result.choices[0].message.content)
```

Key decisions:
- **Use a different model as the judge** than the one you're evaluating. Same-model judging is biased.
- **Ask specific Yes/No questions**, not "rate 1-10." Specific questions give more reliable results.
- **Include the reference answer** so the judge has something to compare against.
- **Ask for reasons** — they help you debug when the judge disagrees with your intuition.

---

## Part 4: Run the full eval (30 min)

```python
def run_eval(system_under_test, test_cases):
    results = []
    for case in test_cases:
        # Get the system's response
        output = system_under_test(case["input"])
        
        # Structural scores
        structural = score_structural(output, case["expected_properties"])
        
        # LLM judge scores
        judge = judge_response(
            case["input"], output, case["reference_answer"]
        )
        
        results.append({
            "input": case["input"],
            "output": output,
            "structural": structural,
            "judge": judge,
            "difficulty": case.get("difficulty", "unknown")
        })
    
    return results

# Run it
results = run_eval(my_rag_system, test_cases)

# Summarize
structural_pass_rate = sum(
    all(v for v in r["structural"].values()) 
    for r in results
) / len(results)

accuracy_rate = sum(
    r["judge"]["accurate"]["answer"] == "Yes" 
    for r in results
) / len(results)

print(f"Structural pass rate: {structural_pass_rate:.0%}")
print(f"Accuracy rate: {accuracy_rate:.0%}")
```

---

## Part 5: Regression testing (30 min)

Now the real value: comparing two versions.

```python
def compare_versions(version_a, version_b, test_cases):
    results_a = run_eval(version_a, test_cases)
    results_b = run_eval(version_b, test_cases)
    
    print("Case-by-case comparison:")
    regressions = 0
    improvements = 0
    
    for i, (ra, rb) in enumerate(zip(results_a, results_b)):
        a_accurate = ra["judge"]["accurate"]["answer"] == "Yes"
        b_accurate = rb["judge"]["accurate"]["answer"] == "Yes"
        
        if a_accurate and not b_accurate:
            regressions += 1
            print(f"  REGRESSION on case {i}: {test_cases[i]['input'][:50]}...")
        elif not a_accurate and b_accurate:
            improvements += 1
            print(f"  IMPROVEMENT on case {i}: {test_cases[i]['input'][:50]}...")
    
    print(f"\nSummary: {improvements} improvements, {regressions} regressions")
    if regressions > 0:
        print("⚠️  Version B has regressions. Review before shipping.")
```

Now change something — your prompt, your chunk size, your model — and run the comparison. This is how you make informed decisions about changes instead of guessing.

---

## Part 6: Calibrate the judge (30 min)

How do you know your LLM judge is trustworthy? Compare it against your own judgment.

Take 20 of your test cases. Read the outputs yourself. Rate them (accurate? yes/no). Then compare your ratings to the judge's ratings.

```python
# Your manual ratings
my_ratings = {0: True, 1: True, 2: False, 3: True, ...}  # case_index: is_accurate

# Compare
agreements = sum(
    my_ratings[i] == (results[i]["judge"]["accurate"]["answer"] == "Yes")
    for i in my_ratings
)
agreement_rate = agreements / len(my_ratings)
print(f"Judge agrees with me {agreement_rate:.0%} of the time")
```

If agreement is above 85%, your judge is probably reliable enough for automated use. If it's below 75%, you need to improve the judge prompt or use a better judge model.

---

## Gotchas

- **LLM-as-judge is not free.** Each judgment is an LLM call. For 30 test cases with 4 dimensions each, that's 30 judge calls. Budget for it.
- **Position bias in pairwise comparison.** If you compare two responses ("which is better, A or B?"), the judge tends to prefer whichever comes first. Randomize order.
- **The eval set ages.** Your test cases from today may not represent user queries in 3 months. Refresh periodically with real production examples.
- **Overfitting to the eval set.** If you tune your prompt until it aces all 30 cases, you've probably overfit. Hold 10 cases back for validation.

---

## What you should have after this workshop

- A test set of 20-30 cases with expected properties
- Structural scorers that catch obvious failures
- An LLM-as-judge that scores quality dimensions
- A regression comparison tool
- Calibration data showing how reliable your judge is
- The ability to answer "did this change help or hurt?" before shipping

## Go deeper

- [Promptfoo](https://www.promptfoo.dev/) — CLI tool that does much of this with less custom code
- [Braintrust](https://www.braintrust.dev/) — hosted eval platform
- [Chapter 04 (Evaluation)](../01-foundations/04-evaluation.md) covers the theory behind what you just built
