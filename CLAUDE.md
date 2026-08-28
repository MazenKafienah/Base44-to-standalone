# CLAUDE.md — Base44-to-standalone

This file is read by Claude Code when working in this repository. The full set of rules lives in [AGENTS.md](AGENTS.md) — read that file first; it applies to Claude Code exactly as it does to any other agent.

## Quick summary

- This repo is application-independent: generic migration methodology and tooling, not a product codebase.
- PERSPECTIVES is the first sanitised reference case, tracked under `examples/perspectives/`.
- Production PERSPECTIVES code and canonical SQL never live here — see `Perspectives-app` and `Perspectives-worker`.
- No credentials, secrets, or real user/production data in any example or fixture.

## Permanent credential-handling rules

Never inspect Keychain, credential helpers, environment variables, shell history, or populated `.env` files as a diagnostic step; never read/print/commit a credential of any kind; never connect a coding agent directly to a live production database (see [AGENTS.md](AGENTS.md) for the full list). If a required CLI or auth is unavailable, stop and report that plainly.

## Current phase

MIG-000 and MIG-001 are both complete for the PERSPECTIVES reference case. The reference docs here (project map, legacy inventory, ledger, open decisions, schema reconciliation) describe workspace structure and verified facts only — they do not claim implementation, replay validation, or grant execution has happened.
