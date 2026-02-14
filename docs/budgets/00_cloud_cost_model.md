# Shared Cloud Cost + Capacity Model

This model is used by all budget architectures.

It aligns with common industry analysis modes for inference systems:

- unit economics (FinOps)
- memory-feasibility constraints
- bandwidth/compute bottleneck analysis
- queueing and SLO/tail-latency behavior under bursty load

## 1) Project Scope Model

Assumed pilot operating profile for a small research team:

- Team size: `5` researchers
- Sources ingested per researcher per week (pilot): `2`
- Active weeks/month: `4`

Monthly source volume:

```math
S = 5 \cdot 2 \cdot 4 = 40 \text{ sources/month (baseline)}
```

Stress-case source volume:

```math
S_{stress} = 1.8 \cdot S = 72 \text{ sources/month}
```

Per-source token envelope for Track A knowledge item generation:

- Input tokens: $T_{in} = 600{,}000$
- Output tokens: $T_{out} = 125{,}000$

## 2) Cloud Unit Economics Variables

Two cloud routes are modeled:

- Economy route:
  - $p^{e}_{in} = 0.25$ USD / 1M input tokens
  - $p^{e}_{cache} = 0.025$ USD / 1M cached input tokens
  - $p^{e}_{out} = 2.00$ USD / 1M output tokens
- High-reasoning route:
  - $p^{h}_{in} = 1.25$
  - $p^{h}_{cache} = 0.125$
  - $p^{h}_{out} = 10.00$

Cache hit-rate assumption:

```math
h = 0.45
```

Per-source cloud cost by route:

```math
c_e = \frac{(1-h)T_{in}p^{e}_{in} + hT_{in}p^{e}_{cache} + T_{out}p^{e}_{out}}{10^6} = 0.33925
```

```math
c_h = \frac{(1-h)T_{in}p^{h}_{in} + hT_{in}p^{h}_{cache} + T_{out}p^{h}_{out}}{10^6} = 1.69625
```

Units: USD per source.

## 3) Memory Feasibility Model

Local throughput claims are valid only if the model and runtime state fit memory.

Memory requirement decomposition:

```math
M_{req} = M_w + M_{kv} + M_{act} + M_{sys}
```

Weights memory (approximation):

```math
M_w \approx \frac{P \cdot b_w}{8}
```

where $P$ is number of parameters and $b_w$ is weight precision in bits.

KV-cache memory (decoder-only, approximate):

```math
M_{kv} \approx 2 \cdot L \cdot n_{kv} \cdot d_h \cdot T_{ctx} \cdot B \cdot b_{kv}
```

where:

- $L$: layers
- $n_{kv}$: KV heads
- $d_h$: head dimension
- $T_{ctx}$: active tokens/request
- $B$: concurrent active requests
- $b_{kv}$: bytes/element

Feasibility constraint:

```math
M_{req} \le \eta \cdot M_{avail}
```

with safety factor $\eta$ (planning value: `0.80` to `0.90`).

## 4) Effective Service Rate (Compute + Bandwidth + Memory)

Let:

- $\mu_{compute}$ = compute-limited token service rate
- $\mu_{memory}$ = memory-bandwidth-limited token service rate
- $f_{fit} \in [0,1]$ = memory pressure multiplier (`1` if fit with headroom, `<1` if cache pressure/offload/fragmentation lowers realized throughput, `0` if local route disabled)

```math
\mu_{eff} = f_{fit} \cdot \min(\mu_{compute}, \mu_{memory})
```

A roofline-style bandwidth bound:

```math
\mu_{memory} \le \frac{BW_{mem}}{I_{bytes/token}}
```

where $BW_{mem}$ is effective memory bandwidth and $I_{bytes/token}$ is bytes moved per generated token.

## 5) Arrival, Queue Pressure, and Overflow

Peak demand model (tokens/sec):

```math
\lambda_{peak} = \phi \cdot \frac{S(T_{in}+T_{out})}{30 \cdot 24 \cdot 3600}
```

where $\phi = 4$ models daytime/batch concentration.

Utilization:

```math
\rho = \frac{\lambda_{peak}}{\mu_{eff}}
```

Stability condition:

```math
\rho < 1
```

Overflow fraction of non-hard tasks to cloud:

```math
r_{over} = \max\left(0, \frac{\lambda_{peak} - \mu_{eff}}{\lambda_{peak}}\right)
```

Hard-task cloud fraction is policy-driven and fixed per architecture: $r_h$.

## 6) Monthly Cloud Cost Equation

