---
title: "Day 1: Local LLM Inference with Ollama"
permalink: /ai-engineering/day-1-local-llm-inference-ollama/
layout: single
excerpt: "Measure local inference with qwen2.5vl:3b: loading, token counts, throughput, output limits, and temperature."
toc: true
toc_label: "Day 1 guide"
toc_sticky: true
sidebar:
  nav: ai-engineering
---

[← AI Engineering roadmap]({{ '/ai-engineering/' | relative_url }})

**Day 1 of 10 · approximately 3 hours · completed experiments**

Today’s objective is to call a local model from Python and explain its behavior with measurements. The model used in the completed experiments was **`qwen2.5vl:3b`**, served by Ollama. These exercises use text input; no image is required.

By the end, you should be able to distinguish model loading from inference, count input and output tokens, calculate generation throughput, and compare output-length limits and temperature settings.

## 1. Prepare the local environment

Install [Ollama](https://ollama.com/download) and Python 3. Start the Ollama application, or run `ollama serve` in a separate terminal if the server is not already running. Then download the model and create an isolated Python environment:

```bash
ollama pull qwen2.5vl:3b
ollama list
ollama --version

python3 -m venv .venv
source .venv/bin/activate
python -m pip install requests
```

On Windows, activate the environment with `.venv\Scripts\Activate.ps1` in PowerShell. The model download requires internet access; subsequent inference runs locally after the download completes. Record your OS, CPU/GPU, memory, Ollama version, and model ID from `ollama list` for future comparisons.

## 2. Make a non-streaming HTTP request

The local server exposes `POST /api/generate` on port `11434`. Set `stream` to `false` to receive one JSON object after generation finishes:

```bash
curl http://localhost:11434/api/generate \
  -H 'Content-Type: application/json' \
  -d '{
    "model": "qwen2.5vl:3b",
    "prompt": "Explain local LLM inference in two sentences.",
    "stream": false,
    "keep_alive": "5m",
    "options": {"num_predict": 100, "temperature": 0}
  }'
```

The generated text is in `response`. Non-streaming makes it convenient to inspect final metrics, but the client waits for the whole response. This request does **not** measure time to first token. The [Ollama generate API reference](https://docs.ollama.com/api/generate) documents the response fields.

## 3. Build a Python inference client

Save this as `inference.py`. It reports both client wall-clock time and the server’s timings. The examples below reuse this function.

```python
import time
import requests

URL = "http://localhost:11434/api/generate"
MODEL = "qwen2.5vl:3b"


def infer(prompt, num_predict=100, temperature=0, seed=None):
    options = {"num_predict": num_predict, "temperature": temperature}
    if seed is not None:
        options["seed"] = seed

    started = time.perf_counter()
    response = requests.post(
        URL,
        json={
            "model": MODEL,
            "prompt": prompt,
            "stream": False,
            "keep_alive": "5m",
            "options": options,
        },
        timeout=(5, 180),  # connection timeout, read timeout in seconds
    )
    response.raise_for_status()
    data = response.json()
    if data.get("error"):
        raise RuntimeError(data["error"])
    if not data.get("done"):
        raise RuntimeError("Expected a completed non-streaming response")

    wall_s = time.perf_counter() - started
    generation_s = data["eval_duration"] / 1_000_000_000
    metrics = {
        "input_tokens": data["prompt_eval_count"],
        "output_tokens": data["eval_count"],
        "wall_s": wall_s,
        "total_s": data["total_duration"] / 1_000_000_000,
        "load_s": data["load_duration"] / 1_000_000_000,
        "prompt_s": data["prompt_eval_duration"] / 1_000_000_000,
        "generation_s": generation_s,
        "tokens_per_s": (
            data["eval_count"] / generation_s if generation_s > 0 else None
        ),
        "done_reason": data.get("done_reason", "not reported"),
    }
    return data["response"], metrics


if __name__ == "__main__":
    text, metrics = infer("Explain local LLM inference in two sentences.")
    print(text)
    for key, value in metrics.items():
        print(f"{key}: {value}")
```

Run `python inference.py`. A connection error usually means the server is not running; a model-not-found error means the model tag has not been downloaded. HTTP failures raise an exception instead of being mistaken for model output. A read timeout limits waiting between response bytes; it is not a hard deadline for the entire operation.

## 4. Understand the timing and token metrics

Tokens are model-specific text units, not words or characters. Use the returned counts rather than splitting text on spaces. Prompt templates can contribute to the input count.

| Metric | What it tells us |
| --- | --- |
| `prompt_eval_count` | Input token count reported by Ollama |
| `eval_count` | Generated output token count |
| `load_duration` | Time spent loading the model |
| `prompt_eval_duration` | Time spent processing the prompt; cached work can reduce this |
| `eval_duration` | Time spent generating output tokens |
| `total_duration` | Server-reported overall request duration |
| Client `wall_s` | Elapsed HTTP call, including transport and client processing |

Ollama reports durations in **nanoseconds**. Divide by `1,000,000,000` to obtain seconds. Generation throughput is:

```text
output tokens/sec = eval_count / (eval_duration / 1,000,000,000)
```

That measures the output-generation phase. Dividing output tokens by wall-clock time instead gives an end-to-end rate that also includes loading, prompt processing, and other overhead. These two rates answer different questions.

### Cold start versus warm inference

A **cold start** occurs when the model is not resident in memory and must be loaded before inference. A **warm request** can reuse a loaded model. Warm requests still process input and generate output; they are not free or instantaneous. Repeated prompts can also benefit from prompt caching, which is separate from model residency.

To compare them, explicitly unload the model, then run the same prompt twice:

```bash
ollama stop qwen2.5vl:3b
```

```python
# Run in a separate script alongside inference.py, with Ollama still serving.
from inference import infer

prompt = "Explain why API timeouts matter in three sentences."
for label in ("cold", "warm"):
    text, metrics = infer(prompt, num_predict=100, temperature=0)
    print(label, metrics)
```

Compare `load_s`, `prompt_s`, `generation_s`, and `wall_s` separately. The completed work explored loading latency and warm inference, but no numeric cold/warm measurements were supplied for this journal. Record your own values rather than assigning the output-length results below to either condition. Operating-system file caches can make even an unloaded-model run faster than a fully cold machine.

## 5. Experiment with output length

`num_predict` sets a maximum generation budget; it does not require the model to produce that many tokens. Use the same prompt and settings while changing only the cap:

```python
from inference import infer

# Illustrative reproduction prompt; the original experiment prompt was not recorded here.
prompt = "Explain how a local LLM processes a prompt and generates a response."
infer(prompt, num_predict=10)  # warm-up; exclude this from the comparison
for cap in (30, 100, 200):
    text, metrics = infer(prompt, num_predict=cap, temperature=0)
    print("max_tokens:", cap, metrics)
    print(text)
```

### Recorded results

These are the supplied measurements from the completed output-length experiment:

| Max tokens | Actual tokens | Latency | Tokens/sec |
| ---------- | ------------- | ------- | ---------- |
| 30 | 30 | 0.54 s | 65.12 |
| 100 | 100 | 1.57 s | 64.48 |
| 200 | 155 | 2.43 s | 64.33 |

The recorded latency column was not identified as client wall time or server total time. Preserve that distinction when comparing new runs. The tokens/sec figures are treated as generation-phase throughput; they do not exactly equal actual tokens divided by the listed latency. Raw timing responses, hardware details, and repeat-run distributions were not supplied, so these are observations from this session rather than a general performance benchmark.

The observed throughput remains near 64–65 tokens/sec while latency increases with output length. A cap of 200 produced only 155 tokens: a larger budget allows more output but does not force it. Natural completion or a configured stop sequence can end generation early; inspect `done_reason` to establish the cause.

### Recognize truncation

The 30- and 100-token runs reached their caps. That is a reason to inspect the response, not proof by itself that it was cut off. Check the stop reason (for example, `length` when reported by the runtime), incomplete sentences, and broken JSON. A response can also finish naturally near a cap. Increase the budget and compare completeness before deciding on a production limit.

An HTTP response with `done: true` means generation finished; it does not guarantee that the answer is complete or useful. In later extraction work, schema validation will provide another check.

## 6. Experiment with temperature and repeatability

Temperature adjusts the sampling distribution. Lower settings favor high-probability continuations; higher settings allow a wider range of choices. More variety does not imply greater correctness.

For each temperature, make three independent requests with the same prompt, model, and token budget:

```python
from inference import infer

# Illustrative prompt for repeating the naming experiment.
prompt = "Suggest one short brand name for an AI invoice processing app. Return only the name."
for temperature in (0, 0.7, 1.2):
    names = []
    for _ in range(3):
        text, metrics = infer(prompt, num_predict=30, temperature=temperature)
        names.append(text.strip())
    print(temperature, names)
```

### Recorded outputs

| Temperature | Run 1 | Run 2 | Run 3 |
| --- | --- | --- | --- |
| 0 | InvoiceSage | InvoiceSage | InvoiceSage |
| 0.7 | FinanceSage | Invocee | InvoiceProAI |
| 1.2 | InvoiceAI | InvoicePulse | InvoicePilot |

Temperature 0 repeated the same name in these three trials. **It does not guarantee determinism.** Runtime changes, hardware kernels, numerical differences, model revisions, or execution conditions can still affect results. A fixed seed can help control sampling for comparisons, but it is not a universal guarantee either. The original seed settings were not supplied; the example leaves the seed unset to explore variation.

Three naming trials illustrate diversity, not a statistical estimate of repeatability. For a stronger comparison, keep the model ID and runtime fixed, save exact prompts and options, run more trials, and evaluate task quality as well as variation. See the [Ollama parameter reference](https://docs.ollama.com/modelfile#valid-parameters-and-values) for temperature, seed, and output-budget settings.

## 7. Record what the experiment actually proves

For each run, save the prompt, model ID, options, generated text, stop reason, token counts, and timing fields. Separate the first request after unloading from warm requests. Repeat runs and report a median and range instead of relying on one timing.

The completed experiments show that output budgets influence latency and can constrain completeness, while temperature influences observed variation. They do not establish extraction accuracy, a universal hardware throughput, or deterministic behavior.

## Day 1 checklist and next chapter

- Call `qwen2.5vl:3b` using the HTTP API with `stream: false`.
- Run the Python `requests` client and inspect input/output token counts.
- Separate loading, prompt evaluation, generation, and wall-clock time.
- Compare 30-, 100-, and 200-token budgets and inspect stop reasons.
- Repeat a naming prompt at temperatures 0, 0.7, and 1.2.
- Save observations with enough context to reproduce them.

**Next: Day 2 — Structured Extraction and Pydantic (upcoming).** Explore typed model output and schema validation. This chapter will be linked from the roadmap when published.

[← Back to the roadmap and chapter index]({{ '/ai-engineering/' | relative_url }}){: .btn .btn--primary}
