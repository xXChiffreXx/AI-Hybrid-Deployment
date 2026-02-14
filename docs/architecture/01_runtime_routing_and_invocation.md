# Runtime Routing and Invocation Architecture

## Purpose

Define request lifecycle behavior and invocation contracts needed to implement the architecture.

## Routing Classes

Derived from budget docs:

- `local-default`: standard local route
- `local-long`: local route with larger context and stricter concurrency control
- `cloud-hard`: directly routed high-reasoning cloud path

## Core Routing Policy

1. classify request by task type, complexity, and policy tags.
2. if route class is `cloud-hard`, invoke cloud immediately.
3. otherwise check queue pressure and memory headroom.
4. attempt local route within configured local time budget.
5. if local route fails quality or exceeds budget, issue one cloud fill escalation call.
6. upsert final record with provenance.

## Cloud Reentry Rule

- cloud providers never write directly to canonical storage.
- every cloud response must pass through `gateway` normalization and then through `kb-writer`.
- mandatory persistence path: `cloud -> gateway -> kb-writer -> SQLite/Parquet`.

## End-to-End Sequence

```mermaid
sequenceDiagram
  actor U as User
  participant G as gateway
  participant D as canonical store
  participant Q as qdrant
  participant L as llama-server
  participant C as cloud llm
  participant W as kb-writer

  U->>G: submit knowledge item request
  G->>D: check complete existing item
  alt complete hit
    D-->>G: existing record
    G-->>U: return cached result
  else DB miss or incomplete
    G->>Q: retrieve domain context
    Q-->>G: context chunks
    G->>L: local invocation with time budget
    alt local success and quality pass
      L-->>G: completed item
    else timeout or quality fail
      G->>C: single fill escalation for missing fields
      C-->>G: filled fields
    end
    G->>W: upsert item plus provenance
    W-->>G: write success
    G-->>U: return final item
  end
```

## Overflow and Hard-Route Sequence

```mermaid
sequenceDiagram
  actor U as User
  participant G as gateway
  participant R as redis queue state
  participant L as llama-server
  participant C as cloud llm
  participant W as kb-writer

  U->>G: request
  G->>R: read pressure and policy counters
  R-->>G: current queue state

  alt route class is cloud-hard
    G->>C: invoke high reasoning route
    C-->>G: response
  else route class is local-default or local-long
    G->>L: attempt local route
    alt local overloaded or timeout
      G->>C: overflow route invocation
      C-->>G: response
    else local success
      L-->>G: response
    end
  end

  G->>W: persist response and provenance
  W-->>G: write success
  G-->>U: normalized response
```

## Invocation Contracts

These contracts are proposed interfaces to implement; they do not exist yet in this repository.

| Interface | Proposed Contract | Notes |
| --- | --- | --- |
| Client to gateway | `POST /v1/chat/completions` | OpenAI-compatible ingress to keep client tooling stable. |
| Gateway to local inference | `POST <llama-server>/v1/chat/completions` | request passthrough with policy-controlled model/context limits. |
| Gateway to cloud route | provider adaptor call | normalization layer should make cloud response shape match local response shape. |
| Gateway to kb-writer | internal RPC or message queue | write path should include `item_id`, provenance, verification fields, and route metadata. |
| Cloud direct to store | not allowed | cloud output must re-enter through gateway and `kb-writer`. |

## Planned Invocation Artifacts

Expected implementation artifacts for this architecture:

- gateway router service
- local invocation adapter
- cloud invocation adapter
- writer service integration module
- policy and threshold configuration file

Current status: none of these artifacts are present yet.