```math
C_{month} = S \cdot \left(r_h c_h + (1-r_h)r_{over}c_e\right)
```

Interpretation:

- hard tasks always use the high-reasoning cloud route
- overflow of remaining tasks uses the economy route

## 6A) Web-Aware Enrichment + Batch Escalation Policy

This project also uses an MVP per-resource enrichment workflow aligned with architecture docs:

1. Ingest each source and required schema fields.
2. Run an initial local extraction pass with Ollama per source and upsert partial fields to canonical storage.
3. Run required-field completion checks from SQL to enumerate unresolved fields across unresolved resources.
4. Build unresolved-entry batches using shared source context and missing-field shape.
5. Apply timed local fill attempts with budget $\tau_{local}$ per unresolved batch.
6. If local timing/confidence thresholds fail, issue one cloud fill call for the unresolved batch payload with schema context, known fields, and explicit missing-field list.
7. Re-enter cloud output through resolver normalization plus `kb-writer` with per-resource upserts, then repeat completion checks until termination.

The cost equations below remain normalized per source. Batching gains are captured operationally through observed escalation rate, call counts, and fill-call cost.

Policy variables:

- $h_{db}$: DB complete-hit rate after initial extraction/upsert and completion checks
- $\tau_{local}$: local time budget per unresolved batch during fill attempts
- $p_{escalate}$: escalation probability after local batch budget
- $c_{fill}$: cost of one cloud fill call for an unresolved batch payload
- $m_{miss}$: average missing-field count when escalation is needed

Escalation probability definition:

```math
p_{escalate} = \Pr\big(t_{local,batch}>\tau_{local}\ \text{or}\ q_{local,batch}<q_{min}\mid \text{unresolved after initial upsert, not overflow}\big)
```

Expected cloud fill calls/month:

```math
N_{fill}=S\cdot(1-r_h)\cdot(1-r_{over})\cdot(1-h_{db})\cdot p_{escalate}
```

Total cloud calls/month (hard route + overflow + fill calls):

```math
N_{cloud\_calls}=S\cdot\left[r_h+(1-r_h)\left(r_{over}+(1-r_{over})(1-h_{db})p_{escalate}\right)\right]
```

Web-aware monthly cloud cost:

```math
C_{month}^{web}=S\cdot\left[r_hc_h+(1-r_h)\left(r_{over}c_e+(1-r_{over})(1-h_{db})p_{escalate}c_{fill}\right)\right]
```

Single-call escalation benefit versus per-field escalation:

```math
N_{fill}^{legacy}\approx S\cdot(1-r_h)\cdot(1-r_{over})\cdot(1-h_{db})\cdot p_{escalate}\cdot m_{miss}
```

```math
\text{Call reduction factor}=\frac{N_{fill}}{N_{fill}^{legacy}}=\frac{1}{m_{miss}}
```

Local batch CLI call budget (operational tracking):

```math
N_{cli}\approx S\cdot(1-r_h)\cdot(1-r_{over})\cdot(1-h_{db})\cdot k_{local}
```

where $k_{local}$ is bounded by local batch retry/step caps.

## 7) Baseline Constants for This Project

- Baseline $S=40$: $\lambda_{peak}=44.75$ tokens/sec
- Stress $S=72$: $\lambda_{peak}=80.56$ tokens/sec
- Conservative planning priors for escalation probability (use until telemetry is available):

| Architecture | $p_{escalate}$ prior |
| ------------ | -------------------: |
| `$599`       |               `0.60` |
| `$2,500`     |               `0.45` |
| `$5,000`     |               `0.35` |
| `$7,500`     |               `0.30` |

## 8) Uncertainty Model (Optional)

If monthly source count is random with $S \sim \text{Poisson}(\mu_S)$ and per-source expected cloud cost is $k$:

```math
E[C] = \mu_S k, \quad \text{Var}(C) = \mu_S k^2
```

Approximate 95% interval:

```math
C \approx E[C] \pm 1.96\sqrt{\mu_S}k
```

## 9) Token-Volume Sensitivity

If observed tokens/source differs from planning assumptions, define:

```math
\alpha = \frac{T^{obs}_{in}+T^{obs}_{out}}{T_{in}+T_{out}}
```

Then cloud cost scales approximately linearly:

```math
C^{obs}_{month} \approx \alpha \cdot C_{month}
```

## 10) SLO-Oriented Latency Mode (Industry Add)

To align with SLO practice, track at least p95/p99 latency in addition to mean throughput.

For directional queue analysis in stable regime ($\lambda < \mu_{eff}$), M/M/1 gives:

