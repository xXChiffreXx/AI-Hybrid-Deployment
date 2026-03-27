# Telemetry Pilot Plan (Summer 2026)

## Purpose

Replace modeled assumptions with observed measurements before the project treats any budget tier or routing policy as stable.

## Reference Pilot Window

Start with this reference configuration:

- architecture tier: `$2,500`
- model: `qwen2.5:14b`
- quantization: `q4_K_M`
- routing pattern: local-first plus single cloud fill escalation per unresolved batch

Pilot length:

- `14` calendar days or `25` completed sources, whichever is later

Snapshot cadence:

- one snapshot row every `24` hours
- one final roll-up at the end of the pilot window

## Required Snapshot Additions

In addition to the existing budget and confidence fields, every pilot window should record:

- `quantization_profile`
- `context_tokens`
- `concurrency`
- review counts needed for Wilson intervals

## Metrics That Must Be Observed

Collect the metrics already modeled in the repository, with special attention to:

- `mu_compute_obs`
- `mu_memory_obs`
- `mu_eff_obs`
- `memory_peak_gb`
- `h_db_obs`
- `p_escalate_obs`
- `cloud_fill_calls_obs`
- `c_fill_obs`
- `q_accept_obs`
- `ci95_accept_lower`
- `ci95_hallucination_upper`

## Decision Rules During the Pilot

Make changes in this order when problems appear:

1. reduce context or concurrency if fit is the issue
2. tighten `q_min` or lower `tau_local` if quality or latency is the issue
3. increase cloud use if unresolved carryover remains too high
4. only then consider switching quantization profile

## Exit Criteria

The pilot is complete when all are true:

- at least one full roll-up table can be populated with observed values
- the project has an observed `quantization_profile` decision, not just an assumed one
- the team can estimate `rho_obs`, cloud spend, and quality gates from real runs
- at least one `TB-03` Cost and Capacity Snapshot and one `TB-05` Routing Decision Record can be produced from the data

## Deliverables for the Project Lead

At the end of the telemetry pilot, prepare:

- an updated cross-architecture comparison table using observed values where available
- a quantization decision note
- a short recommendation on whether to stay on `$2,500` or move to `$5,000`
