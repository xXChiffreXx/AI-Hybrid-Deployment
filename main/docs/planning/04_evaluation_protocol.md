# Evaluation Protocol (Summer 2026 Pilot)

## Purpose

Operationalize the confidence documents into a reviewer workflow that can be used from the first pilot window.

## Primary Evaluation Questions

- Are the generated structured fields source-grounded and reviewable?
- When should unresolved work stay local versus escalate to cloud?
- Does the chosen quantization profile preserve enough quality for research-support outputs?

## Review Unit

Use required fields as the primary review unit for pilot windows.

Secondary roll-up unit:

- completed artifact item

Field-level review is the better first choice because the routing and batching design resolves missing fields one field group at a time.

## Review Labels

Use the same labels as the confidence docs:

- `supported`
- `contradicted`
- `insufficient`

Binary mapping for acceptance:

- accepted = `supported`
- not accepted = `contradicted` or `insufficient`

## Startup Window Policy

Use these default thresholds until enough pilot telemetry exists to replace them:

- minimum reviewed sample size per window: `n_min = 200`
- target acceptance rate: `p_target = 0.80`
- startup hallucination cap: `h_max,policy = 0.20`

Compute the Wilson interval exactly as defined in [`../confidence/00_output_confidence_interval_method.md`](../confidence/00_output_confidence_interval_method.md).

## Sampling Procedure

For the first three reporting windows:

1. review every generated output from the first `10` sources if volume allows
2. after bootstrapping, continue sampling until the window reaches at least `200` reviewed field-level units
3. include both local-only and local-plus-cloud outputs if both routes occur in the window
4. treat a change in model, quantization, prompt, or routing policy as a new evaluation window

## Reviewer Workflow

1. reviewer opens the generated artifact and the linked evidence
2. reviewer labels each checked field as `supported`, `contradicted`, or `insufficient`
3. reviewer flags any medically salient contradiction for faculty or domain review
4. the project records `n`, `x`, Wilson bounds, and gate status for the window

## Failure Response

If a window fails the quality gate:

1. tighten `q_min`
2. increase cloud escalation for unresolved batches
3. move to a less aggressive quantization or lower concurrency if needed
4. open or update a `TA-13` Failure Taxonomy Note and a `TB-05` Routing Decision Record

## Minimal Reporting Block

Each pilot window should report:

- model and `quantization_profile`
- `n`, `x`, and `p_hat`
- `CI95_accept`
- `CI95_hallucination`
- gate status
- counts of local-only versus local-plus-cloud reviewed outputs

## Guardrail

Outputs remain research-support artifacts only. Any result that could be interpreted as patient-specific medical advice must be marked as out of scope for the MVP.
