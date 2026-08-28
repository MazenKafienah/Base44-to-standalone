# AGENTS.md — Base44-to-standalone

Instructions for any AI coding agent (Claude Code, or otherwise) working in this repository.

## What this repository is

An application-independent toolkit and methodology for migrating Base44 applications off the platform. PERSPECTIVES is tracked here only as a sanitised **reference case**, not as a place to build PERSPECTIVES itself.

## Hard boundaries — do not cross these

- **No production PERSPECTIVES code.** Frontend code belongs in `Perspectives-app`; worker code and canonical migrations belong in `Perspectives-worker`. If you find yourself writing a real feature for PERSPECTIVES, stop — it belongs in one of those two repositories.
- **No canonical SQL.** Never create a second canonical copy of PERSPECTIVES' production schema here. Reference or summarise; do not duplicate as a source of truth.
- **No secrets, private data, or production records.** Every fixture, example, and script in this repository must be synthetic or sanitised. Before adding any example content, confirm it contains no credentials, no real user data, and no unredacted internal identifiers.
- **Reusable, not one-off.** Content added under `examples/perspectives/` should generalise into the methodology (`docs/MIGRATION_PLAYBOOK.md`) where applicable — that is the point of using PERSPECTIVES as the first reference case.

## Working with the PERSPECTIVES example

- `examples/perspectives/PROJECT_MAP.md` — the four-repository workspace layout and ownership boundaries. Keep it accurate as repositories evolve.
- `examples/perspectives/LEGACY_INVENTORY.md` — verified facts about the Base44 prototype (entities, functions, stack). Only record what has been verified from the actual repository; flag anything not yet checked.
- `examples/perspectives/MIGRATION_LEDGER.md` — the phase-by-phase migration log with commit references. Append new phases; do not rewrite history that already happened.
- `examples/perspectives/OPEN_DECISIONS.md` — unresolved questions that block later phases. Do not resolve these unilaterally; they require the product owner's explicit confirmation.

## Permanent credential-handling rules (application-independent, do not remove or weaken)

These apply to any migration this toolkit is ever used for, not just PERSPECTIVES, following a real incident where a routine diagnostic command printed a live credential into a session transcript:

- Never run a credential-helper, Keychain, or secret-store lookup as a diagnostic or troubleshooting step, for any platform's CLI.
- Never enumerate or print environment variables as a diagnostic step.
- Never read a populated `.env`, `.env.local`, or similar file.
- Never read, request, print, log, or commit a database credential, API key, access token, or connection string containing credentials.
- If a required CLI or authentication is unavailable, stop and report exactly that — do not probe for an alternate way to recover or work around missing credentials.
- Never connect a coding agent directly to a live production database. Live database facts are gathered only via a read-only query that a human runs manually and exports for the agent to read — see `docs/LIVE_SUPABASE_SCHEMA_CAPTURE.md` for the full method.

## Current status

MIG-000 and MIG-001 governance and PERSPECTIVES reference documentation. The generic live-schema-capture method (`docs/LIVE_SUPABASE_SCHEMA_CAPTURE.md`) now exists and is application-independent. No other generic migration scripts, validators, or templates exist yet.
