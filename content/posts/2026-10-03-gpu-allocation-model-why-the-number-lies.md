---
title: "The GPU allocation model: why the number lies"
date: 2026-10-03
slug: gpu-allocation-model-why-the-number-lies
tags: ["gpu", "kubernetes", "scheduling", "platform-engineering", "multi-tenancy", "mig"]
description: "A fixed GPU fleet, many teams, one dashboard number. On a shared cluster 'available GPUs' is assembled from four sources, and every capacity ticket I handled this year was two of them disagreeing. The model, the six lies, and the reconciler that fixed them."
images: ["images/og/gpu-allocation-lies.png"]
---

A fixed fleet, many teams, one dashboard number: *available GPUs*. On a shared cluster that number is not read — it is assembled from four sources. Every capacity ticket I handled this year was two of them disagreeing.

{{< figure src="/images/gpu-allocation-lies.gif" alt="Four sources (Kubernetes nodes, platform records, scheduler queues, batch scheduler) feed one 'available GPUs' number; five lies cycle through — a node that left, a GPU idle on one side and busy on the other, a frozen usage sync, MIG slices counted as devices, a Ready node with zero GPUs — then the fix" >}}

## The model

{{< mermaid >}}
graph TB
    C["cluster<br/>nodes · GPU types · MIG slices"] -->|"nodes → tenant<br/>target: k8s | slurm"| T["tenant"]
    T -->|"GPUs by type → workspace"| W["workspace<br/>queue: capability · deserved · borrow"]
    W --> P["pods"]
{{< /mermaid >}}

Three levels, three owners. A node serves one *target* — Kubernetes or the batch scheduler — never both. A workspace is a scheduler queue with a hard ceiling, a deserved share, and a borrowing flag.

## Four sources, one number

{{< mermaid >}}
graph LR
    K["kubernetes nodes<br/>capacity"] --> N["available GPUs"]
    R["platform records<br/>allocation"] --> N
    Q["scheduler queues<br/>usage"] --> N
    B["batch scheduler<br/>slurm target"] --> N
    style N fill:#58a6ff22,stroke:#58a6ff
{{< /mermaid >}}

## Six lies

{{< mermaid >}}
graph LR
    L1["3 of 2<br/>node left the cluster,<br/>record stayed"] --> F1["count from<br/>healthy nodes only"]
    L2["idle to slurm,<br/>busy on k8s"] --> F2["filter by target;<br/>busy = any workload"]
    L3["usage frozen<br/>'13Gi' is not an int"] --> F3["parse quantities;<br/>skipped sync = alert"]
    L4["4 of 2<br/>MIG slices counted<br/>as devices"] --> F4["populate from<br/>physical GPUs"]
    L5["Ready, 0 GPUs,<br/>reported healthy"] --> F5["health includes<br/>capacity"]
    L6["H100s listed for<br/>a tenant with T4s"] --> F6["render from allocation,<br/>not scheduler vocabulary"]
    style L1 fill:#f8514922,stroke:#f85149
    style L2 fill:#f8514922,stroke:#f85149
    style L3 fill:#f8514922,stroke:#f85149
    style L4 fill:#f8514922,stroke:#f85149
    style L5 fill:#f8514922,stroke:#f85149
    style L6 fill:#f8514922,stroke:#f85149
    style F1 fill:#3fb95022,stroke:#3fb950
    style F2 fill:#3fb95022,stroke:#3fb950
    style F3 fill:#3fb95022,stroke:#3fb950
    style F4 fill:#3fb95022,stroke:#3fb950
    style F5 fill:#3fb95022,stroke:#3fb950
    style F6 fill:#3fb95022,stroke:#3fb950
{{< /mermaid >}}

Two more that did not fit a box: reclaim between workspaces only works if `deserved` lists *every* resource — CPU, memory, ephemeral storage — not just GPU. And "0/18 nodes: insufficient mig-4g.20gb" lived only in Kubernetes events, which are ephemeral, so the UI showed nothing. Persist the scheduling reason.

## MIG as desired state

Slicing a GPU is cordon → drain → profile → device-plugin restart → uncordon. The imperative version produced stuck nodes. Now it is declarative:

{{< mermaid >}}
graph LR
    D["platform writes<br/>desired: cordon, MIG profile"] --> R["node reconciler"]
    R --> K["kubernetes<br/>cordon · ConfigMap · roll MIG manager"]
    K --> R
    R --> O["observed status<br/>written back"]
    H["health monitor"] -.->|"owns only<br/>its fields"| O
    style R fill:#3fb95022,stroke:#3fb950
{{< /mermaid >}}

The bug that taught field ownership: a health monitor replaced the whole node record on every write and clobbered the desired fields the reconciler had just set. Every writer owns named fields; nobody owns the document.

## Rules

1. One source per number — capacity from Kubernetes, allocation from the platform, usage from queues.
2. Health includes capacity.
3. A skipped sync is an alert, not a warning line.
4. Persist scheduling reasons.
5. Model *target* as a node property from day one.
6. Check for running workloads before deallocating, and say so in the dialog.

## What I would do differently

Design the accounting before the UI — half of these were "UI bugs" whose root cause was a number computed in the wrong place. Make MIG declarative from the start. Put cost next to allocation; once people see it, over-allocation fixes itself.
