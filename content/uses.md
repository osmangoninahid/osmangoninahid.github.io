---
title: "Uses"
url: /uses/
---

What I work with day to day. Updated September 2026.

## Terminal

- **tmux** (prefix `C-Space`, resurrect + continuum, fzf popups). One session per ticket, windows for edit / run / AI, one window per cluster whose name picks the kubeconfig.
- **zsh** with starship, fzf + fd + bat + ripgrep, direnv, mise, zoxide.
- **Neovim** (LazyVim) as the editor. **lazygit** for git, **k9s** for clusters.
- Everyday CLI: `jq`, `yq`, `curl`, `mc` (MinIO client), `skopeo`, `sops` + `age`, `keyring` for secrets in scripts, `gh` and `glab`, `caffeinate`.
- Aliases that earn their keep: `gst`, `gdf`, `gru`, `ggpush`, `ggpull`, `kctx`, `kns`, `ktop`, `twin` (`tmux neww -n`).

## Kubernetes and infra

- kubectl, kubectx/kubens, helm, k9s, Argo CD, GitLab CI.
- Local clusters: kind and k3d (OrbStack), kwok for scheduler experiments, a fake GPU operator for GPU scheduling without GPUs, skaffold for build-and-load loops.
- Terraform and Ansible for what lives outside the cluster.

## Languages and tooling

- **Python**: uv, pytest, ruff, black, mypy. Flask and FastAPI services, Celery on RabbitMQ.
- **Go**: controllers, operators, exporters (client-go informers).
- **Rust**: cargo, for CLI tools and anything that must be fast and small.
- **Bash** and **Jinja**: glue, installers, templated manifests and configs.
- **Node.js / TypeScript** when the job calls for it; Erlang/OTP in a former life (XMPP).

## Observability

- Prometheus and Thanos (multi-cluster), Grafana, Loki, NVIDIA DCGM and AMD SMI exporters.

## AI tooling

- **Claude Code** in the terminal, with my own skills: Clean Architecture and Clean Code review lenses, a career knowledge base that refreshes itself, a contributions ledger mined from git, graphify for codebase knowledge graphs.
- Things I built for this workflow: [gitgossip](https://github.com/osmangoninahid/gitgossip) (commit messages and repo digests from git history) and [tmux-claude-attn](https://github.com/osmangoninahid/tmux-claude-attn) (flags tmux windows where Claude is waiting on you).
- The posts here come out of a small content engine that indexes my repos, ranks stories worth telling, and drafts them with an evidence bundle. That will get its own post.

## Hardware

- MacBook Pro 14" M2 Pro, macOS.
- MacBook Pro 16" M3 Pro, macOS.
- NVIDIA DGX Spark.

## Languages

- Bengali (native), English (fluent), Arabic (beginner).

## Books that shaped how I work

- **Clean Architecture** and **Clean Code**, Robert C. Martin
- **Designing Distributed Systems**, Brendan Burns
- **Building Microservices**, Sam Newman
- **System Design Interview**, Alex Xu
- **Learn You Some Erlang**, Fred Hébert
- **Head First Design Patterns**, Freeman, Bates, Sierra, Robson
