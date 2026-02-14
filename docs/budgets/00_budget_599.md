# Architecture Option D: `$599` Machine Budget (Education M4 Base + 10GbE Add-On)

## Goal

Create the lowest-cost institutional entry point for local knowledge-base generation while preserving the same hybrid architecture and team SSH workflow.

## Budget Assumption

This option is modeled as a `$599` budget class as requested.
If the 10GbE configuration raises actual procurement price above `$599`, keep this document as the lower-bound comparison profile.

## Target Infrastructure Profile

- 1 shared research node
- 10Gb Ethernet (configured)
- Base-class CPU profile
- 16-24 GB unified memory class
- 512 GB to 1 TB SSD
- SSH access for small team (2-4 active users)

Service layer is OS-agnostic because inference is standardized through `llama-server`.

## Logical Architecture

- `gateway` for route policy and budget controls
- `llama-server` local endpoint
- `redis` for cache and queue state
- `qdrant` for sector retrieval memory
- `kb-writer` to canonical `SQLite + Parquet`

## Deployment Pattern

- Single-node compose stack
- Local-first for routine tasks when available
- Cloud for hard tasks and overflow during demand spikes

## Memory-Constrained Operating Point

Assumptions for Track A generation profile:

- Installed memory envelope: $M_{avail} = 16 \text{ to } 24$ GB
- Safety factor: $\eta = 0.85$
- Usable memory envelope: $\eta M_{avail} = 13.6 \text{ to } 20.4$ GB
- Planned runtime working set (small quantized model, strict context/concurrency caps): $M_{req} \approx 13 \text{ to } 18$ GB

Throughput decomposition:

- $\mu_{compute} \approx 78$ tokens/sec
- $\mu_{memory} \approx 63$ tokens/sec
- $f_{fit} = 0.95$ (memory pressure at low-memory edge)

```math
\mu_{eff} = f_{fit}\cdot\min(\mu_{compute},\mu_{memory}) \approx 0.95\cdot 63 = 59.85 \approx 60
```

## Capacity Assumption

- Effective local capacity target: $\mu_{eff} = 60$ tokens/sec
- Per-source token load: $T_{in}+T_{out}=725{,}000$

Theoretical local-only source throughput:

```math
\text{sources/day} = \frac{\mu_{eff} \cdot 86400}{725000} \approx 7.15
```

## Cloud Cost Model (Instantiated)

Using the shared model in `00_cloud_cost_model.md`:

- Hard-task cloud fraction: $r_h = 0.25$
- Baseline peak demand: $\lambda_{peak}=44.75$ tokens/sec
- Stress peak demand: $\lambda_{peak}=80.56$ tokens/sec

Overflow fractions:

```math
r_{over,base}=\max\left(0,\frac{44.75-60}{44.75}\right)=0
```

```math
r_{over,stress}=\max\left(0,\frac{80.56-60}{80.56}\right)=0.2552
```

Monthly cloud cost:

```math
C_{month}=S\left(r_hc_h+(1-r_h)r_{over}c_e\right)
```

For web-aware enrichment workflows, use the extended cost equation in `00_cloud_cost_model.md`:
$C_{month}^{web}$ with $h_{db}$, $p_{escalate}$, and $c_{fill}$.

with $c_e=0.33925$, $c_h=1.69625$.

Baseline (`S=40`):

```math
C_{base}=40\left(0.25\cdot1.69625+0.75\cdot0\cdot0.33925\right)=\$16.96/month
```

Stress (`S=72`):

```math
C_{stress}=72\left(0.25\cdot1.69625+0.75\cdot0.2552\cdot0.33925\right)=\$35.21/month
```

Approximate annual cloud range from this model: `$204 - $422`.

## Test Deployment Snapshot Entries

Use one row per pilot snapshot window (recommended: 24h to 168h windows).

| Snapshot ID | Git Commit | Start (UTC) | End (UTC) | Model Profile | Context Tokens | Concurrency | Sources Completed | Tokens In | Tokens Out | Peak Arrival `lambda_peak_obs` (tok/s) | `mu_compute_obs` (tok/s) | `mu_memory_obs` (tok/s) | `f_fit_obs` | `mu_eff_obs` (tok/s) | `rho_obs` | `r_over_obs` | Peak Memory (GB) | `h_db_obs` | `local_time_budget_sec` | `p_escalate_obs` | `c_fill_obs` (USD/call) | `cloud_fill_calls_obs` | `avg_missing_fields_obs` | `local_cli_calls_obs` | Cloud Spend (USD) | `q_accept_obs` | Notes |
| ----------- | ---------- | ----------- | --------- | ------------- | -------------- | ----------- | ----------------- | --------- | ---------- | -------------------------------------- | ------------------------ | ----------------------- | ----------- | -------------------- | --------- | ------------ | ---------------- | ---------- | ----------------------- | ---------------- | ----------------------- | ---------------------- | ------------------------ | --------------------- | ----------------- | -------------- | ----- |
| snap-001    |            |             |           |               |                |             |                   |           |            |                                        |                          |                         |             |                      |           |              |                  |            |                         |                  |                         |                        |                          |                       |                   |                |       |

Derived fields for this architecture:

```math
\mu_{eff,obs}=f_{fit,obs}\cdot\min(\mu_{compute,obs},\mu_{memory,obs})
```

```math
\rho_{obs}=\frac{\lambda_{peak,obs}}{\mu_{eff,obs}}, \quad r_{over,obs}=\max\left(0,\frac{\lambda_{peak,obs}-\mu_{eff,obs}}{\lambda_{peak,obs}}\right)
```

```math
N_{cloud\_calls,obs}=S_{obs}\left(0.25+0.75\left(r_{over,obs}+(1-r_{over,obs})(1-h_{db,obs})p_{escalate,obs}\right)\right)
```

```math
C_{month,obs}^{web}=S_{obs}\left(0.25\cdot c_h + 0.75\left(r_{over,obs}\cdot c_e + (1-r_{over,obs})(1-h_{db,obs})p_{escalate,obs}c_{fill,obs}\right)\right)
```

Model versus observed quick check:

| Metric                          |   Model | Observed |
| ------------------------------- | ------: | -------: |
| `mu_eff` (tok/s)                |    `60` |          |
| `rho` at baseline               |  `0.75` |          |
| Cloud cost/month baseline (USD) | `16.96` |          |

## Fit for Project

### Pros

- Lowest capital entry point.
- Same software architecture as larger options.
- Useful for initial pilot and low-concurrency operation.

### Constraints

- Memory envelope is narrow, so context length and concurrency must stay tightly controlled.
- Highest expected queue pressure and stress-window overflow.
- Most sensitive to workload spikes and retry overhead.

## Recommended If

Choose this if the priority is immediate start with minimal capex and you accept heavier cloud dependence plus strict local memory limits.
