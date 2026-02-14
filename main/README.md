# AI Hybrid Deployment

This repository defines software architecture and budget-model documentation for the Polypharmacy research program.

## Start Here (First-Time Reviewer)

If you are reviewing this after working with a prior workflow version, begin with:

- [`../QUICK_START.MD`](../QUICK_START.MD)

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

## Quick Comparison (Industry-Style, Architecture Baseline)

| Budget   | Effective Throughput (&mu;<sub>eff</sub>) | Usable Memory Envelope (&eta; M<sub>avail</sub>) | Stability (&rho;<sub>base</sub> / &rho;<sub>stress</sub>) | Est. Cloud Cost / Month (Base) | Est. Cloud Cost / Month (Stress) |
| -------- | ---------------------------------: | ----------------------------------------: | ----------------------------------------: | -----------------------------: | -------------------------------: |
| `$599`   |                        `~60 tok/s` |                            `13.6-20.4 GB` |                             `0.75 / 1.34` |                       `$16.96` |                         `$35.21` |
| `$2,500` |                        `~90 tok/s` |                            `54.4-81.6 GB` |                             `0.50 / 0.90` |                       `$13.57` |                         `$24.43` |
| `$5,000` |                       `~180 tok/s` |                                `108.8 GB` |                             `0.25 / 0.45` |                        `$8.14` |                         `$14.66` |
| `$7,500` |                       `~320 tok/s` |                          `163.2-217.6 GB` |                             `0.14 / 0.25` |                        `$5.43` |                          `$9.77` |

These baseline cloud-cost values assume no incremental fill-escalation term (`p_escalate = 0`).

## Web-Aware Conservative Overlay (Tier-Specific `p_escalate`)

Conservative web-aware assumptions:

- DB complete-hit rate: `h_db = 0.60`
- Fill-call cost: `c_fill = 0.25` USD/call
- Escalation priors by tier: `$599=0.60`, `$2,500=0.45`, `$5,000=0.35`, `$7,500=0.30`

| Budget   | Web-Aware Cloud Cost / Month (Base) | Web-Aware Cloud Cost / Month (Stress) | Cloud Calls / Month (Base) | Cloud Calls / Month (Stress) |
| -------- | ----------------------------------: | ------------------------------------: | -------------------------: | ---------------------------: |
| `$599`   |                            `$18.76` |                              `$37.62` |                    `17.20` |                      `41.43` |
| `$2,500` |                            `$15.01` |                              `$27.02` |                    `13.76` |                      `24.77` |
| `$5,000` |                             `$9.37` |                              `$16.87` |                     `9.73` |                      `17.51` |
| `$7,500` |                             `$6.53` |                              `$11.76` |                     `7.62` |                      `13.71` |

Notes:

- Costs are model-driven estimates from workload assumptions in [`docs/budgets/00_cloud_cost_model.md`](docs/budgets/00_cloud_cost_model.md).
- Current modeled workload baseline is `40` sources/month (stress scenario `72`/month).
- Throughput is memory-constrained effective throughput (&mu;<sub>eff</sub>), not theoretical peak.
- Costs scale approximately linearly with token volume.
- Web-aware enrichment policy (DB-first, batched local resolution, single cloud fill escalation per unresolved batch) is modeled in `docs/budgets/00_cloud_cost_model.md`.
- Snapshot data-entry templates are included in each budget doc and in the shared schema section of [`docs/budgets/00_cloud_cost_model.md`](docs/budgets/00_cloud_cost_model.md).

## Knowledge Base Spec

- [`KNOWLEDGE_BASE_ITEMS.md`](KNOWLEDGE_BASE_ITEMS.md)
