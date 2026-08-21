# Migration Playbook: Base44 → Standalone Stack

A general methodology for taking a Base44-built application off the platform onto a self-hosted, standalone stack. This document is application-independent; PERSPECTIVES is tracked as the first worked example under `examples/perspectives/`.

## Why this playbook exists

Base44 lets you prototype quickly, but it locks the backend into Deno serverless functions running a proprietary SDK (`@base44/sdk`, `base44.asServiceRole.*`). There is no supported path to run that backend on your own infrastructure — every function must be rewritten against a standalone stack. The frontend is portable in principle (it's a normal Vite/React SPA) but typically carries dependency bloat from Base44's default scaffolding that shouldn't be carried forward uncritically.

This playbook treats the Base44 app as a **specification**, not as code to port line-by-line, and separates the migration into repository-scoped phases with explicit stop/go gates.

## General phase structure

1. **Workspace bootstrap** — establish repository boundaries before writing any product code. Decide, in writing, which repository owns the frontend, which owns the backend/worker and canonical database migrations, and which (if any) is a generic toolkit. Protect the legacy Base44 repository as read-only evidence throughout the migration — do not mutate it.
2. **Live-state baseline** — before assuming any planning document is accurate, verify what actually exists in the target infrastructure (e.g. does a database already exist, does it already have a schema, what tables does it actually contain). Planning documents drift; live infrastructure state is checked, not assumed.
3. **Legacy audit** — read the Base44 app's entities, serverless functions, and frontend source directly from the repository, not from memory or from stale planning documents. Record verified counts and names, and flag any discrepancy against what any planning document claims rather than silently trusting the document.
4. **Schema and access design** — design the standalone database schema, then design the *access* layer separately: which tables the public frontend can read (and how), which are user-scoped and require row-level isolation, and which are backend/worker-only and should never be reachable from a public API surface. These are two different design steps and should not be conflated.
5. **Worker rebuild** — rebuild backend logic as an idiomatic module in the new stack's language, not as a transliteration of the Deno functions. Use authoritative external APIs for verifiable facts; reserve LLM calls for genuinely generative work only.
6. **Frontend rebuild** — port UI and presentational components with minimal changes; rewrite the data-access layer entirely against the new backend/database client; drop unused legacy dependencies rather than carrying them forward by default.
7. **Deployment and automation** — wire up scheduling/hosting for the backend and hosting for the frontend as a distinct, later step, once both are individually verified to work locally against real (or realistically seeded) data.

## Reusable checklist for a workspace-bootstrap phase

- [ ] Confirm the exact local and remote (origin) location of every repository involved before writing anything.
- [ ] Confirm every repository is clean and has no in-progress git operation (merge/rebase/cherry-pick/etc.) before creating a working branch.
- [ ] Confirm the legacy source repository is excluded from the write scope of the bootstrap phase.
- [ ] Read all canonical planning documents completely, and explicitly note which document wins when two disagree, rather than picking silently.
- [ ] Verify — from the actual legacy repository, not from a planning document's claim — the entity/table count, the backend function count and names, and the frontend dependency/component footprint. Report any discrepancy exactly as found.
- [ ] Record open, unresolved product/architecture decisions in a dedicated document rather than resolving them implicitly through code choices.
- [ ] Scope every new repository's governance docs (README/AGENTS/CLAUDE-equivalent) to state clearly what does and does not belong in that repository, especially where a secret (service-role-equivalent credential) may or may not live.

## Reusable checklist for a schema/grants reconciliation phase

- [ ] Treat the live database (once inspected) as canonical over any planning document.
- [ ] Capture the live schema in full — tables, columns, types, constraints, indexes, extensions, functions, triggers, generated columns, and existing access-control state — before writing new migration files.
- [ ] Rebuild access-grant scripts (e.g. public API read grants) against the live table set, never against a planning document's older table list.
- [ ] Explicitly resolve, or explicitly record as still-open, any table whose intended access level is ambiguous or contested across planning documents.
- [ ] Never execute state-changing SQL against a live database without a separate, explicit authorisation step distinct from the inspection step.

## Lessons log

Lessons and reusable patterns are appended here as they're learned from the PERSPECTIVES reference migration (see `examples/perspectives/MIGRATION_LEDGER.md` for the phase-by-phase record).

- **MIG-000:** Planning documents can be relocated or go temporarily missing between sessions; a bootstrap phase should verify the exact path of every canonical document before reading, and should stop and report rather than substituting a stale document when a canonical one is absent — but should also accept a corrected path from the user rather than treating relocation as a repository-boundary violation.
- **MIG-000:** A prior audit's counts (entities, functions, components) can go stale as a prototype keeps evolving after the audit was written. Always re-verify counts directly from the repository at bootstrap time rather than propagating a remembered number.
