# MVP Backlog (Summer 2026 Build Order)

## Purpose

Turn the current design repository into a runnable MVP in the smallest order that still preserves provenance, evaluation, and cloud-use control.

## Critical Path

1. build canonical storage and the write path
2. build ingest plus local extraction
3. build unresolved-field scanning and batch planning
4. add cloud fallback
5. add telemetry and reviewed evaluation

## Prioritized Backlog

| Priority | Work Item | Why It Comes Now | Done When |
| --- | --- | --- | --- |
| `P0` | Canonical store layout for `items`, `claims`, `evidence_links`, `runs`, and `metrics` | all later routing depends on stable write targets | SQLite and Parquet write targets exist with seed fixture data |
| `P0` | `kb-writer` module with idempotent per-resource upserts | provenance and normalization must be centralized before inference is added | repeated writes do not duplicate or corrupt records |
| `P0` | schema registry for required fields and completion checks | the resolver needs deterministic definitions of "complete" and "missing" | required-field scans return stable results for test fixtures |
| `P1` | `resolver-gateway` skeleton with `POST /mvp/ingest-source` | provides the orchestration entry point described in the architecture docs | a source plus schema can be accepted and persisted as a tracked job |
| `P1` | Ollama invocation adapter for initial extraction | local-first behavior starts here | the resolver can run a first-pass extraction and capture local provenance |
| `P1` | unresolved-field scanner over SQL state | batching only makes sense after deterministic gap detection | unresolved fields can be queried and grouped from persisted state |
| `P2` | unresolved-entry batch planner and stable batch keys | this is the key optimization over older per-field routing patterns | batches are formed from shared source context and missing-field shape |
| `P2` | policy configuration for `tau_local`, `q_min`, retries, and overwrite rules | routing behavior must be explicit rather than hard-coded | policy settings are versioned and visible in logs or config |
| `P2` | cloud CLI adapter for one fill call per unresolved batch | this completes the hybrid route | escalated batches can re-enter through the same normalized write path |
| `P3` | metrics capture for throughput, memory, escalations, spend, and acceptance | the repo's budget and confidence docs depend on telemetry | daily snapshot rows can be generated without manual spreadsheet work |
| `P3` | backup, restore, and failed-write replay scripts | operational safety is part of the documented architecture | a clean-node restore test succeeds |
| `P4` | read-only generated views or exports for the in-scope Track A artifacts | professor-facing review is easier with generated summaries | the team can inspect generated notes without editing source-of-truth data |
| `P4` | evaluation tooling for review windows and Wilson interval reporting | quality gates must exist before scaling usage | reviewed windows can produce `CI95` acceptance and hallucination reports |

## Suggested Summer Milestones

- Milestone 1: canonical store, writer, and resolver skeleton
- Milestone 2: local extraction, completion scanning, and batch planning
- Milestone 3: cloud fallback plus telemetry
- Milestone 4: reviewed pilot corpus and routing recalibration

## Non-Goals for the First Build

- fully automated clinical decision support
- large-scale retrieval infrastructure beyond what the MVP needs
- advanced causal or simulation workflows before the basic artifact path is stable
