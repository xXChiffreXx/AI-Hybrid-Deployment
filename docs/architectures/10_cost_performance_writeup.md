# Cost-to-Performance Writeup

This document compares the four machine-budget architectures for knowledge-base production.

## 1) Industry-Standard Modeling Framework

### 1.1 Cloud Cost

From `00_cloud_cost_model.md`:

```math
C_{month}=S\left(r_hc_h+(1-r_h)r_{over}c_e\right)
```

Where:
- $S$: sources/month
- $r_h$: hard-task cloud fraction
- $r_{over}$: overflow fraction from local capacity limits
- $c_e, c_h$: per-source economy and high-reasoning cloud costs

### 1.2 Throughput and Utilization

Effective local throughput is $\mu$ tokens/sec.

Peak utilization:

```math
\rho = \frac{\lambda_{peak}}{\mu}
```

Stability condition for queueing systems:

```math
\rho < 1
```

### 1.3 Source Completion Time

Approximate local source completion time:

```math
t_{source} \approx \frac{T_{in}+T_{out}}{\mu}
```

### 1.4 CapEx Amortization (Power Ignored)

Monthly amortized machine cost over $L$ months:

```math
A_{month}=\frac{CapEx}{L}
```

Total monthly cost:

```math
TC_{month}=A_{month}+C_{month}
```

Cost per source:

```math
C_{source}=\frac{TC_{month}}{S}
```

## 2) Runtime Complexity (Big O)

Let:
- $N$: number of sources
- $n$: average tokens per source
- $M$: corpus size in retrieval index

### 2.1 Pipeline Stages

- Ingest + parse + chunk: $O(n)$
- Embedding generation (fixed chunk size): $O(n)$
- Retrieval query (ANN/HNSW-style): typically $O(\log M)$ average-case
- Generation:
  - Transformer prefill: $O(L_{in}^2)$
  - Autoregressive decode: $O(L_{out}L_{ctx})$

### 2.2 End-to-End Expected Complexity

With fixed chunk/context caps (typical production setup), per-source runtime is effectively linear:

```math
T_{source}=O(n)
```

Corpus runtime becomes:

```math
T_{corpus}=O(Nn)
```

Without context controls (naive long-context growth), generation can trend toward superlinear behavior dominated by attention:

```math
T_{source}=O(n^2)
```

## 3) Parameter Set Used for Comparison

- $T_{in}=600{,}000$, $T_{out}=125{,}000$
- $S=130$ baseline sources/month
- $S_{stress}=234$ stress sources/month
- $L=36$ months amortization
- $\lambda_{peak,base}=145.45$, $\lambda_{peak,stress}=261.81$ tokens/sec

## 4) Comparative Results

| Budget | $\mu$ (tokens/sec) | Local-only Sources/Day | $t_{source}$ (hours) | $\rho_{base}$ | $\rho_{stress}$ | Cloud Cost/Month (Base) | Cloud Cost/Month (Stress) | Amortized CapEx/Month | Total Cost/Month (Base) | Cost/Source (Base) |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| `$599` | `60` | `7.15` | `3.36` | `2.42` | `4.36` | `$74.57` | `$145.12` | `$16.64` | `$91.21` | `$0.70` |
| `$2,500` | `90` | `10.73` | `2.24` | `1.62` | `2.91` | `$57.55` | `$121.07` | `$69.44` | `$126.99` | `$0.98` |
| `$5,000` | `180` | `21.46` | `1.12` | `0.81` | `1.45` | `$26.46` | `$69.46` | `$138.89` | `$165.35` | `$1.27` |
| `$7,500` | `320` | `38.18` | `0.63` | `0.45` | `0.82` | `$17.64` | `$31.75` | `$208.33` | `$225.97` | `$1.74` |

## 5) Interpretation

- The `$599` and `$2,500` options are cheapest in strict dollar terms, but both are unstable under modeled peak loads ($\rho>1$).
- The `$5,000` option is stable at baseline and unstable in stress months; it is a balanced middle ground.
- The `$7,500` option is stable in both baseline and stress scenarios and gives strongest long-term throughput headroom.

## 6) Recommendation by Decision Priority

- If priority is lowest upfront cost: `$599` option.
- If priority is best balance of cost and usable throughput: `$5,000` option.
- If priority is long-term institutional throughput and lower queue risk: `$7,500` option.

## 7) Recalibration Procedure

After 2-4 weeks of production telemetry, update:

1. $\mu$ from observed local tokens/sec.
2. $r_h$ from actual hard-route fraction.
3. $r_{over}$ from queue spill metrics.
4. $T_{in}, T_{out}$ from observed medians.
5. Recompute costs and $\rho$ using the same equations.
