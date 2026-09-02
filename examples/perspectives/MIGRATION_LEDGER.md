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
- **Next phase:** unassigned until MIG-001's findings are reviewed. Proposed candidate: **MIG-002 — Local Baseline Replay Validation and Reviewed Grant Execution Planning** (must remain separately authorised; must not execute grant changes against production without further explicit authorisation).

## Later phases

UNASSIGNED — not yet numbered or scoped beyond the MIG-002 candidate named above. Expected to include (at minimum, order and numbering to be assigned when authorised): local baseline replay validation, reviewed grant execution, Python worker build-out (Phase 2 in the planning documents), frontend rebuild (Phase 3), and production deployment/automation (Phase 4+). Do not assume any of these have started.