```math
W_q = \frac{\rho}{\mu_{eff} - \lambda}, \quad R = \frac{1}{\mu_{eff}} + W_q
```

where:

- $W_q$: expected queue wait per token
- $R$: expected response time per token

Planning target for burst tolerance:

```math
\rho_{target} \le 0.70
```

This is stricter than simple stability and helps protect tail latency.

## 11) Quality-Adjusted Unit Economics Mode (Industry Add)

Cost/token is not sufficient if quality differs across routes.

Define accepted knowledge items/month:

```math
N_{accept} = S \cdot q_{accept}
```

where $q_{accept}$ is acceptance rate from rubric-based review.

Quality-adjusted cost:

```math
C_{accept} = \frac{TC_{month}}{N_{accept}}
```

Use this when comparing local-heavy vs cloud-heavy policies that may have different output quality.

## 12) Recalibration Rule

At the end of each month:

1. Replace $T_{in}, T_{out}$ with observed medians.
2. Replace $h$ with observed cache hit rate.
3. Replace $\phi$ with observed peak/average demand ratio.
4. Replace $r_h$ with observed hard-route fraction.
5. Re-estimate $\mu_{compute}$ and $\mu_{memory}$ from measured tokens/sec under production concurrency.
6. Re-estimate memory pressure multiplier $f_{fit}$ from observed OOM, eviction, and KV pressure behavior.
7. Replace $h_{db}$ with observed DB complete-hit rate.
8. Replace $p_{escalate}$ with observed escalation probability at configured $\tau_{local}$ for batched local resolution.
9. Replace $c_{fill}$ with observed mean fill-call cost.
10. Replace $m_{miss}$ with observed average missing-field count.
11. Recompute $C_{month}$, $C_{month}^{web}$, $\rho$, and call-volume metrics.

## 13) Snapshot Data Entry Schema (for Real Numbers)

Use this schema for every test deployment window so model values can be replaced with measured values.

### 13.1 Required Snapshot Fields

| Field                    | Unit       | Purpose                                                     |
| ------------------------ | ---------- | ----------------------------------------------------------- |
| `snapshot_id`            | text       | Unique deployment run identifier.                           |
| `architecture_option`    | text       | One of: `$599`, `$2,500`, `$5,000`, `$7,500`.               |
| `git_commit`             | text       | Code revision used for this snapshot run.                   |
| `window_start_utc`       | datetime   | Snapshot start time.                                        |
| `window_end_utc`         | datetime   | Snapshot end time.                                          |
| `sources_completed`      | count      | Throughput in completed knowledge-source jobs.              |
| `tokens_in`              | tokens     | Actual input token volume.                                  |
| `tokens_out`             | tokens     | Actual output token volume.                                 |
| `lambda_peak_obs`        | tokens/sec | Observed peak arrival rate during the window.               |
| `mu_compute_obs`         | tokens/sec | Compute-limited service rate from load test.                |
| `mu_memory_obs`          | tokens/sec | Memory-bandwidth-limited service rate from load test.       |
| `f_fit_obs`              | 0..1       | Memory pressure multiplier from real run behavior.          |
| `memory_peak_gb`         | GB         | Peak memory observed during run.                            |
| `cache_hit_rate_obs`     | 0..1       | Observed prompt/cache hit rate.                             |
| `h_db_obs`              | 0..1       | Observed DB complete-hit rate.                              |
| `local_time_budget_sec`  | sec        | Configured local resolve time budget before batch escalation. |
| `p_escalate_obs`         | 0..1       | Observed escalation probability after local batch budget.   |
| `c_fill_obs`             | USD/call   | Observed average cost for one batch fill-call escalation.   |
| `cloud_fill_calls_obs`   | count      | Number of cloud fill calls (batch escalations) in window.  |
| `avg_missing_fields_obs` | count      | Average missing fields when escalation occurs.              |
| `local_cli_calls_obs`    | count      | Local CLI call count used for batched resolution attempts.  |
| `batch_count_obs`        | count      | Number of unresolved batches processed in the window.       |
| `batch_size_avg_obs`     | count      | Average unresolved resources per processed batch.           |
| `batch_cloud_escalation_count_obs` | count | Number of batches escalated to cloud fill.              |
| `r_h_obs`                | 0..1       | Observed hard-route fraction to high-reasoning cloud route. |
| `cloud_spend_usd`        | USD        | Cloud spend for snapshot window.                            |
| `q_accept_obs`           | 0..1       | Accepted-item rate from rubric review.                      |
| `accepted_items`         | count      | Accepted knowledge items in snapshot window.                |

