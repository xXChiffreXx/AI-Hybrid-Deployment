# Qwen Budget-Tier Hallucination Standard (MVP)

## 1) Scope

This document defines how to represent Qwen model quality across budget tiers for this deployment.

It complements:

- confidence interval method: `00_output_confidence_interval_method.md`
- budget tier model mapping in `docs/budgets/10_cost_performance_writeup.md`

## 2) Budget to Model Mapping

Current default local model mapping by budget tier:

| Budget Tier | Local Model Profile |
| --- | --- |
| `$599` | `qwen2.5:7b` |
| `$2,500` | `qwen2.5:14b` |
| `$5,000` | `qwen2.5:32b` |
| `$7,500` | `qwen2.5:72b` |

## 3) "Standard Hallucination Rate" Clarification

There is no single universal hallucination rate for a model family that is valid across all tasks, prompts, retrieval context, and evaluation protocols.

For governance, this repository defines a deployment standard from observed, verified outputs:

- observed hallucination in window `w`: `h_obs,w = 1 - x_w / n_w`
- conservative upper bound in window `w`: `h_upper,w = 1 - L_p,w`
  - where `L_p,w` is the Wilson lower bound for acceptance from the CI method

## 4) Deployment Standard by Budget Tier

For each budget tier `b`, calculate over the most recent `K` reporting windows (recommended: `K = 3`):

```math
h_{std,b}=med(h_{obs,w}),\quad w \in W_b
```

```math
h_{guard,b}=P90(h_{upper,w}),\quad w \in W_b
```

Interpretation:

- `h_std,b`: standard operating hallucination rate for that budget/model tier
- `h_guard,b`: guardrail rate for risk-sensitive gating
- `med(...)`: sample median across the selected windows

## 5) Quality Gate Use

A budget tier is in-policy only if all are true for the current window:

```math
n \ge n_{min}
```

```math
h_{upper,current} \le h_{max,policy}
```

```math
h_{upper,current} \le h_{guard,b}
```

If any condition fails, tighten `q_min` and/or route more unresolved batches to cloud fallback until recalibration.

## 6) Reporting Table Template

Populate this table each reporting window:

| Window | Budget Tier | Model | Reviewed `n` | Accepted `x` | `h_obs` | `h_upper` (95%) | `h_std,b` (rolling) | `h_guard,b` (rolling) | Gate |
| --- | --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | --- |
| `win-001` | `$599` | `qwen2.5:7b` |  |  |  |  |  |  |  |
| `win-001` | `$2,500` | `qwen2.5:14b` |  |  |  |  |  |  |  |
| `win-001` | `$5,000` | `qwen2.5:32b` |  |  |  |  |  |  |  |
| `win-001` | `$7,500` | `qwen2.5:72b` |  |  |  |  |  |  |  |

## 7) Pre-Telemetry Default

Before enough windows exist for per-tier rolling standards, use one temporary policy cap:

- `h_max,policy = 0.20` (conservative startup cap)
- `n_min = 200` reviewed units per window

Replace these defaults after the first `K` windows with measured `h_std,b` and `h_guard,b`.
