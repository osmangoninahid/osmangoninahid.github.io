---
title: "The four GPU numbers that decide if your model will serve"
date: 2026-09-23
slug: gpu-numbers-for-llm-inference
tags: [gpu, inference, llm, kubernetes, capacity-planning]
description: "Memory, bandwidth, precision, scaling. How to check each one with a formula, a metric, or a test, and how a GPU platform automates the checks."
draft: true
---

A post on LinkedIn last week put it well: for LLM inference, stop asking "how many TFLOPS?" and ask four other questions first. Memory, bandwidth, precision, scaling. I agree with the list. I work on a platform that schedules GPUs for other people's models, so we had to turn those four questions into code. This is the list with the numbers, the formulas, and what we learned automating it.

## The checklist

{{< mermaid >}}
flowchart LR
  A["1. Memory<br/>does it fit?"] --> B["2. Bandwidth<br/>how fast per token?"]
  B --> C["3. Precision<br/>which math is native?"]
  C --> D["4. Scaling<br/>one GPU or many?"]
  A -. no .-> C
{{< /mermaid >}}

Order matters. If the model does not fit, bandwidth is irrelevant. If it almost fits, precision (step 3) is how you make it fit, so the arrow loops back.

## 1. Memory: does it fit?

Three things need room: weights, KV cache, and runtime overhead.

**Weights**: parameters × bytes per parameter.

| Precision | Bytes/param | 8B model | 70B model | 405B model |
| --- | --- | --- | --- | --- |
| FP16 / BF16 | 2 | 16 GB | 140 GB | 810 GB |
| FP8 / INT8 | 1 | 8 GB | 70 GB | 405 GB |
| INT4 / NVFP4 | 0.5 | 4 GB | 35 GB | 203 GB |

**KV cache** is the part people forget, and it grows with traffic:

```latex
KV\_bytes\_per\_token = 2 \times layers \times kv\_heads \times head\_dim \times bytes
```

Llama-3-70B in BF16: 80 layers, 8 KV heads (GQA), head dim 128, 2 bytes → **320 KB per token**. At 8k context and 16 concurrent requests that is ≈ **42 GB**, on top of 140 GB of weights. Without GQA it would be four times that. Models with MLA (the DeepSeek family) cache a compressed latent instead of full keys and values, so the formula becomes `layers × (latent_rank + rope_dim) × bytes`, an order of magnitude smaller. The architecture decides how much traffic a card can hold, not just the parameter count.

**Overhead**: CUDA context, activations, the serving engine's workspace. 1–3 GB per GPU, more with CUDA graphs.

### Two ways to compute it, and we built both

The **coarse planner** is for the sales-call stage, before anyone has downloaded a checkpoint. It uses `weights × 1.3` for KV cache and runtime, treats only `0.9 × VRAM` as usable, and does `ceil(footprint / usable)` to get GPUs per replica. The comment in that file says it plainly: 1.3 is a rule of thumb, to be replaced by measured footprints. It is right often enough to answer "how many H200s for this model at this concurrency".

The **precise inspector** runs when a model lands on the platform. It reads only the safetensors *headers* (never the weights), sums tensor bytes by dtype, reads the quantization config, and emits: parameter count, effective bits per parameter, minimum VRAM, KV bytes per token, worst-case KV at the model's max context, and which tensor-parallel sizes are valid. A 70 B model with 30 shards is inspected in under a second. Those numbers then drive scheduling and the deployment form, so a user cannot pick a card the model will not fit on.

**How to check by hand**: the serving engine tells you. vLLM logs the memory it reserves for weights and for KV blocks at startup. If the log says few or zero KV blocks, the model loaded but cannot serve anyone.

## 2. Bandwidth: how fast can a token move?

Decoding one token means reading every weight once, per request. For small batches inference is memory-bound, and the ceiling is:

```latex
tokens/s \approx \frac{bandwidth}{bytes\ read\ per\ token}
```

A 70B model in FP8 reads 70 GB per token. On an H100 SXM (3.35 TB/s) that caps single-stream decode near 48 tokens/s; on an H200 (4.8 TB/s) near 68. Batching reuses the weights across requests, which is how aggregate throughput reaches thousands of tokens per second, but batching grows the KV cache (step 1). The two numbers trade against each other.

Bandwidth by card, and this is where spec sheets bite:

| GPU | Memory | Bandwidth |
| --- | --- | --- |
| A100 40 GB PCIe | 40 GB HBM2 | 1.55 TB/s |
| A100 80 GB SXM | 80 GB HBM2e | 2.0 TB/s |
| H100 PCIe | 80 GB HBM2e | 2.0 TB/s |
| H100 SXM | 80 GB HBM3 | 3.35 TB/s |
| H200 | 141 GB HBM3e | 4.8 TB/s |
| MI300X | 192 GB HBM3 | 5.3 TB/s |

