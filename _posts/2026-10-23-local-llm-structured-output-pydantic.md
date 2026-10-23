---
layout: single
permalink: /ai-engineering/day-2-structured-extraction-pydantic/
toc: true
toc_label: "Day 2 guide"
title: "Building Reliable Local LLM Outputs with Ollama and Pydantic"
date: 2026-10-23
categories:
  - AI Engineering
tags:
  - Ollama
  - Python
  - Pydantic
  - Structured Output
  - Local LLM
excerpt: "A practical Day 2 lab: turn incident logs into validated JSON, understand schema failures, and design bounded retries with a local LLM."
read_time: true
---

> **AI Engineering Learning Journal — Day 2**  
> **Status:** Core structured-output extraction tested successfully; retry failure-path exercise demonstrated in code but not yet verified in a recorded run.
{: .journal-callout .journal-callout--info}

On Day 1, I explored local inference with Ollama: warm versus cold starts, token throughput, output limits, and temperature. On Day 2, I moved to a different engineering challenge: **how do we turn probabilistic LLM responses into data an application can safely consume?**

The lab uses a familiar SRE scenario: extract an incident summary from an application log, then validate the result before passing it to another system.

## The problem: plausible text isn't a reliable interface

A language model might produce a readable incident summary, but automation needs predictable fields and types. Even valid JSON can contain missing fields, unsupported severity values, or invalid confidence numbers.

Our pipeline is:

```text
Application log
     |
     v
Ollama local model (JSON mode)
     |
     v
Pydantic schema validation
     |
     +--> Valid: use structured Incident object
     |
     +--> Invalid: return validation feedback and retry (bounded)
```

**JSON syntax, schema validity, and factual accuracy are three different guarantees.** The first two can be checked mechanically; the third requires evidence and domain-specific checks.
{: .journal-callout .journal-callout--caution}

## Environment

- Apple Silicon Mac Mini, 16 GB unified memory
- Python 3.12 in a virtual environment
- Ollama running locally at `http://localhost:11434`
- `requests` and Pydantic 2
- Original lab model: `qwen2.5vl:3b`

Install the Python dependencies in your existing environment:

```bash
python3.12 -m pip install requests 'pydantic>=2,<3'
```

## Lab 1: Extract and validate an incident

Create `day2_structured.py`:

```python
import requests
from typing import Literal
from pydantic import BaseModel, Field, ValidationError

MODEL = "qwen2.5vl:3b"  # Replace with any locally installed Ollama model.

class Incident(BaseModel):
    service: str
    severity: Literal["low", "medium", "high", "critical"]
    issue: str
    probable_cause: str
    recommended_action: str
    confidence: float = Field(ge=0.0, le=1.0)

log = """
2026-10-15 11:05:23 payment-api ERROR
Database connection pool exhausted: 100/100.
HTTP 503 responses increased to 42%.
Average latency: 8500ms.
"""

prompt = f"""
Extract an incident report from this log.
Return only a JSON object with these keys:
service, severity, issue, probable_cause,
recommended_action, confidence.

Severity must be low, medium, high, or critical.
Confidence must be a number between 0 and 1.
Treat probable causes as hypotheses; do not invent evidence.

Log:
{log}
"""

response = requests.post(
    "http://localhost:11434/api/generate",
    json={
        "model": MODEL,
        "prompt": prompt,
        "format": "json",
        "stream": False,
        "options": {"temperature": 0},
    },
    timeout=120,
)
response.raise_for_status()
raw = response.json()["response"]

try:
    incident = Incident.model_validate_json(raw)
    print(incident.model_dump_json(indent=2))
except ValidationError as exc:
    print("Schema validation failed:")
    print(exc)
```

Run:

```bash
python3.12 day2_structured.py
```

`"format": "json"` asks Ollama for JSON-formatted output. `Incident.model_validate_json()` independently checks the resulting data against our application schema.

## Observed result

I ran the follow-up extraction script twice. Both runs passed validation on the first attempt and returned the same incident object:
{: .journal-callout .journal-callout--success}

