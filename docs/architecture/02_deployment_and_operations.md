# Deployment and Operations Architecture

## Deployment Baseline

All current budget options describe a single-node deployment pattern with local-first inference and cloud spill.

```mermaid
flowchart TB
  subgraph H[Single Research Node]
    G[Gateway]
    L[llama-server]
    R[Redis]
    Q[Qdrant]
    W[kb-writer]
    S[SQLite]
    P[Parquet]
    B[backup job]

    G --> L
    G --> R
    G --> Q
    G --> W
    W --> S
    W --> P
    B --> S
    B --> P
  end

  V[Consumer Tools (Read-Only)] --> S
  V --> P
  U[Research Clients over SSH or API] --> G
  G --> C[Cloud LLM]
```

## Runtime Control Plane

Control decisions should be centralized in `gateway`:

- route class selection (`local-default`, `local-long`, `cloud-hard`)
- queue admission and concurrency caps
- local timeout budget and escalation triggers
- cloud budget ceilings and fallback limits
- cloud reentry enforcement (`cloud -> gateway -> kb-writer -> canonical store`)

## Operational Metrics

The following metrics are required for architecture validation:

- `mu_compute_obs`, `mu_memory_obs`, `mu_eff_obs`
- `rho_obs`, `r_over_obs`
- `h_db_obs`, `p_escalate_obs`
- `cloud_fill_calls_obs`, `local_cli_calls_obs`
- `cloud_spend_usd`, `q_accept_obs`

These map directly to the snapshot schemas in `docs/budgets/00_cloud_cost_model.md`.

## Failure Modes and Controls

| Failure Mode | Control |
| --- | --- |
| local inference saturation | queue cap, overflow-to-cloud policy, route throttling |
| low memory headroom | strict context/concurrency limits and `f_fit` monitoring |
| repeated unresolved items | single cloud fill escalation with missing-field targeting |
| canonical store write failures | idempotent writer operations and retry with dead-letter capture |
| stale derived views | regenerate read-only views from canonical store snapshots |

## Backup and Recovery

- snapshot `SQLite` and `Parquet` on schedule.
- keep retention windows aligned with research cycle checkpoints.
- periodically test restore into a clean node.
- treat any downstream read-only view artifacts as regenerable.

## Implementation Gap Register

Current repository gaps relative to this architecture:

- no gateway service code
- no local/cloud invocation adapters
- no deployment manifests or compose files
- no operational scripts for snapshot, restore, or metric export

This means the repository now has architecture documentation and budget analysis, but not a runnable stack yet.
