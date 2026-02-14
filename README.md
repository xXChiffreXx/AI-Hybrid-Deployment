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
- Shared model: [`docs/architectures/00_cloud_cost_model.md`](docs/architectures/00_cloud_cost_model.md)
- Cost/performance analysis: [`docs/architectures/10_cost_performance_writeup.md`](docs/architectures/10_cost_performance_writeup.md)

## Quick Comparison (Industry-Style)

| Budget | Effective Throughput ($\mu_{eff}$) | Usable Memory Envelope ($\eta M_{avail}$) | Stability (`\rho base / stress`) | Est. Cloud Cost / Month (Base) | Est. Cloud Cost / Month (Stress) |
|---|---:|---:|---:|---:|---:|
| `$599` | `~60 tok/s` | `13.6-20.4 GB` | `0.75 / 1.34` | `$16.96` | `$35.21` |
| `$2,500` | `~90 tok/s` | `54.4-81.6 GB` | `0.50 / 0.90` | `$13.57` | `$24.43` |
| `$5,000` | `~180 tok/s` | `108.8 GB` | `0.25 / 0.45` | `$8.14` | `$14.66` |
| `$7,500` | `~320 tok/s` | `163.2-217.6 GB` | `0.14 / 0.25` | `$5.43` | `$9.77` |

Notes:

- Costs are model-driven estimates from workload assumptions in [`docs/architectures/00_cloud_cost_model.md`](docs/architectures/00_cloud_cost_model.md).
- Current modeled workload baseline is `40` sources/month (stress scenario `72`/month).
- Throughput is memory-constrained effective throughput (`\mu_{eff}`), not theoretical peak.
- Costs scale approximately linearly with token volume.
- Snapshot data-entry templates are included in each architecture doc and in the shared schema section of [`docs/architectures/00_cloud_cost_model.md`](docs/architectures/00_cloud_cost_model.md).

## Knowledge Base Spec

- [`KNOWLEDGE_BASE_ITEMS.md`](KNOWLEDGE_BASE_ITEMS.md)
