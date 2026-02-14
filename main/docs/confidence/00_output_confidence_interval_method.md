# Output Confidence Interval Method (MVP)

## 1) Purpose

This document defines a deployment-level statistical method for estimating uncertainty in generated output quality.

It is designed for this repository's local-first plus cloud-fallback workflow and aligns with:

- local acceptance threshold `q_min`
- observed acceptance metric `q_accept_obs`
- verification labels recorded during review

## 2) Measurement Unit and Labels

Pick one unit per reporting window and keep it fixed:

- per output item, or
- per required field

Each reviewed unit must be labeled with one of:

- `supported`: evidence-backed and accepted
- `contradicted`: incorrect relative to evidence
- `insufficient`: not enough evidence to accept

Recommended binary mapping for CI:

- accepted (`1`) = `supported`
- not accepted (`0`) = `contradicted` or `insufficient`

This keeps the acceptance estimate conservative and directly tied to hallucination risk.

## 3) Core Estimator

For a given window:

- `n` = number of reviewed units
- `x` = number of accepted units
- `p_hat = x / n` = observed acceptance rate

This is the same conceptual quantity as `q_accept_obs`.

## 4) 95% Confidence Interval (Wilson)

Use the Wilson interval instead of a normal approximation, especially when `n` is small or `p_hat` is near `0` or `1`.

Let `z = 1.96` for 95% confidence.

```math
center=\frac{p_{hat}+z^2/(2n)}{1+z^2/n}
```

```math
half=\frac{z\sqrt{p_{hat}(1-p_{hat})/n+z^2/(4n^2)}}{1+z^2/n}
```

```math
CI_{95\%}(p)=[center-half,\ center+half]
```

Interpretation:

- lower bound `L_p` = minimum plausible true acceptance rate
- upper bound `U_p` = maximum plausible true acceptance rate

## 5) Hallucination Interval

Define hallucination rate as:

```math
h = 1 - p
```

Convert the acceptance interval directly:

```math
CI_{95\%}(h)=[1-U_p,\ 1-L_p]
```

This avoids recomputing with a separate model.

## 6) Decision Rule for This Deployment

For each reporting window, choose target thresholds:

- minimum acceptable true acceptance rate: `p_target`
- maximum acceptable true hallucination rate: `h_max`
- minimum reviewed sample size: `n_min`

Pass window quality gate only if all are true:

```math
n \ge n_{min}
```

```math
L_p \ge p_{target}
```

```math
1-L_p \le h_{max}
```

Operational interpretation:

- if gate passes, keep current `q_min` and routing policy
- if gate fails, tighten `q_min` and/or increase cloud escalation until recalibration

## 7) Sample Size Planning

When planning a review batch size for margin of error `e`, use:

```math
n \approx \frac{z^2 p_0(1-p_0)}{e^2}
```

where `p_0` is expected acceptance rate.

Conservative default if uncertain: `p_0 = 0.5`.

## 8) Roll-Up Rules

For multi-day or multi-run reporting, aggregate counts first:

- `x_total = sum(x_i)`
- `n_total = sum(n_i)`
- compute one interval from `x_total, n_total`

Do not average per-run percentages; that biases the interval.

## 9) Snapshot Schema Additions

Existing snapshot fields already include:

- `q_accept_obs`
- `accepted_items`

To compute confidence intervals directly, add:

| Field | Unit | Description |
| --- | --- | --- |
| `reviewed_items` | count | Total reviewed units in snapshot window (`n`). |
| `accepted_items` | count | Accepted reviewed units (`x`). |
| `rejected_items` | count | Reviewed units not accepted (`n - x`). |
| `ci95_accept_lower` | 0..1 | Wilson lower bound for acceptance rate (`L_p`). |
| `ci95_accept_upper` | 0..1 | Wilson upper bound for acceptance rate (`U_p`). |
| `ci95_hallucination_upper` | 0..1 | Conservative upper bound on hallucination (`1 - L_p`). |

## 10) Minimal Reporting Block

Each window summary should include:

- `n`, `x`, `p_hat`
- `CI95_accept = [L_p, U_p]`
- `CI95_hallucination = [1-U_p, 1-L_p]`
- gate status (`pass` or `fail`) and configured thresholds

This is the minimum needed for reproducible quality governance across architecture tiers.
