---
title: "We were hashing every byte twice: making 500 GiB volume imports fast"
date: 2026-09-19
slug: hashing-every-byte-twice
images: [images/og/hashing-every-byte-twice.png]
tags: [storage, kubernetes, python, performance, postmortem]
description: "A sync pod that downloads a model into a volume was spending hours in the step after the download. Four root causes, one small fix, and how to prove it without a 500 GiB test."
---

On the platform I work on, a *workspace volume* is a Kubernetes PVC that users fill from Hugging Face or an S3 bucket and then mount into training or inference workloads. The import runs as a one-off pod: download, then record metadata (a file tree with per-file hashes, and one digest for the whole volume) back to the control plane.

Customers importing 500 GiB+ models started reporting that the download finished in a reasonable time and then the pod sat for **hours** in the metadata step. This is the story of why, and what fixed it.

## What the pod did

{{< mermaid >}}
flowchart LR
  A[download<br/>HF / S3] --> B[file tree<br/>hash every file]
  B --> C[volume checksum<br/>hash every file again]
  C --> D[model metadata]
  B -. retry wraps hash + POST .-> B
  C -. retry wraps hash + POST .-> C
{{< /mermaid >}}

Two shared library functions did the metadata work. `store_file_tree` walked the volume, hashed each file with BLAKE3 and POSTed batches to the API. `store_checksum` walked the volume again, hashed each file again, folded the results into a directory digest and POSTed that. Each was wrapped in a retry decorator with five attempts and exponential backoff.

BLAKE3 is fast. On my laptop it does about 11 GiB/s with mmap and threads, 1.5 GiB/s single-threaded. So "hashing is slow" was not a satisfying explanation. Reading the code and running the pod under a CPU limit turned up four separate problems.

## Root cause 1: every byte hashed twice

The file tree and the volume checksum were independent functions with independent walks. For a 500 GiB volume that is a terabyte of reads through a FUSE or NFS mount, and disk, not CPU, was the bottleneck. The two results are derived from the same per-file digests; there was never a reason to compute them separately.

## Root cause 2: the retry re-hashed the whole volume

The retry decorator wrapped the entire function: walk, hash, POST. A transient 502 from the API on the final POST, which happens, meant attempt two started from the first file again. Worst case: five full hashes of the volume for one flaky request.

## Root cause 3: thread count sized to the host, not the pod

`blake3(max_threads=AUTO)` looks at `os.cpu_count()`, which inside a container returns the node's cores, not the cgroup quota. On a 64-core node with a 2-CPU limit the hasher spawned 64 threads and spent most of its time being throttled by CFS. Throttling shows up as wall time with no obvious cause; CPU usage looks "busy".

## Root cause 4: the checksum was not even stable

`huggingface_hub.snapshot_download(local_dir=...)` leaves a `.cache/huggingface/` directory inside the target with a `.metadata` file per download, containing timestamps. Those files were inside the volume, so they were hashed. Two imports of the same model produced two different volume checksums. Nobody had noticed because nobody had compared them.

## The fix

{{< figure src="/images/hashing-every-byte-twice.gif" alt="Before: download, hash all files for the tree, hash all files again for the checksum, 1000 GiB read. After: download, hash once, then tree and checksum reuse the digests, 500 GiB read." caption="Before and after: the same digest, half the reads." >}}

Small, once the causes were clear:

1. Hash once. A new `compute_file_hashes(mount_path)` returns `{path: digest}` and both `store_file_tree` and `store_checksum` accept that map instead of hashing themselves.
2. Retry only the cheap part. The hash step has its own retry; the two POST steps retry a directory walk and an HTTP call, never a hash.
3. Size threads from the cgroup quota (`/sys/fs/cgroup/cpu.max`), falling back to `os.cpu_count()` only outside a container.
4. Delete `<target>/.cache/huggingface` after the download, before anything is hashed.

The shape of the change in the entrypoint:

```python
downloaded = download(source, target)
remove_hf_cache_dir(target)

hashes = compute_hashes(target)       # retried on its own, runs once on success
save_file_tree(hashes)                 # retry = walk + POST
save_checksum(hashes)                  # retry = fold + POST
save_model_metadata(target)
```

The digest formula stayed byte-identical, so existing volumes did not change checksum after the upgrade. That was a hard constraint: a checksum that changes on a library bump is a checksum nobody trusts.

## What about provider hashes?

An obvious question: Hugging Face and S3 already publish hashes, why compute anything? Because they are the wrong hashes. Hugging Face gives SHA-256 for LFS files and a git blob SHA-1 for small ones. S3 ETags are MD5 for single-part uploads and an MD5-of-MD5s with a part count for multipart, which cannot be reproduced without knowing the uploader's part size. Our contract with the rest of the platform is a BLAKE3 digest, and we want one algorithm for every source. Provider hashes could be used to *verify* the download, which is a separate improvement.

## Proving it without a 500 GiB test

The tempting test is "import a big model, time it". It takes hours, depends on the network, and one number tells you little. The scale-independent version:

- Run the pod on a kind cluster with `cpu: 2` against a small volume.
- Read `/proc/1/io` inside the pod after it finishes. `read_bytes` for the old code is about 2× the volume size; for the new code about 1×.
- Inject a failure into the checksum POST and count `compute_file_hashes` calls: exactly one.

Those three checks hold at 1 GiB and at 500 GiB, and they run in CI.

## What could be better

- The two libraries still duplicate the file schema by hand (`relative_path`, `size`, `checksum`) with the API's request model. Adding a field means touching three repos. A shared contract package would remove that.
- Downloads are sequential and the download retry is also whole-function: a failure at file 9,999 of 10,000 re-downloads everything unless a no-overwrite strategy is set. Per-file resume is the next real win.
- Hashing happens after the download instead of streaming during it. For volumes that are network-bound, hashing while downloading would hide the metadata step entirely.
- Provider hashes should be used as a download verification layer, even though they cannot replace the volume digest.

## Takeaways

- When a "slow" step uses a fast primitive, count the bytes, not the seconds. `read_bytes ≈ 2× volume` said everything.
- Put retries around the cheapest unit that can fail, not around the function that is convenient to decorate.
- Inside containers, thread pools sized from `cpu_count()` are a throttling bug waiting to happen.
- Anything that lands inside a volume becomes part of its identity. Clean up tool caches before you hash.
