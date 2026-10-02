---
title: "Shipping one platform to many clusters: release engineering for an AI platform"
date: 2026-09-30
slug: release-engineering-ai-platform-many-clusters
tags: ["platform-engineering", "release-engineering", "kubernetes", "gitops", "helm", "sre"]
description: "How a GPU platform gets from a merge to twenty customer clusters on different versions, some air-gapped. The pipeline we built, the migration rules we learned the hard way, and what I would design differently."
images: ["images/og/release-migrations.png"]
---

A SaaS deploys one version to one place. A platform sold to enterprises deploys to every customer's cluster, each on the version they last accepted, several with no internet. Release engineering is the part of platform engineering that makes that boring. This is how ours works, and the tickets that shaped it.

{{< figure src="/images/release-migrations.gif" alt="A migration chain of preflight, database, object storage, Kubernetes entities and smoke test; before, a failing step did not stop the chain; after, it aborts and reports; three clusters on different versions walk the same steps" >}}

## The shape of the problem

- Around forty services, each its own repository and pipeline.
- Two branch kinds everywhere: `main` and `release-X.Y.Z`.
- Thirty-plus Helm charts in one monorepo, from the application down to the message broker and the GPU exporters.
- Customer clusters on 1.7, 1.10, 1.13 at the same time. Some pull from our registry; some get a mirrored registry inside their network and nothing else.

Every upgrade has to move three things that live in three places: the **database schema**, the **object-storage layout**, and **Kubernetes objects** (labels, services, cluster records). Miss one and the platform comes up "healthy" and wrong.

## What the pipeline looks like

{{< mermaid >}}
graph TB
    subgraph repos["~40 service repos"]
        S1["service<br/>.gitlab-ci.yml"] --> C["shared CI components<br/>build · scan · release-image"]
    end
    C -->|"main"| I1["image dev-N"]
    C -->|"release-X.Y.Z"| RM["release manager job<br/>auto patch bump · git tag · image X.Y.N"]
    RM --> I2["image X.Y.N"]
    I2 --> H["chart monorepo<br/>test + package per chart"]
    H --> INST["installer<br/>helmfile + env json"]
    INST --> TOOL["tooling<br/>validate · template · check · install/upgrade"]
    TOOL --> MIG["migration framework<br/>db · object storage · k8s<br/>checksums · info · validate"]
    MIG --> K1["connected cluster"]
    MIG --> K2["air-gapped cluster<br/>(phase 1: mirror registry)"]
    RD["release dashboard<br/>cuts release branches across repos,<br/>watches pipelines, verifies images"] -.-> RM
{{< /mermaid >}}

Three decisions carry most of the weight:

**1. Release branches are the unit of release, and a job owns the version.** A `release-1.10.0` branch does not get hand-tagged. A small release-manager job runs on the latest commit, bumps the patch, tags `v1.10.145`, pushes the image with the same number. `main` gets `dev-N`. Nobody types a version, so nobody types the wrong one. The catch we hit: a merge request pipeline and the branch pipeline both built an image, and the MR one overwrote the release one. Fix: MR builds and branch builds write to different names. Second catch: the image tag came from the branch *slug* (`release-1-10-0`) in one job and the branch *name* (`release-1.10.0`) in another. One line, two months of "which image is this".

**2. Charts are packaged per chart, not per repo.** The chart monorepo has a test and a package job for every chart — the application, the broker, the databases, identity, cert-manager, the metrics stack, the GPU exporters. A change to one chart runs one pair of jobs. The installer then pins chart versions in an environment file per cluster: registry, storage class, domain, component versions. That file is the cluster's identity; everything else is generated.

**3. Upgrades are a numbered chain, and the chain stops.** The migration framework applies versioned scripts in order — Python scripts with injected database, object-storage and scheduler clients, or JavaScript run straight in the database shell. `info` prints what has run; `validate` compares checksums of applied scripts against the repository; `--strict` refuses to run if they differ.

## The rules the tickets taught us

**Stop on the first exception.** During one upgrade a migration that creates coordination leases failed, and the next migrations ran anyway on a half-applied state. Found in production. The framework now aborts on any exception and reports what was applied. Obvious in hindsight; the original loop had a `try/except` that logged and continued, because "one bad script shouldn't block the rest". It should.

**An executed migration is immutable.** After a release retrospective we added a CI check that fails the build if any historical migration file changes. Checksums in the applied-migrations table make the same check at run time. Editing an old migration is the fastest way to have two clusters that ran "the same" version and differ.

**Migrations create data through the application's own code path.** A migration that introduced multi-cluster support wrote the initial cluster record by hand. Fresh installs then reported a healthy cluster with zero nodes and zero GPUs, while Kubernetes had plenty — the hand-built document was missing fields the application's reader filtered on. If the app has a function that creates the thing, the migration calls it.

**Test the upgrade path from real old versions.** The application pipeline runs the migration suite against snapshots of the last two supported releases whenever the migrations directory changes, or on demand with a commit-message flag. Most "upgrade broke X" tickets were caught here after that job existed, and before it they were not.

**The fleet upgrades itself if you let it.** A GPU operator with automatic driver upgrades enabled started upgrading drivers on production nodes, failed on several, and cordoned them. Nothing in our pipeline did that; a default did. Pin driver versions, disable auto-upgrade, and treat drivers as a release like any other.

**Upgrades change more than code.** After one upgrade a customer's model-onboarding script stopped working — an identity client had to be recreated under the new version. The code was fine; the checklist was missing. Every release now ships with an operator checklist: what changed in auth, in required config, in the environment file.

## Air-gapped is the same pipeline with a phase zero

For clusters without internet the installer runs in two phases: populate their internal registry with every image and chart the manifest names, then install from that registry. The manifest is generated from the same environment file, so "what does this version need" is a build artifact, not a spreadsheet. The tooling also carries a proxy option for the half-connected case. The point is that air-gapped is not a different product; it is the normal path with the network removed, and the manifest is what makes that true.

## What I would do differently

- **Start with the migration framework, not the third upgrade.** We built it after the upgrades hurt. Every rule above would have been a design choice instead of a retrospective item.
- **One environment file per cluster, in git, from day one.** Cluster-specific values that live in someone's shell history are the start of every snowflake.
- **Make the release cut a workflow, not a person.** The release dashboard — create a release, pick repositories, start, watch branches → pipelines → images — replaced a checklist that one engineer held in their head. A durable workflow engine underneath means a half-finished release survives a restart.
- **Version the checklist with the code.** The operator notes for an upgrade belong in the release branch, generated into the release, not in a wiki page someone remembers to update.

None of this is specific to AI. What is specific is the blast radius: a bad upgrade on a GPU cluster idles hardware that costs more per hour than the engineers fixing it.
