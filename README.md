# AI Hybrid Deployment

This repository defines deployment architecture for the Polypharmacy research program.

The implementation is intentionally OS-agnostic at the service layer:
- `llama-server` provides the local OpenAI-compatible endpoint.
- A gateway applies routing, budgets, and fallback.
- Track A research artifacts are stored in a canonical structured store (`SQLite + Parquet`).
- Obsidian remains a projection/index layer, not source-of-truth.

## Architecture Options

- [`docs/architectures/00_budget_599.md`](docs/architectures/00_budget_599.md)
- [`docs/architectures/01_budget_2500.md`](docs/architectures/01_budget_2500.md)
- [`docs/architectures/02_budget_5000.md`](docs/architectures/02_budget_5000.md)
- [`docs/architectures/03_budget_7500.md`](docs/architectures/03_budget_7500.md)
- Shared math model: [`docs/architectures/00_cloud_cost_model.md`](docs/architectures/00_cloud_cost_model.md)
- Cost/performance writeup: [`docs/architectures/10_cost_performance_writeup.md`](docs/architectures/10_cost_performance_writeup.md)

## Quick Comparison

| Budget | Local Capacity Target (effective tokens/sec) | Cloud Strategy | Estimated Cloud Cost / Month (Baseline Scope) | Estimated Cloud Cost / Month (Stress Scope) |
|---|---:|---|---:|---:|
| `$599` | `~60` | Cloud for hard tasks + frequent overflow | `$74.57` | `$145.12` |
| `$2,500` | `~90` | Cloud for hard tasks + overflow | `$57.55` | `$121.07` |
| `$5,000` | `~180` | Cloud for hard tasks, overflow only during spikes | `$26.46` | `$69.46` |
| `$7,500` | `~320` | Cloud only for selective high-reasoning tasks | `$17.64` | `$31.75` |

Notes:
- Costs above are model-driven estimates from project workload assumptions in `docs/architectures/00_cloud_cost_model.md`.
- Costs scale linearly with tokens. If your observed tokens/source is 2x, cloud cost is ~2x.

## Knowledge Base Spec

- [`KNOWLEDGE_BASE_ITEMS.md`](KNOWLEDGE_BASE_ITEMS.md)
