---
title: "The four GPU numbers that decide if your model will serve"
date: 2026-09-23
slug: gpu-numbers-for-llm-inference
tags: [gpu, inference, llm, kubernetes, capacity-planning]
description: "Memory, bandwidth, precision, scaling. How to check each one with a formula, a metric, or a test before you buy or schedule a GPU for LLM inference."
draft: true
---

A post on LinkedIn last week put it well: for LLM inference, stop asking "how many TFLOPS?" and ask four other questions first. Memory, bandwidth, precision, scaling. I agree with the list. I run a platform that schedules these GPUs for other people's models, so this is the version with the numbers, the formulas, and the checks I use.

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

**Weights** are easy: parameters × bytes per parameter.

| Precision | Bytes/param | 8B model | 70B model | 405B model |
| --- | --- | --- | --- | --- |
| FP16 / BF16 | 2 | 16 GB | 140 GB | 810 GB |
| FP8 / INT8 | 1 | 8 GB | 70 GB | 405 GB |
| INT4 | 0.5 | 4 GB | 35 GB | 203 GB |

**KV cache** is the part people forget, and it is what grows with traffic:

```latex
KV\_bytes = 2 \times layers \times kv\_heads \times head\_dim \times bytes \times seq\_len \times batch
```

Llama-3-70B in BF16: 80 layers, 8 KV heads (GQA), head dim 128, 2 bytes → 2 × 80 × 8 × 128 × 2 = **320 KB per token**. At 8k context and 16 concurrent requests that is 320 KB × 8192 × 16 ≈ **42 GB**, on top of 140 GB of weights. Without GQA (32 KV heads) it would be 1.3 MB per token and 168 GB of cache. This is why GQA models serve so much better.

**Overhead**: CUDA context, activations, the serving engine's workspace. Budget 1–3 GB per GPU, more if the engine uses CUDA graphs.

So a 70B model in BF16 with real traffic wants about 185 GB. It does not fit on one 80 GB or one 141 GB GPU. It fits on two H100s with tensor parallel (step 4), or on one H200 in FP8 (step 3). That is the whole planning conversation, and it comes from the fit calculation, not from TFLOPS.

**How I check it**: the serving engine tells you. vLLM logs the memory it reserves for weights and for KV blocks at startup, and `gpu_memory_utilization` is the knob. If the log says "0 KV blocks", the model fits but cannot serve anyone. On our platform we run this on a cluster with a fake GPU operator before touching real hardware, so scheduling logic gets tested for free; the fit itself still needs real memory.

## 2. Bandwidth: how fast can a token move?

Decoding one token means reading every weight once. Per token, per request. So for small batches inference is memory-bound, and the ceiling is:

```latex
tokens/s \approx \frac{bandwidth}{bytes\ read\ per\ token}
```

A 70B model in FP8 reads 70 GB per token. On an H100 SXM (3.35 TB/s) that caps single-stream decode at about 48 tokens/s. On an H200 (4.8 TB/s) about 68. Batching reuses the weights across requests, which is how you get to thousands of tokens per second in aggregate, but it also grows the KV cache (step 1).

Bandwidth by card, and this is where spec sheets bite:

| GPU | Memory | Bandwidth |
| --- | --- | --- |
| A100 40 GB PCIe | 40 GB HBM2 | 1.55 TB/s |
| A100 80 GB SXM | 80 GB HBM2e | 2.0 TB/s |
| H100 PCIe | 80 GB HBM2e | 2.0 TB/s |
| H100 SXM | 80 GB HBM3 | 3.35 TB/s |
| H200 | 141 GB HBM3e | 4.8 TB/s |
| MI300X | 192 GB HBM3 | 5.3 TB/s |

Same name, different bus, 40% different bandwidth. Always check which variant you have.

**How I check it**: DCGM exports `DCGM_FI_PROF_DRAM_ACTIVE` (memory bandwidth utilisation) next to `DCGM_FI_PROF_GR_ENGINE_ACTIVE` (compute). A serving pod that shows DRAM active near 100% and graphics engine at 30% is bandwidth-bound: more TFLOPS will not help, a faster memory bus or a smaller dtype will. Watch these two together on a dashboard per deployment; the ratio tells you what to buy next.

## 3. Precision: what does the hardware run natively?

Lower precision does two things: halves the bytes (steps 1 and 2 both improve) and uses tensor cores built for that dtype.

| GPU | FP16/BF16 | FP8 | INT8 | FP4 |
| --- | --- | --- | --- | --- |
| A100 | yes | no | yes | no |
| H100 / H200 | yes | yes | yes | no |
| B200 | yes | yes | yes | yes |
| MI300X | yes | yes | yes | no |

An A100 can run an FP8 checkpoint only by dequantising, which throws away the win. So "does the model fit in FP8?" is only a real question on Hopper or newer. Ask the TFLOPS question last, and ask it as "TFLOPS at the precision I will actually use, dense, not the sparse marketing number".

**How I check it**: run the model's own benchmark at each dtype you can, and compare tokens/s and an eval score. Quantisation is free speed until it is not; the eval tells you where "not" is.

## 4. Scaling: one GPU or many?

When the model does not fit on one card you split it. Tensor parallel splits each layer across GPUs and talks after every layer, so it needs NVLink (900 GB/s on H100 SXM) and stays inside one node. Pipeline parallel splits by layers, talks less, and can cross nodes. PCIe-only boxes (no NVLink) make tensor parallel slow; that is the hidden cost of the cheaper card.

For serving more traffic rather than a bigger model, replicas beat parallelism: run N copies and load-balance. That needs a scheduler that packs GPUs well and a gateway in front.

**MIG** goes the other way: an A100 or H100 can be split into up to seven isolated slices. A 7B model in FP8 fits in a 20 GB slice, so one 80 GB card serves several small models with hard isolation. On our platform MIG profiles are a first-class resource users can pick; the scheduler treats a slice like a small GPU. The catch is that MIG slices have proportionally less bandwidth, so step 2 applies per slice.

**How I check it**: a fit test on the cluster with the real topology. Kubernetes tells you GPU count; only `nvidia-smi topo -m` tells you whether the pairs are NVLinked. We learned that once by getting half the expected tensor-parallel throughput on a box that had NVLink bridges on only some pairs.

## Putting it together

The question I ask before any deployment:

1. Weights + KV cache at the target concurrency + overhead → fits on which card, at which precision?
2. Bytes per token ÷ bandwidth → is single-stream latency acceptable?
3. Is that precision native on that card?
4. If it needs more than one card: NVLink or not, and would replicas do instead?

TFLOPS enter only in step 3, and only for prefill-heavy workloads (long prompts, RAG) where compute finally matters.

## What could be better

- The KV cache formula assumes dense attention. Paged attention and prefix caching in modern engines change the effective number; measure with your real prompt distribution.
- The bandwidth ceiling ignores speculative decoding, which reads the big model less often. Worth a separate post.
- I have not touched cost. A card that fits the model in one slot is often cheaper per token than two cheaper cards with tensor parallel, once you count the NVLink tax.

## Takeaways

- Fit first: weights + KV cache + overhead, at real concurrency.
- Decode is memory-bound; bandwidth sets the single-stream ceiling, batching sets the aggregate.
- Precision is how you buy back memory and bandwidth, but only where the hardware runs it natively.
- Scaling out needs NVLink for tensor parallel; replicas are simpler when the model already fits.
- Two DCGM metrics, DRAM active and graphics engine active, tell you which of the four you are short of.
