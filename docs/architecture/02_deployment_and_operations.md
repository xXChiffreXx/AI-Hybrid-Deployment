# Deployment and Operations Architecture (MVP)

## Deployment Baseline

MVP is a single-node deployment that resolves one resource job through local-first enrichment and cloud fallback.

```mermaid
flowchart TB
  subgraph H["Single Research Node"]
    G["Resolver Gateway"]
    O["Ollama Server"]
    W["kb-writer"]
    S["SQLite"]
    P["Parquet"]
    B["Backup Job"]

    G --> O
    G --> W
    W --> S
    W --> P
    B --> S
    B --> P
  end

  U["Research Client"] --> G
  G --> D["Online Drug Data Source"]
  G --> C["Cloud LLM via CLI"]
  V["Consumer Tools (Read-Only)"] --> S
  V --> P
```

## Runtime Control Plane

Control decisions are centralized in `resolver-gateway`:

- start and track per-resource job lifecycle
- run initial local extraction pass through Ollama
- enumerate missing required fields after initial SQL upsert
- enforce `tau_local` per local fill attempt
- evaluate confidence threshold before accepting local output
- invoke cloud CLI fallback when local attempts fail threshold
- enforce cloud reentry (`cloud -> resolver-gateway -> kb-writer -> canonical store`)
- terminate job on schema completion

## Operational Metrics

The following metrics are required for MVP control and validation:

- `resources_started`, `resources_completed`, `resources_failed`
- `fields_required_total`, `fields_resolved_local`, `fields_resolved_cloud`
- `fields_unresolved`
- `local_attempt_count`, `local_timeout_count`
- `cloud_fill_call_count`, `cloud_fill_cost_usd`
- `completion_latency_sec` (per resource)
- `write_retry_count`, `write_failure_count`

## Failure Modes and Controls

| Failure Mode | Control |
| --- | --- |
| local extraction misses many fields | run deterministic missing-field scan immediately after initial upsert |
| local fill attempts exceed time budget | enforce `tau_local` and trigger cloud CLI fallback |
| low-confidence local fills | reject local fill and escalate to cloud CLI |
| cloud output bypasses write policy | block direct DB writes; require resolver-gateway reentry path |
| canonical write failures | idempotent upserts, retries, and failed-write capture for replay |
| never-ending resource jobs | enforce termination rule based on required-field completion check |

## Termination Policy

A resource job terminates when:

- all required schema fields are complete in SQL, and
- provenance is present for each resolved field, and
- the final write transaction succeeds.

If unresolved fields remain after configured retry limits, terminate as failed and persist unresolved-field diagnostics.

## Backup and Recovery

- snapshot `SQLite` and `Parquet` on schedule.
- keep retention windows aligned with research cycle checkpoints.
- periodically test restore into a clean node.
- treat downstream read-only view artifacts as regenerable.

## Implementation Gap Register

Current repository gaps relative to this architecture:

- no resolver-gateway service code
- no Ollama/cloud CLI adapters
- no deployment manifests or compose files
- no operational scripts for snapshot, restore, or metric export

This repository currently contains architecture documentation and budget analysis, but not a runnable MVP stack yet.