Same name, different bus, 40% different bandwidth. Always check the variant.

**How to check it**: DCGM exports `DCGM_FI_PROF_DRAM_ACTIVE` (memory bandwidth in use) next to `DCGM_FI_PROF_GR_ENGINE_ACTIVE` (compute in use). A serving pod at DRAM 95% and graphics engine 30% is bandwidth-bound; more TFLOPS will not help, a faster bus or a smaller dtype will. Two operational notes from running this at scale: the DCGM exporter caps out at 128 metrics per GPU, so you have to choose which counters to ship (we cut our list from ~150 to ~40), and a brand-new card model may not report the profiling counters until you add them by hand.

## 3. Precision: what does the hardware run natively?

Lower precision halves the bytes, so steps 1 and 2 both improve, and it uses tensor cores built for that dtype.

| GPU | FP16/BF16 | FP8 | INT8 | FP4 |
| --- | --- | --- | --- | --- |
| A100 | yes | no | yes | no |
| H100 / H200 | yes | yes | yes | no |
| B200 / B300 | yes | yes | yes | yes |
| MI300X | yes | yes | yes | no |

An A100 can run an FP8 checkpoint only by dequantising, which throws away the win. "Does it fit in FP8?" is only a real question on Hopper or newer.

Our planner encodes a small policy here that took a while to get right: pick the *smallest verified quantization the GPU supports*, except prefer a 4-bit microscaling format (NVFP4, MXFP4) over plain INT4 when both exist, because per-block scaling gives noticeably better quality at the same bit width. And when a GPU's supported precisions are unknown, assume BF16 only and say so in the output. Guessing a generous ladder is how you promise an FP8 deployment on a card that cannot do it.

**How to check it**: run the model's benchmark at each dtype you can, and compare tokens/s against an eval score. Quantisation is free speed until it is not; the eval tells you where "not" is.

## 4. Scaling: one GPU or many?

When the model does not fit on one card you split it. Tensor parallel splits each layer across GPUs and talks after every layer, so it needs NVLink and stays inside one node. Pipeline parallel splits by layers, talks less, and can cross nodes. PCIe-only boxes make tensor parallel slow; that is the hidden cost of the cheaper card.

Not every TP size is valid. Attention heads have to shard cleanly: `tp` must divide the attention heads, and the KV heads must divide `tp` or be divisible by it. The inspector computes the valid list per model, and the serving template takes the TP size from the platform rather than from the user, because "8 GPUs" for a model with 40 attention heads is a crash at startup, not a warning.

For more traffic rather than a bigger model, replicas beat parallelism: run N copies behind a gateway.

**MIG** goes the other way: an A100 or H100/H200 splits into up to seven isolated slices, so one card serves several small models with hard isolation. We made slices a first-class resource users can pick next to whole GPUs. One bug from doing that is worth sharing: every slice was being handed the whole card's share of the node's CPU, so a one-seventh slice looked as generous as the full card. The fix reads the fraction from the slice name and scales the allowance, 2 cores for the seventh, 14 for the whole card. Bandwidth scales the same way: a slice gets its fraction, so step 2 applies per slice.

**How to check it**: Kubernetes tells you GPU count; only `nvidia-smi topo -m` tells you which pairs are NVLinked.

## Putting it together

Before any deployment:

1. Weights + KV cache at target concurrency + overhead → fits on which card, at which precision?
2. Bytes per token ÷ bandwidth → is single-stream latency acceptable?
3. Is that precision native on that card?
4. More than one card: is the TP size valid, is there NVLink, would replicas do instead?

TFLOPS enter only in step 3, and mostly for prefill-heavy work (long prompts, RAG), where compute finally matters.

## What could be better

- The coarse 1.3× headroom should become measured KV footprints per model family; the inspector already produces the numbers, the planner does not read them yet.
- The KV formula assumes dense attention. Paged attention and prefix caching change the effective number; measure with your real prompt distribution.
- The bandwidth ceiling ignores speculative decoding, which reads the big model less often.
- Cost is missing. One card that fits the model is often cheaper per token than two cheaper cards with tensor parallel, once you count the NVLink tax.

## Takeaways

- Fit first: weights + KV cache + overhead, at real concurrency, and the KV term depends on the architecture (GQA, MLA), not just the parameter count.
- Decode is memory-bound; bandwidth sets the single-stream ceiling, batching sets the aggregate.
- Precision buys back memory and bandwidth only where the hardware runs it natively; when in doubt, assume BF16 and say so.
- Tensor parallel has a valid-size list; compute it, do not let users guess.
- Two DCGM metrics, DRAM active and graphics engine active, tell you which of the four you are short of.
