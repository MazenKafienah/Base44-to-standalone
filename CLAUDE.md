# CLAUDE.md — Base44-to-standalone

This file is read by Claude Code when working in this repository. The full set of rules lives in [AGENTS.md](AGENTS.md) — read that file first; it applies to Claude Code exactly as it does to any other agent.

## Quick summary

- This repo is application-independent: generic migration methodology and tooling, not a product codebase.
- PERSPECTIVES is the first sanitised reference case, tracked under `examples/perspectives/`.
- Production PERSPECTIVES code and canonical SQL never live here — see `Perspectives-app` and `Perspectives-worker`.
- No credentials, secrets, or real user/production data in any example or fixture.

## Current phase

This repository is in **MIG-000** (governance bootstrap) as of the commit that added this file. The PERSPECTIVES reference docs here (project map, legacy inventory, ledger, open decisions) describe workspace structure and verified facts only — they do not claim any later migration phase is complete.
