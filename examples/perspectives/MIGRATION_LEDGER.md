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

- **Status:** Not started. Scope proposed by MIG-000, requires separate authorisation before execution.
- **Proposed scope:**
  - Read-only inspection of the live Supabase schema.
  - Capture tables, columns, types, constraints, indexes, extensions, functions, triggers, generated columns, RLS status, RLS policies, and existing grants.
  - Reconcile the live schema against the planning documents (see `OPEN_DECISIONS.md` item 5's context note on the handover-vs-master-doc schema shape conflict).
  - Create a reproducible baseline in `Perspectives-worker`, marked as representing existing production state — not to be blindly reapplied.
  - Rebuild explicit least-privilege Data API grant SQL against the live tables.
  - No state-changing SQL executed without separate authorisation beyond MIG-001 itself.
  - Document the reusable schema-capture method in this toolkit (`docs/MIGRATION_PLAYBOOK.md`).

## Later phases

UNASSIGNED — not yet numbered or scoped. Expected to include (at minimum, order and numbering to be assigned when authorised): Python worker build-out (Phase 2 in the planning documents), frontend rebuild (Phase 3), and production deployment/automation (Phase 4+). Do not assume any of these have started.
