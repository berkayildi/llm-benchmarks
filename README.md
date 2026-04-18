# llm-benchmarks

Benchmark datasets and results for [LLMShot](https://llmshot.vercel.app). Evaluating LLMs across providers on quality, latency, and cost.

## Domains

### Real-Time Inference

5 models, 3 providers, 30 questions across 3 task types. Which model works for latency-critical, streaming use cases?

| Model                 | Faithfulness | Relevance | TTFT   | Cost/Query |
| --------------------- | ------------ | --------- | ------ | ---------- |
| Claude Sonnet 4.6     | 0.88         | 0.92      | 1491ms | $0.0114    |
| Claude Haiku 4.5      | 0.82         | 0.88      | 488ms  | $0.0028    |
| GPT-4o-mini           | 0.62         | 0.90      | 245ms  | $0.0003    |
| Gemini 2.5 Flash      | 0.82         | 0.87      | 661ms  | $0.0005    |
| Gemini 2.5 Flash-Lite | 0.80         | 0.88      | 560ms  | $0.0002    |

Total cost: $0.26 for 150 runs.

### Text Generation

Structured text output quality across eval gate and content analysis tasks.

**Eval Gates** — 3 models, 3 questions (factual, reasoning, summarization):

| Model                 | Faithfulness | Relevance | TTFT   | Cost/Query |
| --------------------- | ------------ | --------- | ------ | ---------- |
| Claude Haiku 4.5      | 0.67         | 0.67      | 748ms  | $0.0009    |
| GPT-4o-mini           | 0.83         | 0.83      | 1397ms | $0.0001    |
| Gemini 2.5 Flash-Lite | 1.00         | 1.00      | 539ms  | $0.0000    |

**Content Pipeline** — 2 models, 10 questions (video analysis, X feed digest):

| Model                 | Faithfulness | Relevance | TTFT  | Cost/Query |
| --------------------- | ------------ | --------- | ----- | ---------- |
| Claude Sonnet 4.6     | 1.00         | 1.00      | 837ms | $0.0092    |
| Gemini 2.5 Flash-Lite | 0.95         | 0.95      | 560ms | $0.0002    |

## Structure

```
├── realtime/
│   ├── summary.json       # Aggregate stats per model
│   └── benchmark.json     # Per-question per-model results
└── text-generation/
    ├── eval-gates-summary.json
    ├── eval-gates-benchmark.json
    ├── content-pipeline-summary.json
    └── content-pipeline-benchmark.json
```

## How it works

Each benchmark runs questions through LLMs via streaming APIs, captures TTFT and token usage, then scores responses with an LLM-as-judge (GPT-4o-mini) on faithfulness (0-1) and relevance (0-1).

Results are consumed by [LLMShot](https://llmshot.vercel.app) for interactive visualization.

## Evaluation tools

- [mcp-llm-eval](https://github.com/berkayildi/mcp-llm-eval) — MCP server + CLI for running evaluations and enforcing quality gates in CI/CD
- [LLMShot](https://github.com/berkayildi/llmshot) — Benchmark visualization dashboard

## Key findings

- No single model wins across all dimensions
- Gemini 2.5 Flash-Lite consistently scores highest on text generation at the lowest cost
- GPT-4o-mini has a dangerous profile: high relevance but low faithfulness in real-time contexts
- Claude Sonnet leads on quality but TTFT makes it unsuitable for latency-critical use cases
- Cost varies 45x between providers for comparable quality

## License

MIT © Berkay Yildirim
