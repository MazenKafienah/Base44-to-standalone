# PERSPECTIVES — Migration Ledger

Phase-by-phase record of the PERSPECTIVES migration. Append new phases as they happen; do not rewrite the record of a phase that already occurred.

## MIG-000 — Multi-Repository Workspace Bootstrap

- **Status:** Complete.
- **Scope:** Established governance, repository boundaries, and migration documentation across the four PERSPECTIVES repositories. No production code, dependencies, SQL, or deployment.
- **`Perspectives-app` bootstrap commit:** `d3bf2717c79ba5ef607911aad4abc24b1e39295a` (branch `bootstrap/mig-000`, message: `chore: bootstrap MIG-000 repository governance`)
- **`Perspectives-worker` bootstrap commit:** `e4ea9c9ec747b619c185123b54fd12455c6fc6df` (branch `bootstrap/mig-000`, message: `chore: bootstrap MIG-000 repository governance`)
- **`Base44-to-standalone` bootstrap:** recorded in the containing commit that added this ledger entry, on branch `bootstrap/mig-000` of this repository.
- **Note:** the canonical planning documents (`PERSPECTIVES_COMPLETE_HANDOVER.docx`, `PERSPECTIVES_MASTER_DOC.md`, `PERSPECTIVES_EXECUTION_RUNBOOK.md`, `PERSPECTIVES_SESSION_HANDOFF.md`) were located mid-phase at the top-level workspace folder `Perspectives Migration Project/` (a sibling of all four repositories, not inside any of them) rather than at their originally expected path inside `Perspectives_Prototype`. MIG-000 paused, was given the corrected path by the workspace owner, and resumed from the planning-document reading step.

## MIG-001 — Live Supabase Schema Baseline and Data API Grant Reconciliation

- **Status:** Complete (both repository branches committed and pushed; neither merged to `main`).
- **Scope actually performed:**
  - Read-only, credential-free capture of the live Supabase schema: Claude Code authored a catalogue-only SQL query; the project owner ran it manually in the Supabase SQL Editor and exported the single-row result to a local file outside every Git repository; Claude Code read that file from disk. No direct Supabase connection of any kind occurred.
  - Reconciled the live schema against all three planning documents. Resolved the handover-vs-master-doc schema-shape conflict in the handover's favour (confirmed, not assumed). Surfaced two new facts neither document anticipated (author relationships via a single FK + JSON co-authors field, not a join table; tag/subtopic relationships via array columns, not a join table) and one new gap (the `articles.embedding` vector dimension is not recoverable from `information_schema`).
  - Two decisions were resolved by the project owner during this phase: `article_processing_log` is worker-only (Data API), and V1 requires authenticated sign-in for all reader-facing reads (no anonymous browsing). See `OPEN_DECISIONS.md`.
  - Produced a reproducible, guarded existing-state baseline (`Perspectives-worker/supabase/baseline/`) and a guarded, least-privilege Data API grant proposal (`Perspectives-worker/supabase/grants/`) — both explicitly marked as not replay-tested and not executed.
  - Documented the reusable, application-independent capture method in this toolkit (`docs/LIVE_SUPABASE_SCHEMA_CAPTURE.md`, referenced from `docs/MIGRATION_PLAYBOOK.md`).
  - **Live mutation: none.** **Baseline replay validation: pending.** **Grant execution: pending.**
- **`Perspectives-worker` MIG-001 commits** (branch `audit/mig-001-live-schema`):
  - Stage A: `2cf8f43c19d3806a477a1fc7552a9bc7c961bbe6` — `chore: add MIG-001 read-only schema capture`
  - Stage B: `b0e6fa58877b964dfe985404e0672147504f9b87` — `chore: capture and reconcile live Supabase schema`
- **`Base44-to-standalone` MIG-001 commit:** recorded in the containing commit that added this ledger update, on branch `audit/mig-001-live-schema` of this repository.
- **Mid-phase incident:** during the preceding MIG-000A review/merge phase (not MIG-001 itself), a diagnostic credential-helper lookup printed a live GitHub OAuth token into a session transcript. The token was revoked and replaced before MIG-001 began; MIG-001 operated under a permanent, standing prohibition on credential-helper/Keychain/environment-variable inspection as a result (see AGENTS.md in every repository).

## MIG-002 — Local Baseline Replay Validation and Reviewed Grant Execution Planning

