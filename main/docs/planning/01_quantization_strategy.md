# Quantization Strategy (Summer 2026 MVP)

## Purpose

The budget docs already assume quantized local models. This document makes that assumption operational by defining which quantization profiles are allowed, how they are compared, and when they may be promoted into pilot use.

## Scope

This strategy covers local weight quantization for Ollama-served models. It does not yet standardize KV-cache precision, speculative decoding, distillation, or fine-tuning.

## Serving Assumption

- local serving remains standardized through `ollama-server`
- when explicit artifact control is needed, use locally created or imported GGUF-backed Ollama models
- quantization is a first-class experiment variable and must never be treated as invisible implementation detail
- the explicit quantization levels used in this repo align with the Ollama-documented options `q8_0`, `q4_K_S`, and `q4_K_M`

## Quantization Roles

Use three named quantization roles across the project:

| Role | Quantization | Use | Promotion Rule |
| --- | --- | --- | --- |
| Reference production | `q4_K_M` | default pilot setting for record-producing runs | remains default unless a different profile is shown to be clearly better in both fit and reviewed quality |
| Emergency fit | `q4_K_S` | use only when `q4_K_M` does not fit the memory and concurrency target | allowed only after its own evaluation window passes the quality gate |
| Evaluation control | `q8_0` | spot-check higher-fidelity local behavior on a representative subset | use for comparison windows, not as the default summer serving profile |

## Budget-Tier Mapping

| Budget Tier | Model | Reference Quantization | Allowed Alternate | Operational Note |
| --- | --- | --- | --- | --- |
| `$599` | `qwen2.5:7b` | `q4_K_M` | `q4_K_S` | keep context and concurrency tightly capped |
| `$2,500` | `qwen2.5:14b` | `q4_K_M` | `q8_0` for spot evaluation, `q4_K_S` only if fit fails | this is the summer reference tier |
| `$5,000` | `qwen2.5:32b` | `q4_K_M` | `q8_0` for targeted evaluation | preferred upgrade path after the pilot |
| `$7,500` | `qwen2.5:72b` | `q4_K_M` | `q8_0` for targeted evaluation only | not required for the reference summer pilot |

The exact context and concurrency envelope still has to be measured per machine. Quantization policy does not remove the need for telemetry.

## Operational Rules

- every snapshot window must log `model_profile`, `quantization_profile`, `context_tokens`, `concurrency`, `tau_local`, and `q_min`
- do not compare or aggregate unlike quantization windows under the same `snapshot_id`
- any quantization change starts a new evaluation window and a new rolling baseline
- reduce context or concurrency before dropping from `q4_K_M` to `q4_K_S` when possible
- do not use `q4_K_S` for professor-facing benchmark claims unless it has its own passing evaluation window

## Promotion Criteria

Treat a quantization profile as acceptable for pilot use only if all are true:

1. it fits the target memory envelope with the planned context and concurrency
2. it meets the startup quality gate from [`../confidence/00_output_confidence_interval_method.md`](../confidence/00_output_confidence_interval_method.md)
3. it does not reduce observed acceptance by more than `5` percentage points relative to the comparison control window
4. it does not materially increase unresolved-field carryover or overwrite corrections

## Minimal Validation Procedure

Before promoting a new quantization profile:

1. run the candidate profile and the comparison control on the same representative source bundle
2. review at least `200` field-level units for the candidate window
3. capture `mu_eff_obs`, peak memory, cloud escalations, and reviewed acceptance
4. record the decision in a `TB-05` Routing Decision Record

## Notes for Implementation

- the current summer reference is `q4_K_M`, not because it is universally best, but because it is a reasonable default that balances fit and quality for a local-first pilot
- `q8_0` is valuable mainly as a local comparison anchor
- quantization must be reported anywhere hallucination or throughput claims are reported

## Relevant References

- Ollama import and custom model creation: [https://docs.ollama.com/import](https://docs.ollama.com/import)
- `llama.cpp` GGUF and quantization overview: [https://github.com/ggml-org/llama.cpp](https://github.com/ggml-org/llama.cpp)
