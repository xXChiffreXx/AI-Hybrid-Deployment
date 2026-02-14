# Architecture Option D: $599 Machine Budget (Education M4 Base + 10GbE Add-On)

## Goal

Create the lowest-cost institutional entry point for local knowledge-base generation while preserving the same hybrid architecture and team SSH workflow.

## Budget Assumption

This option is modeled as a `$599` budget class as requested.
If the 10GbE configuration raises actual procurement price above `$599`, keep this document as the lower-bound comparison profile.

## Target Infrastructure Profile

- 1 shared research node
- 10Gb Ethernet (configured)
- Base-class CPU profile
- 16-24 GB unified memory class
- 512 GB to 1 TB SSD
- SSH access for small team (2-4 active users)

Service layer is OS-agnostic because inference is standardized through `llama-server`.

## Logical Architecture

- `gateway` for route policy and budget controls
- `llama-server` local endpoint
- `redis` for cache and queue state
- `qdrant` for sector retrieval memory
- `kb-writer` to canonical `SQLite + Parquet`
- `obsidian-projector` for generated vault views

## Deployment Pattern

- Single-node compose stack
- Local-first for routine tasks when available
- Cloud for hard tasks and overflow during demand spikes

## Capacity Assumption

- Effective local capacity target: $\mu = 60$ tokens/sec
- Per-source token load: $T_{in}+T_{out}=725{,}000$

Theoretical local-only source throughput:

$$

\text{sources/day} = \frac{60 \cdot 86400}{725000} \approx 7.15

$$

## Cloud Cost Model (Instantiated)

Using the shared model in `00_cloud_cost_model.md`:

- Hard-task cloud fraction: $r_h = 0.25$
- Baseline peak demand: $\lambda_{peak}=145.45$ tokens/sec
- Stress peak demand: $\lambda_{peak}=261.81$ tokens/sec

Overflow fractions:

$$

r_{over,base}=\max\left(0,\frac{145.45-60}{145.45}\right)=0.5875

$$

$$

r_{over,stress}=\max\left(0,\frac{261.81-60}{261.81}\right)=0.7708

$$

Monthly cloud cost:

$$

C_{month}=S\left(r_hc_h+(1-r_h)r_{over}c_e\right)

$$

with $c_e=0.33925$, $c_h=1.69625$.

Baseline (`S=130`):

$$

C_{base}=130\left(0.25\cdot1.69625+0.75\cdot0.5875\cdot0.33925\right)=\$74.57/month

$$

Stress (`S=234`):

$$

C_{stress}=234\left(0.25\cdot1.69625+0.75\cdot0.7708\cdot0.33925\right)=\$145.12/month

$$

Approximate annual cloud range from this model: `$895 - $1,741`.

## Fit for Project

### Pros

- Lowest capital entry point.
- Same software architecture as larger options.
- Useful for initial pilot and low-concurrency operation.

### Constraints

- Highest expected queue pressure and overflow.
- Less local headroom for larger context/model experiments.
- Most sensitive to workload spikes and retry overhead.

## Recommended If

Choose this if the priority is immediate start with minimal capex and you accept heavier cloud dependence plus slower local turnaround.
