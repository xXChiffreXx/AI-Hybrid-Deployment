# Pilot Track A Scope (Summer 2026)

## Purpose

Freeze a narrow Track A scope so the summer project validates the local/cloud routing architecture on artifacts that can be reviewed reproducibly.

## Scope Rule

The summer pilot should optimize for structured research-support outputs with clear provenance and reviewability. It should not try to cover the full downstream clinical or causal-analysis surface area in the first build.

## In-Scope Items

| Item | Pilot Status | Why It Is In Scope |
| --- | --- | --- |
| `TA-01` Research Question Card | prerequisite | stabilizes the hypothesis, endpoints, and exclusions before automation |
| `TA-02` Cohort Logic Card | core | foundational for all later analysis artifacts |
| `TA-03` Data Dictionary and Mapping Card | core | essential for cross-source normalization in a polypharmacy setting |
| `TA-05` Endpoint Definition Card | core | makes label logic explicit before benchmark work |
| `TA-06` Feature Specification Card | core | ties cohort and endpoint logic to reproducible model inputs |
| `TA-07` Baseline Benchmark Note | core | gives the project a non-LLM or simpler-model baseline |
| `TA-08` Proposed Model Benchmark Note | core | captures the main model comparison target |
| `TA-13` Failure Taxonomy Note | core | creates a disciplined mechanism for documenting unsafe or incorrect outputs |
| `TA-09` Subgroup and Sensitivity Note | optional extension | useful if the core pipeline becomes stable early enough |

## Deferred Items

| Item | Pilot Status | Reason for Deferral |
| --- | --- | --- |
| `TA-10` Regimen Risk and Brittleness Case Card | defer | requires more mature exemplar selection and explanation tooling |
| `TA-11` Counterfactual/Simulation Note | defer | depends on stable scenario-design assumptions that the MVP does not yet encode |
| `TA-12` Causal Effect Estimation Note | defer | causal work deserves a dedicated design and evaluation track |
| `TA-14` Deployment Readiness Card | defer | only makes sense after the research outputs and failure modes are already stable |

## Acceptance Rules for In-Scope Items

Every in-scope item accepted into the pilot knowledge base should include:

- required metadata from [`../../KNOWLEDGE_BASE_ITEMS.md`](../../KNOWLEDGE_BASE_ITEMS.md)
- source or evidence links for every non-trivial claim
- verification labels for medical-risk claims
- reproducibility fields when the item is analysis- or benchmark-oriented

## Summer Deliverable Shape

By the end of the summer, the project should be able to show:

- at least one complete pass through the core in-scope items for a seed corpus
- a clear record of which outputs remained human-authored, local-only, or local-plus-cloud
- an explicit list of deferred items for the next project phase