- **Status:** Complete (both repository branches committed and pushed; neither merged — stacked on the still-open, still-unmerged MIG-001 branches/PRs in each repo).
- **Scope actually performed:**
  - Installed a local, credential-free, Supabase-faithful replay environment (Colima + the official Supabase CLI local development stack) after stating exactly what would be installed and why; no production connection, login, or project link at any point.
  - Replayed the MIG-001 existing-state baseline against the disposable local stack: **full success, zero errors, no correction needed.**
  - Structurally compared the live capture against a fresh local capture (using the identical committed capture query): **220 of 223 comparisons exact match**; the remaining 3 are expected environment noise (extension patch version, platform-internal schema names) plus the pre-existing, still-open embedding-dimension limitation.
  - Corrected a documentation imprecision from MIG-001: the capture query returns exactly **26** top-level JSON keys (not "24" as MIG-001's prose said in places) — a prose-only correction, no MIG-001 file altered.
  - Built a deterministic, evidence-cited 33-row target access matrix (11 tables × 3 roles) and locally rehearsed the reviewed grant plan: **exact match to target, idempotent (verified by double-application), and reversible (verified rollback in both directions).**
  - Empirically proved, with a live before/after test against the local replica, that the residual `article_processing_log` RLS policy alone cannot grant Data API access once the underlying table privilege is revoked — confirming the worker-only decision is enforceable today via the grant alone, independent of when the stale policy itself is eventually removed.
  - Produced a review-only production execution plan (pre-requisites, sequence, explicit abort conditions) — not executed, not authorising execution.
  - Documented the reusable, application-independent replay/rehearsal method in this toolkit (`docs/LOCAL_SUPABASE_REPLAY_AND_GRANT_REHEARSAL.md`).
  - **Live mutation: none.** **Baseline replay validation: complete (PASS).** **Grant execution: still pending — requires a further, separately authorised phase.**
- **`Perspectives-worker` MIG-002 commits** (branch `audit/mig-002-local-replay`, based on `audit/mig-001-live-schema`):
  - `083a6309cb215bdaf90f30786b7ce1aa0c143a75` — "MIG-002 local baseline replay and schema comparison"
  - `cba58025bd0e0b691dfe3881925a7f9b87cb0e77` — "MIG-002 grant/rollback rehearsal, execution planning and final verification"
  - Stacked PR: `Perspectives-worker` #3, base `audit/mig-001-live-schema`, explicitly dependent on the still-unmerged MIG-001 PR #2.
- **`Base44-to-standalone` MIG-002 commit:** recorded in the containing commit that added this ledger update, on branch `audit/mig-002-local-replay` of this repository (stacked on `audit/mig-001-live-schema`).
- **Next phase:** unassigned until MIG-002's findings are reviewed. Proposed candidate remains a reviewed **production grant execution phase**, gated on the pre-requisites in `Perspectives-worker/docs/MIG002_PRODUCTION_EXECUTION_PLAN.md` (fresh live capture, drift check, explicit separate authorisation, approved SQL hash, rollback readiness) — must not execute anything against production without that separate authorisation.

## MIG-003 — Controlled Production Grant Execution and Independent Post-Execution Verification

- **Status:** **MIG-003 production client-role grant transition: COMPLETE AND VERIFIED.**
- **Scope actually performed:**
  - Read-only preflight: independently re-verified all four repositories, both MIG-001 and MIG-002 pull requests (open, unmerged), and the exact SHA-256 of the reviewed grant and rollback artifacts. Zero drift found anywhere.
  - Fresh, credential-free pre-execution capture (manual, by the project owner) compared against the MIG-001 historical capture: **225/225 exact matches, zero drift** — confirmed every MIG-002 target assumption still held against the live database before touching anything.
  - The project owner executed the exact, unmodified MIG-002 reviewed grant plan (`Perspectives-worker/supabase/grants/MIG002_REVIEWED_GRANT_PLAN.sql`, SHA-256 `c1fb20bac47386f8e4402f66718db556f2f9a43100e6b6c92fe332e329088303`) manually in the live Supabase SQL Editor, bypassing only its execution guard in the editor's paste buffer — the committed file itself was never modified. Claude Code never connected to production, never held a production credential, and did not execute the SQL itself.
  - Fresh post-execution capture (again manual) independently verified: **exact match to the target access matrix on all 22 (table, role) pairs**, **zero non-privilege structural change** across all 16 compared categories (tables, columns, constraints, indexes, triggers, functions, policies, RLS flags, and more), `service_role` fully unaffected, and the residual `article_processing_log` RLS policy confirmed still present and — as MIG-002 had already proven locally — inert without the now-revoked table grant.
  - Rollback was not required and was not performed; every pass criterion was satisfied on the first check.
  - **Live mutation: yes — the Data API grant/revoke transition only.** **Not resolved by this phase:** the residual `article_processing_log` RLS policy remains in place (cleanup requires separate authorisation); the `articles.embedding` live vector dimension remains unverified; no application behaviour was tested; nothing was deployed; the schema baseline itself has still never been applied to production.
- **`Perspectives-worker` MIG-003 commits** (branch `audit/mig-003-production-grants`, based on `audit/mig-002-local-replay`):
  - `449e3f63e73e4b30a9734b0a7c6ac13fac63fd4f` — "MIG-003: pre-execution verification and drift gate (GO)"
  - `8f5cf521538ee3e911c73bc665a30788081a2ff9` — "MIG-003: record production grant execution and verification"
  - Stacked PR: `Perspectives-worker` #4, base `audit/mig-002-local-replay`, explicitly dependent on the still-unmerged MIG-002 PR #3 (itself dependent on MIG-001 PR #2).
- **`Base44-to-standalone` MIG-003 commit:** recorded in the containing commit that added this ledger update, on branch `audit/mig-003-production-grants` of this repository (stacked on `audit/mig-002-local-replay`).
- **Next phase:** unassigned. Candidates include the residual `article_processing_log` RLS policy cleanup (separate authorisation required) and live verification of the `articles.embedding` vector dimension — neither is urgent, since the grant layer already enforces the intended access boundary regardless of the stale policy's presence.

## Later phases

UNASSIGNED. Expected to include (at minimum, order and numbering to be assigned when authorised): residual `article_processing_log` RLS policy cleanup, live verification of the `articles.embedding` vector dimension, Python worker build-out (Phase 2 in the planning documents), frontend rebuild (Phase 3), and production deployment/automation (Phase 4+). Do not assume any of these have started.
