# llm-benchmarks

Benchmark datasets and results for [LLMShot](https://llmshot.vercel.app). Evaluating LLMs across providers on quality, latency, and cost.

## Domains

### Real-Time Inference

5 models, 3 providers, 30 questions across 3 task types. Which model works for latency-critical, streaming use cases?

| Model                 | Faithfulness | Relevance | TTFT   | Cost/Query |
| --------------------- | ------------ | --------- | ------ | ---------- |
| Claude Sonnet 4.6     | 0.88         | 0.90      | 1491ms | $0.0079    |
| Claude Haiku 4.5      | 0.75         | 0.80      | 826ms  | $0.0026    |
| GPT-4o-mini           | 0.62         | 0.90      | 897ms  | $0.0002    |
| Gemini 2.5 Flash      | 0.70         | 0.80      | 540ms  | $0.0003    |
| Gemini 2.5 Flash-Lite | 0.75         | 0.85      | 560ms  | $0.0001    |

Total cost: $0.26 for 150 runs.

### Text Generation

5 models, 2 sub-benchmarks, 95 total runs. Structured text output quality across eval gate and content analysis tasks.

**Eval Gates** — 5 models, 9 questions, 45 runs (factual, reasoning, summarization):

| Model                 | Faithfulness | Relevance | TTFT  | Cost/Query |
| --------------------- | ------------ | --------- | ----- | ---------- |
| Claude Sonnet 4.6     | 0.89         | 0.94      | 982ms | $0.0034    |
| Claude Haiku 4.5      | 0.89         | 0.94      | 662ms | $0.0011    |
| GPT-4o-mini           | 0.94         | 0.94      | 865ms | $0.0001    |
| Gemini 2.5 Flash      | 0.94         | 0.94      | 533ms | $0.0002    |
| Gemini 2.5 Flash-Lite | 0.83         | 0.94      | 542ms | $0.0001    |

**Content Pipeline** — 5 models, 10 questions, 50 runs (video analysis, X feed digest):

| Model                 | Faithfulness | Relevance | TTFT  | Cost/Query |
| --------------------- | ------------ | --------- | ----- | ---------- |
| Claude Sonnet 4.6     | 1.00         | 1.00      | 973ms | $0.0125    |
| Claude Haiku 4.5      | 0.95         | 1.00      | 568ms | $0.0030    |
| GPT-4o-mini           | 0.90         | 0.90      | 970ms | $0.0003    |
| Gemini 2.5 Flash      | 1.00         | 1.00      | 845ms | $0.0005    |
| Gemini 2.5 Flash-Lite | 0.95         | 1.00      | 574ms | $0.0002    |

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

## Methodology

- `max_output_tokens`: 2048 across all providers
- Temperature: provider defaults (plumbing `temperature=0` planned)
- Gemini 2.5 Flash: thinking disabled (`thinking_budget=0`) for benchmark parity
- Judge model: `gpt-4o-mini`
- Evaluation engine: [mcp-llm-eval](https://github.com/berkayildi/mcp-llm-eval) v0.4.0+

## How it works

Each benchmark runs questions through LLMs via streaming APIs, captures TTFT and token usage, then scores responses with an LLM-as-judge (GPT-4o-mini) on faithfulness (0-1) and relevance (0-1).

Results are consumed by [LLMShot](https://llmshot.vercel.app) for interactive visualization.

## Evaluation tools

- [mcp-llm-eval](https://github.com/berkayildi/mcp-llm-eval) — MCP server + CLI for running evaluations and enforcing quality gates in CI/CD
- [LLMShot](https://github.com/berkayildi/llmshot) — Benchmark visualization dashboard

## Key findings

- Gemini 2.5 Flash-Lite remains the cost/quality leader: cheapest on every benchmark, with faithfulness/relevance within ~0.05 of the top score.
- Gemini 2.5 Flash is now competitive on text generation post thinking-fix (0.94–1.00 on both sub-benchmarks).
- Claude Sonnet 4.6 leads on overall text-generation quality, but costs 3–20x Flash for comparable output.
- GPT-4o-mini is strong on short-form reasoning (Eval Gates: 0.94/0.94) but weaker on longer structured content (Content Pipeline: 0.90/0.90 — lowest of the five models).
- Caveat: the Real-Time Inference benchmark predates the Gemini thinking-fix. Flash numbers there reflect thinking-enabled output and will change on re-run.

## License

MIT © Berkay Yildirim
