---
title: "Uses"
url: /uses/
---

What I work with day to day. Updated September 2026.

## Terminal

- **tmux** (prefix `C-Space`, resurrect + continuum, fzf popups). One session per ticket rooted at the workspace, windows for edit / run / AI, plus one window per cluster whose name selects the kubeconfig.
- **zsh** with starship, fzf + fd + bat, direnv, mise.
- **Neovim** with LazyVim as the daily editor; lazygit for git; k9s for clusters.
- A few aliases that earn their keep: `gst`, `gdf`, `gru` (`git remote update --prune`), `kctx`, `kns`, `ktop`, `twin` (`tmux neww -n`).

## Kubernetes and infra

- kubectl, helm, k9s, Argo CD, GitLab CI.
- Local clusters: kind and k3d (OrbStack VM), kwok for scheduler experiments, a fake GPU operator for GPU scheduling without GPUs, skaffold for build-and-load loops.
- Terraform and Ansible for the bits outside the cluster.

## Languages and tooling

- **Python**: uv, pytest, ruff, black, mypy. Flask and FastAPI services, Celery on RabbitMQ.
- **Go**: controllers, operators, exporters (client-go informers).
- **Node.js / TypeScript** when the job calls for it; Erlang/OTP in a former life (XMPP).

## Data and messaging

- MongoDB, PostgreSQL, Redis, RabbitMQ, NATS, MinIO and other S3-compatible object stores.

## Observability

- Prometheus and Thanos (multi-cluster), Grafana, Loki, NVIDIA DCGM and AMD SMI exporters.

## AI tooling

- **Claude Code** in the terminal, with my own skills: Clean Architecture and Clean Code review lenses, a career knowledge base that refreshes itself, a contributions ledger mined from git, graphify for codebase knowledge graphs.
- Things I built for this workflow: [gitgossip](https://github.com/osmangoninahid/gitgossip) (commit messages and repo digests from git history) and [tmux-claude-attn](https://github.com/osmangoninahid/tmux-claude-attn) (flags tmux windows where Claude is waiting on you).
- The posts here come out of a small content engine that indexes my repos, ranks stories worth telling, and drafts them with an evidence bundle. That will get its own post.

## Hardware

- MacBook Pro 14" M2 Pro, macOS.

## Languages

- Bengali (native), English (fluent), Arabic (beginner).

## Books that shaped how I work

- **Clean Architecture** and **Clean Code**, Robert C. Martin
- **Designing Distributed Systems**, Brendan Burns
- **Building Microservices**, Sam Newman
- **System Design Interview**, Alex Xu
- **Learn You Some Erlang**, Fred Hébert
- **Head First Design Patterns**, Freeman, Bates, Sierra, Robson
