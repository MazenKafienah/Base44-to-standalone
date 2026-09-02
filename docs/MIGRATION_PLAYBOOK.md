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

- [ ] Treat the live database (once inspected) as canonical over any planning document **for factual, already-existing state** — but do not let that same evidence silently settle a product decision that hasn't actually been made yet. "It already works this way" and "it should keep working this way" are different claims.
- [ ] Capture the live schema in full — tables, columns, types, constraints, indexes, extensions, functions, triggers, generated columns, RLS status/policies, and existing access-control state — before writing new migration files. Use the credential-free, read-only capture method in [`LIVE_SUPABASE_SCHEMA_CAPTURE.md`](LIVE_SUPABASE_SCHEMA_CAPTURE.md): a human runs one catalogue-only query by hand and exports the result; the coding agent never connects to the database directly.
- [ ] Rebuild access-grant scripts (e.g. public API read grants) against the live table set, never against a planning document's older table list.
- [ ] Explicitly resolve, or explicitly record as still-open, any table whose intended access level is ambiguous or contested across planning documents. When a live RLS policy or grant already implements one side of a documented conflict, present that as evidence for a human decision — do not treat "it's already configured this way" as the decision itself.
- [ ] Keep schema-shape SQL and access-grant SQL in separate files — they get reviewed and executed on different timelines by different people.
- [ ] Never execute state-changing SQL against a live database without a separate, explicit authorisation step distinct from the inspection step. Guard any drafted DDL/grant SQL against accidental execution (e.g. a `RAISE EXCEPTION` guard block) until that separate authorisation exists.
- [ ] Keep capture, baseline generation, replay validation, and production execution as four distinct steps — never assume a baseline "works" until it has actually been replay-tested against a disposable database, and say so explicitly wherever the baseline is referenced.

## Lessons log

Lessons and reusable patterns are appended here as they're learned from the PERSPECTIVES reference migration (see `examples/perspectives/MIGRATION_LEDGER.md` for the phase-by-phase record).

- **MIG-000:** Planning documents can be relocated or go temporarily missing between sessions; a bootstrap phase should verify the exact path of every canonical document before reading, and should stop and report rather than substituting a stale document when a canonical one is absent — but should also accept a corrected path from the user rather than treating relocation as a repository-boundary violation.
- **MIG-000:** A prior audit's counts (entities, functions, components) can go stale as a prototype keeps evolving after the audit was written. Always re-verify counts directly from the repository at bootstrap time rather than propagating a remembered number.
- **MIG-000A:** A routine "check whether an alternate auth path exists" diagnostic can itself leak a live credential (e.g. a credential-helper lookup printing a stored OAuth token to command output). The safe default is to never run credential-helper or Keychain-style lookups at all, even as a read-only diagnostic — if a tool isn't already authenticated, stop and say so instead of probing for how it's authenticated elsewhere.
- **MIG-001:** A live database's actual grant/RLS configuration can directly resolve a documented "which of these two behaviours is correct" conflict — but can also surface a *new* conflict nobody had written down (e.g. a table's live RLS policy shape matching a different category of table than expected). Both are worth surfacing explicitly rather than only checking off the conflicts that were already anticipated.
- **MIG-001:** Reconstructing DDL from `information_schema` alone has real gaps — e.g. a `vector` extension column's dimension isn't exposed the way a `varchar`'s length is. Flag such gaps explicitly in the baseline rather than filling them in from a planning document's assumption.
- **MIG-002:** A freshly-provisioned local Supabase development stack can exhibit the exact same "default" behavior as a live project (e.g. broad table grants to `anon`/`authenticated` on new tables) — this is strong, cheap evidence that a finding is a platform default rather than a project-specific misconfiguration, and it also means the local environment is unusually well-suited to rehearsing exactly the fix. See [`LOCAL_SUPABASE_REPLAY_AND_GRANT_REHEARSAL.md`](LOCAL_SUPABASE_REPLAY_AND_GRANT_REHEARSAL.md).
- **MIG-002:** `SET LOCAL` outside an explicit transaction silently does nothing in Postgres — an effective-access test built on it will appear to pass (or look confusing) while actually still running as the original, usually far-more-privileged, role. Always wrap role/claims-switching test queries in an explicit `BEGIN ... COMMIT`.
- **MIG-003:** Re-verify reviewed-artifact hashes at *every* gate that touches production — preflight, immediately before handing over execution instructions, and again after execution — not just once at the start of the phase. Nothing changed in this case, but the check is cheap and the one time it matters is exactly the time a phase's authors won't expect it.
- **MIG-003:** A "drift gate" (comparing a fresh capture against the last-known-good one immediately before any production change) is worth running even when nothing is expected to have changed — it's what turns "we assume production still looks like it did when we planned this" into a checked fact, for the cost of one read-only query.
- **MIG-003:** When a coding agent must never touch production directly, the actual execution step is necessarily a human pasting reviewed SQL into a console. Make that moment as low-risk as possible: name the exact guard to remove and nowhere else to touch, state plainly what "success" looks like, and give an explicit "stop and report the exact error, don't improvise" instruction for anything else. A multi-statement SQL block pasted as one submission runs as one implicit Postgres transaction — worth confirming and stating explicitly, since it means a failure partway through rolls back everything in that submission rather than leaving a half-applied state.
