# Knowledge Base Items the LLM Is Intended to Produce

## Scope

This project uses LLMs to generate structured knowledge artifacts for two tracks:

- **Track A (primary):** polypharmacy research data analysis.
- **Track B (supporting):** local/cloud LLM system operations.

The LLM is **not** intended to provide autonomous patient-specific clinical decisions.

Canonical storage is **not Markdown**. Markdown is optional as a human-readable view layer only.

## Track A Knowledge Base Items (Primary)

These are the core artifacts needed to build the research analysis knowledge base.

| ID      | Item Type                              | Intended Use                                        | Minimum Inputs                              | Required Outputs                                           |
| ------- | -------------------------------------- | --------------------------------------------------- | ------------------------------------------- | ---------------------------------------------------------- |
| `TA-01` | Research Question Card                 | Pin the exact hypothesis/endpoints for a study pass | Problem statement + sources                 | Question, endpoints, assumptions, exclusions               |
| `TA-02` | Cohort Logic Card                      | Define inclusion/exclusion and index-date logic     | Cohort criteria + data schema               | Cohort rules, code-like logic, edge cases                  |
| `TA-03` | Data Dictionary and Mapping Card       | Normalize meds/outcomes across sources              | Raw schema + RxNorm/OMOP/FHIR mappings      | Field dictionary, concept mappings, unresolved mappings    |
| `TA-04` | Data Quality and Leakage Check Note    | Record validity checks before modeling              | Dataset snapshot + split policy             | Missingness profile, leakage checks, action items          |
| `TA-05` | Endpoint Definition Card               | Freeze outcome labels/time windows                  | Clinical outcome definitions                | Label logic, observation windows, censoring notes          |
| `TA-06` | Feature Specification Card             | Define model features and construction logic        | Cohort + endpoint schema                    | Feature list, derivation logic, rationale                  |
| `TA-07` | Baseline Benchmark Note                | Capture rule/pairwise baseline results              | Baseline run logs                           | Metrics, calibration, limits, reproducibility refs         |
| `TA-08` | Proposed Model Benchmark Note          | Capture graph/temporal/proposed model results       | Experiment run logs                         | Metrics, comparison vs baseline, error summary             |
| `TA-09` | Subgroup and Sensitivity Note          | Track heterogeneity and robustness                  | Stratified run outputs                      | Subgroup metrics, sensitivity outcomes, risks              |
| `TA-10` | Regimen Risk and Brittleness Case Card | Explain set-level risk on exemplar cases            | De-identified regimen cases + model outputs | Risk decomposition, brittleness score, explanation         |
| `TA-11` | Counterfactual/Simulation Note         | Document medication-change simulations              | Scenario configs + outputs                  | Scenario table, expected impacts, caveats                  |
| `TA-12` | Causal Effect Estimation Note          | Capture effect-estimation methodology and outputs   | Target trial/causal setup + results         | Estimand, model spec, estimates, uncertainty               |
| `TA-13` | Failure Taxonomy Note                  | Track unsafe/incorrect recommendations              | Failed cases + evidence                     | Failure category, root-cause hypothesis, mitigations       |
| `TA-14` | Deployment Readiness Card              | Bridge analysis outputs to CDS/FHIR integration     | Model outputs + integration constraints     | CDS card schema mapping, override reasons, monitoring plan |

## Track B Knowledge Base Items (Supporting)

Track B items support reliable generation and governance of Track A artifacts.

| ID      | Item Type                  | Intended Use                                 | Minimum Inputs                            | Required Outputs                                               |
| ------- | -------------------------- | -------------------------------------------- | ----------------------------------------- | -------------------------------------------------------------- |
| `TB-01` | Workflow/Prompt Card       | Version reusable LLM workflows               | Prompt/pipeline config                    | Prompt template, routing intent, token profile                 |
| `TB-02` | Architecture Note          | Record local/cloud component design          | Constraints + components                  | Topology, trust boundaries, tradeoffs                          |
| `TB-03` | Cost and Capacity Snapshot | Track token spend and throughput assumptions | Token logs + pricing/hardware assumptions | Cost model table, break-even estimate, capacity recommendation |
| `TB-04` | Evaluation Result Note     | Compare LLM methodology quality/cost         | Run logs + verification labels            | Hallucination/safety/cost metrics, decision                    |
| `TB-05` | Routing Decision Record    | Preserve policy decisions over time          | Alternatives + constraints                | Selected route policy, rationale, revisit trigger              |

