# Shared Cloud Cost Model

This model is used by all three budget architectures.

## 1) Project Scope Model

Assumed semester operating profile for a small research team:

- Team size: `5` researchers
- Sources ingested per researcher per week: `6`
- Active weeks/month: `4.3`

Monthly source volume:

\[
S = 5 \cdot 6 \cdot 4.3 = 129 \approx 130 \text{ sources/month (baseline)}
\]

Stress-case volume:

\[
S_{stress} = 1.8 \cdot S = 234 \text{ sources/month}
\]

Per-source token envelope for Track A knowledge item generation:

- Input tokens: \(T_{in} = 600{,}000\)
- Output tokens: \(T_{out} = 125{,}000\)

## 2) Pricing Variables

Two cloud routes are modeled:

- Economy route (e.g., mini model):
  - \(p^{e}_{in} = 0.25\) USD / 1M input tokens
  - \(p^{e}_{cache} = 0.025\) USD / 1M cached input tokens
  - \(p^{e}_{out} = 2.00\) USD / 1M output tokens
- High-reasoning route (frontier model):
  - \(p^{h}_{in} = 1.25\)
  - \(p^{h}_{cache} = 0.125\)
  - \(p^{h}_{out} = 10.00\)

Cache hit rate assumption:

\[
h = 0.45
\]

Per-source cloud cost for each route:

\[
c_e = \frac{(1-h)T_{in}p^{e}_{in} + hT_{in}p^{e}_{cache} + T_{out}p^{e}_{out}}{10^6} = 0.33925
\]

\[
c_h = \frac{(1-h)T_{in}p^{h}_{in} + hT_{in}p^{h}_{cache} + T_{out}p^{h}_{out}}{10^6} = 1.69625
\]

Units: USD per source.

## 3) Throughput -> Overflow Model

Let local inference capacity be \(\mu\) tokens/sec.

Peak token demand is modeled as:

\[
\lambda_{peak} = \phi \cdot \frac{S(T_{in}+T_{out})}{30 \cdot 24 \cdot 3600}
\]

where \(\phi = 4\) is a concentration factor for daytime/batch peaks.

Overflow fraction of non-hard tasks to cloud:

\[
r_{over} = \max\left(0, \frac{\lambda_{peak} - \mu}{\lambda_{peak}}\right)
\]

Fixed hard-task cloud fraction for a given architecture: \(r_h\).

## 4) Monthly Cloud Cost Equation

\[
C_{month} = S \cdot \left(r_h c_h + (1-r_h)r_{over}c_e\right)
\]

Interpretation:
- hard tasks always use cloud high-reasoning route,
- overflow of remaining tasks uses economy cloud route.

## 5) Baseline Constants for This Project

- Baseline \(S=130\): \(\lambda_{peak}=145.45\) tokens/sec
- Stress \(S=234\): \(\lambda_{peak}=261.81\) tokens/sec

## 6) Uncertainty Model (Optional)

If monthly source count is random with \(S \sim \text{Poisson}(\mu_S)\), define per-source expected cloud cost \(k\):

\[
E[C] = \mu_S k, \quad \text{Var}(C) = \mu_S k^2
\]

Approximate 95% interval:

\[
C \approx E[C] \pm 1.96\sqrt{\mu_S}k
\]

## 7) Recalibration Rule

At end of each month, replace assumptions with telemetry:

- replace \(T_{in}, T_{out}\) with observed medians per source,
- replace \(h\) with observed cache hit rate,
- replace \(\phi\) with observed peak/average demand ratio,
- replace \(r_h\) with observed hard-route fraction,
- recompute \(C_{month}\) for the next planning cycle.

## 8) Token-Volume Sensitivity

If observed tokens per source differ from planning assumptions, define:

\[
\alpha = \frac{T^{obs}_{in}+T^{obs}_{out}}{T_{in}+T_{out}}
\]

Then cloud cost scales approximately linearly:

\[
C^{obs}_{month} \approx \alpha \cdot C_{month}
\]

Example: if real workloads are 3x the planned token envelope, expected cloud cost is approximately 3x.
