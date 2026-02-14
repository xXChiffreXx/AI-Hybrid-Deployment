# Software Architecture Documentation

This folder contains implementation-oriented architecture docs derived from the current budget models and repository assumptions.

## Documents

- [`00_system_architecture.md`](00_system_architecture.md): system context, component boundaries, trust boundaries, and interfaces.
- [`01_runtime_routing_and_invocation.md`](01_runtime_routing_and_invocation.md): routing policy, execution flows, and proposed invocation contracts.
- [`02_deployment_and_operations.md`](02_deployment_and_operations.md): deployment topology, runtime controls, observability, and reliability operations.

## Scope Note

These documents now describe the MVP flow:

- initial local extraction pass over a user-fed online source via Ollama
- SQL-first persistence of extracted fields
- timed local fill attempts for remaining missing fields
- cloud CLI fallback when local resolution exceeds time or confidence thresholds
- final upsert with provenance and per-resource termination on completion

The repository still does not include the corresponding runtime implementation (gateway/resolver service, invocation scripts, or deployment manifests).
