# Cost-to-Performance Writeup

This document compares the four machine-budget architectures for Track A knowledge-base production.

## 1) Industry-Standard Comparison

| Industry analysis mode | Common source baseline | Previous doc state | Current state |
|---|---|---|---|
| Unit economics (`$/source`, `$/accepted-item`) | FinOps unit economics | Partial | Included |
| Throughput/latency/concurrency | MLPerf + Triton optimization practice | Included | Included |
| Memory-first feasibility before throughput claims | TensorRT-LLM + HF optimization guidance | Missing | Included |
| Tail-latency/SLO framing (not only average throughput) | SRE practice | Missing | Included |
| Stress/scenario modeling | Capacity planning practice | Partial | Included |

Relevant mode that was under-modeled before: memory-constrained throughput (`\mu_{eff}`) plus SLO/tail-latency behavior. Both are now explicit.

## 2) Core Equations Used

Cloud monthly cost:

```math
C_{month}=S\left(r_hc_h+(1-r_h)r_{over}c_e\right)
```

Memory-constrained effective throughput:

```math
\mu_{eff}=f_{fit}\cdot\min(\mu_{compute},\mu_{memory})
```

Utilization and stability:

```math
\rho = \frac{\lambda_{peak}}{\mu_{eff}}, \quad \rho < 1
```

Source completion time:

```math
t_{source} \approx \frac{T_{in}+T_{out}}{\mu_{eff}}
```

CapEx amortization (power excluded by scope):

```math
A_{month}=\frac{CapEx}{L}, \quad TC_{month}=A_{month}+C_{month}
```

Quality-adjusted unit economics:

```math
N_{accept}=S\cdot q_{accept}, \quad C_{accept}=\frac{TC_{month}}{N_{accept}}
```

SLO queueing (directional, stable regime):

```math
W_q = \frac{\rho}{\mu_{eff}-\lambda}, \quad R = \frac{1}{\mu_{eff}} + W_q
```

## 3) Parameter Set Used for Comparison

- $T_{in}=600{,}000$, $T_{out}=125{,}000$
- $S=40$ baseline sources/month
- $S_{stress}=72$ stress sources/month
- $L=36$ months amortization
- $\lambda_{peak,base}=44.75$, $\lambda_{peak,stress}=80.56$ tokens/sec
- Memory safety factor: $\eta=0.85$

## 4) Memory Envelope and Effective Throughput

| Budget | Installed Memory Envelope | Usable Envelope ($\eta M_{avail}$) | $f_{fit}$ | $\mu_{eff}$ (tokens/sec) |
|---|---:|---:|---:|---:|
| `$599` | `16-24 GB` | `13.6-20.4 GB` | `0.95` | `60` |
| `$2,500` | `64-96 GB` | `54.4-81.6 GB` | `1.00` | `90` |
| `$5,000` | `128 GB` | `108.8 GB` | `1.00` | `180` |
| `$7,500` | `192-256 GB` | `163.2-217.6 GB` | `1.00` | `320` |

## 5) Comparative Results

| Budget | $\mu_{eff}$ (tokens/sec) | Local-only Sources/Day | $t_{source}$ (hours) | $\rho_{base}$ | $\rho_{stress}$ | Cloud Cost/Month (Base) | Cloud Cost/Month (Stress) | Amortized CapEx/Month | Total Cost/Month (Base) | Cost/Source (Base) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| `$599` | `60` | `7.15` | `3.36` | `0.75` | `1.34` | `$16.96` | `$35.21` | `$16.64` | `$33.60` | `$0.84` |
| `$2,500` | `90` | `10.73` | `2.24` | `0.50` | `0.90` | `$13.57` | `$24.43` | `$69.44` | `$83.01` | `$2.08` |
| `$5,000` | `180` | `21.46` | `1.12` | `0.25` | `0.45` | `$8.14` | `$14.66` | `$138.89` | `$147.03` | `$3.68` |
| `$7,500` | `320` | `38.18` | `0.63` | `0.14` | `0.25` | `$5.43` | `$9.77` | `$208.33` | `$213.76` | `$5.34` |

## 6) Interpretation Against Industry Thresholds

- Stability threshold (`\rho < 1`): `$599` fails stress; `$2,500`, `$5,000`, and `$7,500` pass baseline and stress.
- Tail-latency headroom target (`\rho \le 0.70`): `$2,500`, `$5,000`, and `$7,500` pass baseline; `$5,000` and `$7,500` pass stress.
- Memory-fit risk: `$599` is the only class with materially narrow memory headroom and therefore the highest operational sensitivity.

## 7) Additional Analysis Mode Still Worth Adding

One relevant mode still not fully integrated into the monthly equation is storage/network growth cost for long-lived datasets and backups:

```math
TC_{month}^{ext}=TC_{month}+C_{storage}+C_{network}
```

This is usually smaller than inference cost early, but becomes material as corpus size and backup retention expand.

## 8) Recommendation by Decision Priority

- If priority is minimum upfront spend and pilot-only operation: `$599`.
- If priority is balanced throughput and cost: `$5,000`.
- If priority is institutional durability, lower queue risk, and better SLO margin: `$7,500`.

## 9) Recalibration Procedure

After 2-4 weeks of production telemetry, update:

1. $\mu_{compute}$ and $\mu_{memory}$ from observed load tests.
2. $f_{fit}$ from observed memory pressure behavior.
3. $r_h$ from actual hard-route fraction.
4. $r_{over}$ from queue spill metrics.
5. $T_{in}, T_{out}$ from observed medians.
6. Recompute costs, utilization, and queue-risk indicators.

## 10) Snapshot Roll-Up Table (Cross-Architecture)

Populate this table from the per-architecture snapshot entries to compare real pilot performance.

| Snapshot ID | Git Commit | Date (UTC) | Architecture | Model Profile | Context | Concurrency | Sources/Day Observed | `mu_eff_obs` (tok/s) | `rho_obs` | `r_over_obs` | Peak Memory (GB) | Cloud Spend Window (USD) | `q_accept_obs` | Notes |
|---|---|---|---|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---|
| snap-001 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| snap-002 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |

Use this roll-up to replace model values with observed medians in Sections 3-6 after the pilot window.

## 11) Sources

- FinOps Unit Economics: https://www.finops.org/framework/capabilities/unit-economics/
- AWS Well-Architected Cost Optimization: https://docs.aws.amazon.com/wellarchitected/latest/framework/a-cost-optimization.html
- MLPerf Inference (datacenter scenarios): https://mlcommons.org/benchmarks/inference-datacenter/
- NVIDIA Triton optimization: https://docs.nvidia.com/deeplearning/triton-inference-server/archives/triton-inference-server-2610/user-guide/docs/optimization.html
- TensorRT-LLM memory reference: https://nvidia.github.io/TensorRT-LLM/reference/memory.html
- Google SRE SLO chapter: https://sre.google/sre-book/service-level-objectives/