```json
{
  "service": "payment-api",
  "severity": "high",
  "issue": "Database connection pool exhausted",
  "probable_cause": "High load on the database",
  "recommended_action": "Increase the database connection pool size",
  "confidence": 0.9
}
```

This was a **successful structural validation**, not proof that the diagnosis was correct. The log demonstrates pool exhaustion, but it does not establish high database load as the cause. Likewise, increasing pool size may not help if the actual issue is a connection leak, long-running queries, or excessive application concurrency. The `0.9` is a model-produced number, not a calibrated probability.

## Lab 2: Bounded retries on validation errors

A production-oriented pipeline should have a defined failure path. Here is the core retry pattern:

```python
from pydantic import ValidationError


def extract_with_retry(log: str, max_attempts: int = 3) -> Incident:
    prompt = f"""
    Extract an incident from the log as JSON with fields:
    service, severity, issue, probable_cause,
    recommended_action, confidence.
    Severity: low, medium, high, or critical.
    Confidence: 0 to 1.
    Treat causes as hypotheses.
    Log:\n{log}
    """

    for attempt in range(1, max_attempts + 1):
        response = requests.post(
            "http://localhost:11434/api/generate",
            json={
                "model": MODEL,
                "prompt": prompt,
                "format": "json",
                "stream": False,
                "options": {"temperature": 0},
            },
            timeout=120,
        )
        response.raise_for_status()
        raw = response.json()["response"]

        try:
            return Incident.model_validate_json(raw)
        except ValidationError as exc:
            if attempt == max_attempts:
                raise ValueError("Incident extraction failed validation") from exc
            prompt = f"""
            Extract an incident from the original log as valid JSON.
            Original log:\n{log}
            Previous JSON:\n{raw}
            Validation errors:\n{exc}
            Return only a corrected JSON object using the required fields.
            """

    raise RuntimeError("Unreachable retry state")
```

The loop has a maximum number of attempts, checks HTTP errors, and feeds validation feedback into the next prompt. It **does not** retry HTTP timeouts or connection failures; those require separate handling and a latency budget.

To verify the error-handling path deterministically, inject an invalid first response in a local test:

```python
bad_output = '''{
  "service": "payment-api",
  "severity": "urgent",
  "issue": "Database pool exhausted",
  "confidence": 1.5
}'''

try:
    Incident.model_validate_json(bad_output)
except ValidationError as exc:
    print(exc)
```

This must fail because `urgent` is not an allowed severity, confidence exceeds 1, and required fields are absent. The actual model-assisted recovery path was **not confirmed by my recorded runs**, so I won't present it as a measured success.

## Five engineering lessons

1. **Constrain the format:** Ollama JSON mode helps with syntactic consistency.
2. **Validate independently:** Pydantic catches missing fields, incorrect types, and invalid ranges.
3. **Bound retries:** Never allow an LLM correction loop to run indefinitely.
4. **Separate facts from hypotheses:** A valid object can still contain an unsupported diagnosis.
5. **Test failure paths deliberately:** A model that succeeds on the first try doesn't prove the retry logic works.

## What I'd improve for a production incident analyzer

- Pass a JSON Schema to Ollama's `format` parameter, rather than relying on JSON mode and prompt wording alone.
- Use explicit `unknown` or nullable fields when evidence is missing.
- Ground severity in measurable operational thresholds, not only the model's judgment.
- Add request deadlines, structured logging, and metrics for validation failures and retries.
- Compare extracted claims with the original log before recommending remediation.
- Restrict any eventual automation to approved tools and require confirmation for changes to live systems.

## What's next: Day 3 — Embeddings and semantic search

Next, I'll move from structured extraction to **embeddings**: representing text as vectors, measuring semantic similarity, and building a local search tool for incident logs and troubleshooting notes. The goal is to retrieve relevant evidence before asking an LLM to reason about it.

*This article is part of my hands-on AI Engineering learning journal. The examples are educational and should not be treated as production-ready incident remediation.*


[← Day 1: Local LLM Inference]({{ "/ai-engineering/day-1-local-llm-inference-ollama/" | relative_url }}) · [All chapters]({{ "/ai-engineering/" | relative_url }})
