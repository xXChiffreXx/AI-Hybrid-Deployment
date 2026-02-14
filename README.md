# AI Hybrid Deployment

This repository defines software architecture and budget-model documentation for the Polypharmacy research program.

## Software Architecture

Implementation-oriented architecture docs are in `docs/architecture`:

- [`docs/architecture/README.md`](docs/architecture/README.md)
- [`docs/architecture/00_system_architecture.md`](docs/architecture/00_system_architecture.md)
- [`docs/architecture/01_runtime_routing_and_invocation.md`](docs/architecture/01_runtime_routing_and_invocation.md)
- [`docs/architecture/02_deployment_and_operations.md`](docs/architecture/02_deployment_and_operations.md)

## Budget and Cost Modeling

Budget-tier comparisons and shared cost equations are in `docs/budgets`:

- [`docs/budgets/00_budget_599.md`](docs/budgets/00_budget_599.md)
- [`docs/budgets/01_budget_2500.md`](docs/budgets/01_budget_2500.md)
- [`docs/budgets/02_budget_5000.md`](docs/budgets/02_budget_5000.md)
- [`docs/budgets/03_budget_7500.md`](docs/budgets/03_budget_7500.md)
- Shared model: [`docs/budgets/00_cloud_cost_model.md`](docs/budgets/00_cloud_cost_model.md)
- Cost/performance analysis: [`docs/budgets/10_cost_performance_writeup.md`](docs/budgets/10_cost_performance_writeup.md)

## Quick Comparison (Industry-Style)

| Budget   | Effective Throughput ($\mu_{eff}$) | Usable Memory Envelope ($\eta M_{avail}$) | Stability ($\rho_{base} / \rho_{stress}$) | Est. Cloud Cost / Month (Base) | Est. Cloud Cost / Month (Stress) |
| -------- | ---------------------------------: | ----------------------------------------: | ----------------------------------------: | -----------------------------: | -------------------------------: |
| `$599`   |                        `~60 tok/s` |                            `13.6-20.4 GB` |                             `0.75 / 1.34` |                       `$16.96` |                         `$35.21` |
| `$2,500` |                        `~90 tok/s` |                            `54.4-81.6 GB` |                             `0.50 / 0.90` |                       `$13.57` |                         `$24.43` |
| `$5,000` |                       `~180 tok/s` |                                `108.8 GB` |                             `0.25 / 0.45` |                        `$8.14` |                         `$14.66` |
| `$7,500` |                       `~320 tok/s` |                          `163.2-217.6 GB` |                             `0.14 / 0.25` |                        `$5.43` |                          `$9.77` |

Notes:

- Costs are model-driven estimates from workload assumptions in [`docs/budgets/00_cloud_cost_model.md`](docs/budgets/00_cloud_cost_model.md).
- Current modeled workload baseline is `40` sources/month (stress scenario `72`/month).
- Throughput is memory-constrained effective throughput (`\mu_{eff}`), not theoretical peak.
- Costs scale approximately linearly with token volume.
- Web-aware enrichment policy (DB-first, local time budget, single cloud fill escalation) is modeled in `docs/budgets/00_cloud_cost_model.md`.
- Snapshot data-entry templates are included in each budget doc and in the shared schema section of [`docs/budgets/00_cloud_cost_model.md`](docs/budgets/00_cloud_cost_model.md).

## Knowledge Base Spec

- [`KNOWLEDGE_BASE_ITEMS.md`](KNOWLEDGE_BASE_ITEMS.md)
