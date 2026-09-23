---
title: "Open Source contributions"
url: /os-contributions/
---
Things I have contributed to or published.

## Campfire (coming soon)

A local-first cockpit for everyday AI work: browse, search, thread and continue every AI session from one place. Desktop and terminal faces over one engine, written in Rust. Nothing leaves your machine. Public release once the logo is done and a few bugs are fixed.

## [configurable-http-proxy (JupyterHub)](https://github.com/jupyterhub/configurable-http-proxy)

A Redis-backed route store so routes survive proxy restarts: implementation, tests, docs and CLI flags. [Commit d739434](https://github.com/osmangoninahid/configurable-http-proxy/commit/d739434).

## [modelaudit (promptfoo)](https://github.com/promptfoo/modelaudit)

A false positive: a SafeTensors header byte (`0x82`) was read as a pickle `EXT1` opcode. [Commit 6d70f50](https://github.com/osmangoninahid/modelaudit/commit/6d70f500).

## [registry-image-check](https://github.com/osmangoninahid/registry-image-check)

Fork with fixes for container-registry image verification: **OCI manifest support**, a Harbor auth fix, and ECR basic-auth handling (2026).

## [tmux-claude-hatch](https://github.com/osmangoninahid/tmux-claude-session-manager)

A fork of [tmux-claude-hatch](https://github.com/craftzdog/tmux-claude-hatch) by Takuya Matsuyama: a tmux plugin that runs Claude Code in a popup per project, with an `fzf` picker and a bell for the window that needs you. My additions: `prefix+y` always opens a fresh session instead of re-attaching to the directory's session (the old behaviour stays behind `@claude_reattach`), and sessions are named from the first prompt, with a title column in the picker, so you can tell ten sessions apart.

## [mkui](https://github.com/osmangoninahid/mkui)

Browse and run Makefile targets from a menu: lists every target, filter by typing, pick one and run it. One static binary, `brew install osmangoninahid/tap/mkui`.

## [tmux-claude-attn](https://github.com/osmangoninahid/tmux-claude-attn)

A tmux plugin that marks windows where Claude Code is waiting for you. Hooks, no polling. TPM install, CI on Ubuntu and macOS. MIT.

## [GitGossip](https://github.com/osmangoninahid/gitgossip)

A CLI that turns git history into readable summaries of commits and merge requests. v0.2 can use Claude Code or Codex directly with no API key, adds a `commit` command and custom prompts.

Published at [PyPI](https://pypi.org/project/gitgossip/) and [Homebrew](https://github.com/osmangoninahid/homebrew-gitgossip)
Give it a try - `uv tool install gitgossip` or `brew tap osmangoninahid/gitgossip && brew install osmangoninahid/gitgossip/gitgossip`

## [potaka()](https://github.com/osmangoninahid/potaka)

A programming language for beginners where every statement reads as a Bengali sentence. Built to teach the first ideas of programming to people who do not read English well.

Have a look at http://potaka.io

## [spready](https://www.npmjs.com/package/spready)

Scaffolds a modular Node.js, Express and MongoDB REST backend with CRUD in place.

## [flaskipy](https://pypi.org/project/flaskipy/)

Same idea for Python: scaffolds a Flask and PostgreSQL REST backend with CRUD.

## [nestjs-shopee](https://www.npmjs.com/package/@osmangoninahid/nestjs-shopee)

A NestJS module for the Shopee API v2: auth, stores, orders and inventory.

## [Android-Material-Design-Template](https://github.com/osmangoninahid/Android-Material-Design-Template)

Android Material Design UI Template , with Google Design support , card view , butterknife , CoordinateLayout,
CollapsingToolbar .


## [graphql-api](https://github.com/osmangoninahid/graphql-api)

A modularized approach to building an API using GraphQL, NodeJS/ExpressJS, and MongoDB.

## [microservice_k8s_nats](https://github.com/osmangoninahid/microservice_k8s_nats)

A sample boilerplate for an event-driven microservice designed for a ticket booking e-commerce platform using NodeJS/Typescript, MongoDB, NATs, Kubernetes, and Docker.

## [AgroSkyLab](https://github.com/TeamDurbar/AgroSkyLab)

Built for the NASA Space Apps Challenge 2016. Farmers get soil analysis from NASA satellite images for their location, backed by lab tests, through a mobile or web app. People's Choice winner.

