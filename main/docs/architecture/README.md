# Software Architecture Documentation

## Documents

- [`00_system_architecture.md`](00_system_architecture.md): system context, component boundaries, trust boundaries, and interfaces.
- [`01_runtime_routing_and_invocation.md`](01_runtime_routing_and_invocation.md): routing policy, execution flows, and proposed invocation contracts.
- [`02_deployment_and_operations.md`](02_deployment_and_operations.md): deployment topology, runtime controls, observability, and reliability operations.

## Scope Note

These documents now describe the MVP flow:

- initial local extraction pass over a user-fed online source via Ollama
- SQL-first persistence of extracted fields
- batched timed local fill attempts for remaining missing fields across unresolved entries
- cloud CLI fallback when local resolution exceeds time or confidence thresholds
- final per-resource upserts with provenance and per-resource termination on completion
