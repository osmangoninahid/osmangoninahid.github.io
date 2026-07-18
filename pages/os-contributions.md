---
layout: page
title: Open Source contributions
permalink: /os-contributions/
---

Notable contributions to Open Source projects

## [configurable-http-proxy (JupyterHub)](https://github.com/jupyterhub/configurable-http-proxy)

Contributed a **Redis-backed route store** to JupyterHub's proxy so routes survive proxy restarts: a new `lib/store.js` implementation, a 155-line test spec, documentation, and CLI flags. Upstream [PR #654](https://github.com/jupyterhub/configurable-http-proxy/pull/654) — in review.

## [modelaudit (promptfoo)](https://github.com/promptfoo/modelaudit)

Fixed a bug in the AI model security scanner where a **SafeTensors** u64 header low byte (`0x82`) was misclassified as a pickle `EXT1` opcode, producing false positives on legitimate model files. Fix plus tests in [PR #1745](https://github.com/promptfoo/modelaudit/pull/1745) — in review.

## [registry-image-check](https://github.com/osmangoninahid/registry-image-check)

Fork with fixes for container-registry image verification: **OCI manifest support**, a Harbor auth fix, and ECR basic-auth handling (2026).

## [tmux-claude-attn](https://github.com/osmangoninahid/tmux-claude-attn)

A tmux plugin that flags windows where **Claude Code is waiting for approval** — event-driven via hooks (no polling), TPM install, headless tmux test harness, shellcheck + Ubuntu/macOS CI. MIT licensed, v1 released 2026.

## [GitGossip](https://github.com/osmangoninahid/gitgossip)

GitGossip is an AI-powered CLI tool that generates **human-readable summaries** of Git commits, repository activity, and merge requests — helping developers quickly understand code changes and history.  
It’s designed with modular architecture principles, clean code practices, and built-in support for LLM-based semantic summarization. **v0.2** (2026) adds a zero-setup **agent-CLI provider** (Claude Code / Codex — no API key), a `commit` command, and custom prompt templates.

Published at [PyPI](https://pypi.org/project/gitgossip/) and [Homebrew](https://github.com/osmangoninahid/homebrew-gitgossip)
Give it a try - `uv tool install gitgossip` or `brew tap osmangoninahid/gitgossip && brew install osmangoninahid/gitgossip/gitgossip`

## [potaka()](https://github.com/osmangoninahid/potaka)

"Potaka" is programming language for the new learners. All the syntax of "Potaka" is written in Bengali. The syntaxes are organized in such a way so that people can read the code as a meaningful Bengali sentence. In other words, "Potaka" discloses a native Bengali sentence into programming logics. The main goal of "Potaka" is, giving the basic concept of programming and motivating the beginners to learn to program.

Have a look at http://potaka.io

## [spready](https://www.npmjs.com/package/spready)

Spready CLI simplifies the creation of modular scaffolding, enabling the swift setup of a backend RESTful structure. 
It assists in generating NodeJS, Express.js, and MongoDB frameworks, complete with essential CRUD functionalities.

## [flaskipy](https://pypi.org/project/flaskipy/)

The Flaskipy CLI streamlines the process of building modular scaffolding, facilitating the quick establishment of a backend RESTful setup. It aids in generating Python, Flask, and PostgreSQL frameworks, incorporating fundamental CRUD functionalities.

## [nestjs-shopee](https://www.npmjs.com/package/@osmangoninahid/nestjs-shopee)

Open-source NestJS module that simplifies Shopee API v2.0 interactions. This library streamlines Shopee authentication, store management, order processing (list, details, shipping, cancellations), and product inventory management. With easy-to-use methods, it handles key functionalities for Shopee's platform.

## [Android-Material-Design-Template](https://github.com/osmangoninahid/Android-Material-Design-Template)

Android Material Design UI Template , with Google Design support , card view , butterknife , CoordinateLayout,
CollapsingToolbar .


## [graphql-api](https://github.com/osmangoninahid/graphql-api)

A modularized approach to building an API using GraphQL, NodeJS/ExpressJS, and MongoDB.

## [microservice_k8s_nats](https://github.com/osmangoninahid/microservice_k8s_nats)

A sample boilerplate for an event-driven microservice designed for a ticket booking e-commerce platform using NodeJS/Typescript, MongoDB, NATs, Kubernetes, and Docker.

## [AgroSkyLab](https://github.com/TeamDurbar/AgroSkyLab)

Agro SkyLab was developed for the NASA Space Apps Challenge 2016 in Bangladesh. It caters to farmers or government authorities through mobile or web applications, utilizing location tracking to acquire NASA satellite images for detailed soil analysis. Soil samples are collected, tested in labs, and examined by experts. The conclusive findings are communicated to farmers or authorities via the app or web platform, aiding in informed decision-making.

