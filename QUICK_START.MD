# QUICK START

## Why this exists

This brief is for reviewers who worked with a prior version of the pharmacology/polypharmacy research workflow and had difficulties with implementing LLM data curation.

Primary entry document:

- [`main/README.md`](main/README.md)

## What this repository is

- A design and planning repository for AI-enabled research workflow architecture.
- A budget and capacity analysis for local-first plus cloud-fallback operation.
- A specification for what structured knowledge artifacts the pipeline is meant to produce.

## What this repository is not

- Not a runnable software stack yet.
- Not a deployment package.
- Not a defined database structure. Output produced by this system is subject to the structure imposed by the main project's needs.

## 5-minute orientation path

1. Read [`main/README.md`](main/README.md) for scope and budget comparison.
2. Read [`main/docs/architecture/README.md`](main/docs/architecture/README.md) for architecture document map and MVP scope note.
3. Read [`main/docs/architecture/00_system_architecture.md`](main/docs/architecture/00_system_architecture.md) for system boundaries and trust model.
4. Read [`main/docs/architecture/01_runtime_routing_and_invocation.md`](main/docs/architecture/01_runtime_routing_and_invocation.md) for end-to-end runtime flow.
5. Read [`main/KNOWLEDGE_BASE_ITEMS.md`](main/KNOWLEDGE_BASE_ITEMS.md) for output artifact schema and quality gates.

## Plain-language MVP workflow (current design intent)

1. A researcher submits an online source and required schema fields.
2. Local model (Ollama) performs first-pass extraction.
3. Extracted fields are written to canonical storage with provenance.
4. Missing fields are detected from SQL across unresolved jobs.
5. Unresolved records are grouped into batches by shared context.
6. Local model attempts batch fill under fixed time/confidence limits.
7. Only unresolved batches escalate to a single cloud fill call.
8. Cloud results re-enter through the same local write path with provenance.
9. Each resource job ends when required fields are complete.

## Key architectural changes versus common older AI integration patterns

- Local-first is explicit, with cloud as controlled fallback rather than default primary route.
- SQL-first completion scanning is formalized before secondary enrichment.
- Batched unresolved-field filling replaces ad hoc per-field escalation.
- Cloud is not allowed to write directly to storage; all writes are normalized through one path.
- Provenance is required per resolved field.

## Why this may help with prior integration struggles

- Reduces uncontrolled cloud usage by gating escalation.
- Improves reproducibility via explicit write path and run metadata expectations.
- Makes failure modes explicit (timeouts, low-confidence fills, retry/termination behavior).
- Separates architecture decisions from implementation details so tradeoffs can be reviewed first.

## Current implementation gap (important)

This repository currently documents architecture and cost models only. It does not yet include:

- resolver service implementation
- Ollama/cloud adapters
- deployment manifests
- operational scripts (backup/restore/metrics export)

## Terms used repeatedly

- `resolver-gateway`: orchestrates job flow, policy checks, and write control.
- `tau_local`: local time budget before cloud escalation.
- `q_min`: minimum confidence threshold for accepting local fill output.
- `mu_eff`: memory-constrained effective token throughput.
- `rho`: utilization ratio (`lambda_peak / mu_eff`), where values near or above `1` indicate queue risk.
