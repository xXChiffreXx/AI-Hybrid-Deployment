# Summer 2026 Project Brief

## Purpose

Translate this repository's architecture and budget analysis into a concrete pilot recommendation for a professor-led summer polypharmacy project.

## Recommended Reference Configuration

For the Summer 2026 pilot, use the `$2,500` tier as the reference deployment target.

Reference profile:

- Budget tier: `$2,500`
- Local model: `qwen2.5:14b`
- Reference quantization: `q4_K_M`
- Local serving path: `ollama-server`
- Orchestration policy: `resolver-gateway`
- Cloud policy: one cloud fill escalation per unresolved batch only after local `tau_local` or `q_min` failure

Why this tier is the reference:

- It keeps the same local-first architecture as the higher tiers.
- It materially improves memory headroom over the `$599` class without asking for the larger capital commitment of the `$5,000` or `$7,500` tiers.
- It remains within the modeled stress stability threshold at the current workload assumptions (`rho_stress = 0.90 < 1`).
- It is easier to justify as a serious summer pilot than the `$599` tier, which has the narrowest memory headroom and the highest queue sensitivity.

Upgrade note:

- If the project becomes a permanent shared lab service, expects heavier concurrent use, or needs larger local contexts, the preferred next step is the `$5,000` tier.

## Recommended Pilot Scope

Treat the summer as a validation pilot for research-support artifacts, not a clinical decision-support deployment.

Human-led prerequisite:

- `TA-01` Research Question Card should be fixed before automated runs so the extraction target and review criteria are stable.

Core summer artifact set:

- `TA-02` Cohort Logic Card
- `TA-03` Data Dictionary and Mapping Card
- `TA-05` Endpoint Definition Card
- `TA-06` Feature Specification Card
- `TA-07` Baseline Benchmark Note
- `TA-08` Proposed Model Benchmark Note
- `TA-13` Failure Taxonomy Note

Optional extension if the core path is stable:

- `TA-09` Subgroup and Sensitivity Note

Defer until after pilot validation:

- `TA-10` Regimen Risk and Brittleness Case Card
- `TA-11` Counterfactual/Simulation Note
- `TA-12` Causal Effect Estimation Note
- `TA-14` Deployment Readiness Card

These deferred items depend on more mature modeling, simulation, causal design, or downstream CDS assumptions than the MVP currently guarantees.

## Summer Build Priorities

1. Implement the controlled canonical write path.
2. Implement local extraction plus unresolved-batch planning.
3. Add cloud fill fallback, telemetry, and snapshot reporting.
4. Run reviewed pilot windows and recalibrate quantization, `tau_local`, and `q_min`.

## Success Criteria

A successful summer pilot should demonstrate all of the following:

- end-to-end generation of the core Track A artifacts for a seed corpus
- provenance on each resolved field
- reviewed quality windows that satisfy the startup hallucination cap
- enough telemetry to replace modeled assumptions with measured values

## Intended Audience

This document is meant to help a project lead decide what to build first, what hardware commitment is justified for the summer, and how narrow the first pilot should stay.