### 13.2 Derived Snapshot Metrics

```math
T_{total,obs} = tokens_{in} + tokens_{out}
```

```math
\mu_{eff,obs} = f_{fit,obs} \cdot \min(\mu_{compute,obs}, \mu_{memory,obs})
```

```math
\rho_{obs} = \frac{\lambda_{peak,obs}}{\mu_{eff,obs}}
```

```math
r_{over,obs} = \max\left(0, \frac{\lambda_{peak,obs} - \mu_{eff,obs}}{\lambda_{peak,obs}}\right)
```

```math
N_{fill,obs}=sources_{completed}\cdot(1-r_{h,obs})\cdot(1-r_{over,obs})\cdot(1-h_{db,obs})\cdot p_{escalate,obs}
```

```math
N_{cloud\_calls,obs}=sources_{completed}\cdot\left[r_{h,obs}+(1-r_{h,obs})\left(r_{over,obs}+(1-r_{over,obs})(1-h_{db,obs})p_{escalate,obs}\right)\right]
```

```math
C_{month,obs}^{web}=S_{obs}\cdot\left[r_{h,obs}c_h+(1-r_{h,obs})\left(r_{over,obs}c_e+(1-r_{over,obs})(1-h_{db,obs})p_{escalate,obs}c_{fill,obs}\right)\right]
```

```math
c_{cloud/source,obs} = \frac{cloud\_spend\_usd}{sources_{completed}}
```

```math
C_{accept,obs} = \frac{TC_{month,obs}}{accepted\_items}
```

### 13.3 Snapshot Row Template

| snapshot_id | architecture_option | git_commit | window_start_utc | window_end_utc | sources_completed | tokens_in | tokens_out | lambda_peak_obs | mu_compute_obs | mu_memory_obs | f_fit_obs | mu_eff_obs | rho_obs | r_over_obs | memory_peak_gb | cache_hit_rate_obs | h_db_obs | local_time_budget_sec | p_escalate_obs | c_fill_obs | cloud_fill_calls_obs | avg_missing_fields_obs | local_cli_calls_obs | batch_count_obs | batch_size_avg_obs | batch_cloud_escalation_count_obs | r_h_obs | cloud_spend_usd | q_accept_obs | accepted_items | notes |
| ----------- | ------------------- | ---------- | ---------------- | -------------- | ----------------- | --------- | ---------- | --------------- | -------------- | ------------- | --------- | ---------- | ------- | ---------- | -------------- | ------------------ | --------------- | --------------------- | -------------- | ---------- | -------------------- | ---------------------- | ------------------- | --------------- | ------------------ | -------------------------------- | ------- | --------------- | ------------ | -------------- | ----- |
| snap-001    |                     |            |                  |                |                   |           |            |                 |                |               |           |            |         |            |                |                    |                 |                       |                |            |                      |                        |                     |                 |                    |                                  |         |                 |              |                |       |

## 14) Reference Baselines Used

- FinOps Unit Economics: [https://www.finops.org/framework/capabilities/unit-economics/](https://www.finops.org/framework/capabilities/unit-economics/)
- AWS Well-Architected Cost Optimization: [https://docs.aws.amazon.com/wellarchitected/latest/framework/a-cost-optimization.html](https://docs.aws.amazon.com/wellarchitected/latest/framework/a-cost-optimization.html)
- MLPerf Inference scenarios and metrics: [https://mlcommons.org/benchmarks/inference-datacenter/](https://mlcommons.org/benchmarks/inference-datacenter/)
- NVIDIA Triton optimization (throughput/latency/concurrency): [https://docs.nvidia.com/deeplearning/triton-inference-server/archives/triton-inference-server-2610/user-guide/docs/optimization.html](https://docs.nvidia.com/deeplearning/triton-inference-server/archives/triton-inference-server-2610/user-guide/docs/optimization.html)
- TensorRT-LLM memory decomposition and KV behavior: [https://nvidia.github.io/TensorRT-LLM/reference/memory.html](https://nvidia.github.io/TensorRT-LLM/reference/memory.html)
- Google SRE SLO guidance (percentiles/error budgets): [https://sre.google/sre-book/service-level-objectives/](https://sre.google/sre-book/service-level-objectives/)
- Roofline model background: [https://cacm.acm.org/research/roofline-an-insightful-visual-performance-model-for-multicore-architectures/](https://cacm.acm.org/research/roofline-an-insightful-visual-performance-model-for-multicore-architectures/)
