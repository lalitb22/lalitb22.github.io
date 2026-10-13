---
title: "AI Engineering: A Practical Learning Journal"
permalink: /ai-engineering/
layout: single
excerpt: "10 days, 30 hours: practical experiments from local LLM inference to production reliability."
toc: true
toc_label: "Learning roadmap"
toc_sticky: true
sidebar:
  nav: ai-engineering
---

**10 days · 30 hours · engineering through experiments**

This journal follows a practical AI Engineering learning journey for engineers who want to build, measure, and improve real applications. Each day sets aside roughly three hours for hands-on work with local LLMs, Python, APIs, structured extraction, evaluation, and production reliability.

The goal is to understand the behavior behind an API response: what the model produced, how long it took, where it failed, and how to make the surrounding system dependable. Chapters will be added as the work progresses. Existing articles remain available in the [article archive]({{ '/' | relative_url }}).

## The 10-day roadmap

Day 1 documents completed experiments. Days 2–10 are the planned curriculum; their tutorials will be linked here when published.

| Day | Focus | Practical deliverable | Status |
| --- | --- | --- | --- |
| [1]({{ '/ai-engineering/day-1-local-llm-inference-ollama/' | relative_url }}) | Local LLM inference with Ollama | Python client; token, latency, output-length, and temperature experiments | Published |
| 2 | Python inference workflows | Reusable client, configuration, and saved experiment results | Planned |
| 3 | Building an API | A small inference service with validated requests and responses | Planned |
| 4 | Structured extraction | Invoice fields returned as JSON against a schema | Planned |
| 5 | Validation and failure handling | Schema checks, missing-field handling, and bounded retries | Planned |
| 6 | Retrieval and context | A small grounded question-answering workflow | Planned |
| 7 | Evaluation | A fixed test set and measurable quality criteria | Planned |
| 8 | Performance and cost | Compare latency, throughput, and resource tradeoffs | Planned |
| 9 | Production reliability | Timeouts, observability, concurrency limits, and fallbacks | Planned |
| 10 | End-to-end project | An evaluated extraction API with an operating checklist | Planned |

## How to use this journal

Spend the first hour understanding and reproducing the baseline, the second changing one variable at a time, and the third recording results and failure cases. Keep prompts, settings, model versions, and measurements together so comparisons remain useful.

These are learning notes from specific experiments, not universal benchmarks. Hardware, model versions, input length, caching, and runtime settings can all change the results.

## Start with Day 1

[Local LLM Inference with Ollama]({{ '/ai-engineering/day-1-local-llm-inference-ollama/' | relative_url }}) walks through a non-streaming HTTP request, a Python inference client, and the measured behavior of `qwen2.5vl:3b`.

[Read Day 1 →]({{ '/ai-engineering/day-1-local-llm-inference-ollama/' | relative_url }}){: .btn .btn--primary}