## Required Metadata for Every Item

All items must include:

- `item_id`
- `item_type`
- `title`
- `track` (`A` or `B`)
- `sector` (`clinical_guidelines`, `ddi_modeling`, `deployment_ops`)
- `created_at` (ISO timestamp)
- `authoring_method` (`human`, `llm`, or `llm+human-review`)
- `sources` (URLs/docs/papers/internal references)
- `verification_status` (`unverified`, `partially_verified`, `verified`)
- `confidence` (`low`, `medium`, `high`)
- `version`

If LLM-generated, include:

- `model_or_route`
- `prompt_hash` (or workflow version ID)

If analysis/result item (`TA-04` to `TA-14`), include:

- `dataset_snapshot_id`
- `split_policy`
- `run_id`
- `code_ref` (commit SHA or notebook/script reference)

## Quality Gates

Before accepting an item into the knowledge base:

1. Every non-trivial claim must be source-linked or evidence-linked.
2. Medical-risk claims require explicit verification label (`supported`, `contradicted`, or `insufficient`).
3. Analysis items must document leakage controls and split policy where relevant.
4. Benchmark items must include reproducibility fields (`run_id`, `code_ref`, dataset snapshot).
5. Outputs must be framed as research support, not patient-specific medical advice.

## Canonical Storage Model (Not Markdown)

Use a structured store as system-of-record:

- `SQLite` (transactional metadata + relational links)
- `Parquet` (large analysis tables, metrics, and claim/evidence matrices)

These table/file examples define integration expectations only. They are not a fixed production schema; physical schema should follow brittleness-analysis requirements.

Recommended canonical layout:

- `kb_store/sqlite/kb.sqlite`
- `kb_store/parquet/*.parquet` (for example: `items.parquet`, `claims.parquet`, `metrics.parquet`)

Minimum table groups:

- `items`: one row per `TA-*`/`TB-*` artifact
- `claims` + `evidence_links`: normalized claims and claim-to-source support labels
- `runs` + `metrics`: reproducibility metadata and evaluation outcomes

Markdown notes, if used, should be generated read-only views from this store, not hand-maintained source data.

## Minimum Intended Production Set (Per Research Cycle)

Track A minimum:

- 1+ `TA-02` Cohort Logic Card
- 1+ `TA-03` Data Dictionary and Mapping Card
- 1+ `TA-05` Endpoint Definition Card
- 1+ `TA-06` Feature Specification Card
- 2+ `TA-07`/`TA-08` benchmark notes
- 1+ `TA-09` Subgroup and Sensitivity Note
- 3+ `TA-10` Regimen Risk and Brittleness Case Cards
- 1+ `TA-12` Causal Effect Estimation Note
- 1+ `TA-13` Failure Taxonomy Note

Track B minimum:

- 1+ `TB-01` Workflow/Prompt Card
- 1+ `TB-03` Cost and Capacity Snapshot
- 1+ `TB-05` Routing Decision Record

This ensures the knowledge base is led by Track A research analysis outputs, with Track B serving reliability and cost governance.

## View-Layer Mapping (Tool-Agnostic)

Any team can use any read tool. Canonical storage remains the source-of-truth.

### Track-level mapping fields

Every item row in the canonical store must carry:

- `track` (`A` or `B`)
- `item_type` (`TA-*` or `TB-*`)
- `view_track` (consumer grouping label, e.g. `track_a` or `track_b`)
- `view_logic_group` (logical bucket, e.g. `cohort_logic`, `baseline_benchmarks`)

Use deterministic, human-readable `view_logic_group` names and keep the mapping table in code/config (not handwritten notes). Example mappings:

- `TA-02` -> `cohort_logic`
- `TA-10` -> `risk_brittleness_cases`
- `TB-03` -> `cost_capacity_snapshots`

### Derived view strategy

If you want notes, dashboards, or exports, generate them from the store:

- one index note per logical group
- optional per-item summary note with canonical `item_id` and deep link to store record
- no manual editing of generated views
