# PERSPECTIVES — MIG-001 Schema Reconciliation (Sanitised Summary)

This is the toolkit-side, sanitised summary of MIG-001. The complete, canonical reconciliation report — with full object inventories, grant tables, and RLS policy text — lives in `Perspectives-worker/docs/MIG001_LIVE_SCHEMA_RECONCILIATION_REPORT.md`. No credential, connection string, project reference, or private data appears in either document.

## What MIG-001 did

Captured the live PERSPECTIVES Supabase schema **read-only, without any coding agent ever connecting to the database** — a human ran one catalogue-only SQL query by hand in the Supabase SQL Editor and exported the single-row result to a local file, which Claude Code then read from disk. See `docs/LIVE_SUPABASE_SCHEMA_CAPTURE.md` in this repository for the reusable method.

## Headline outcome

The live 11-table schema matches the project's **handover document**, resolving a previously-open "which of two conflicting planning documents describes the real database" question decisively. It also surfaced that the live database's **Data API grants were far broader than intended** on every table (a legacy platform default, not a deliberate choice) — RLS was the only thing preventing that from being exploitable. A corrected, least-privilege grant proposal was drafted from the verified live table set; nothing was executed.

## Decisions this phase resolved (product-owner confirmed, not inferred)

1. One previously-open table-access question was resolved toward the more conservative of two options, specifically because the table holds internal, non-reader-facing records — even though the live evidence at the time actually pointed toward the *other* option having already been configured. This is a useful reminder for the general method: live evidence can inform a decision without being allowed to make it.
2. A related, newly-discovered question (whether anonymous/unauthenticated requests should be able to read certain reader-facing tables) was resolved in favour of requiring sign-in for all such reads in the initial version of the product — matching what the live database's row-level security already enforced, so no database change was actually needed for that half of the finding.

## New facts this phase surfaced (not previously documented)

- The live database's approach to a many-to-many relationship (authors on articles) turned out to differ from *both* competing planning documents' assumptions — neither predicted the actual mechanism in use. A reminder that "the live database matches document A, not document B" doesn't mean document A is fully correct either; each specific claim still needs independent verification.
- Reconstructing full DDL from catalogue metadata alone has real limits — a vector-type column's dimension parameter isn't exposed by `information_schema` the way a text column's length limit is. This is now a documented gap rather than an assumed value.

## Status

Baseline and grant-proposal artifacts exist in `Perspectives-worker` (`supabase/baseline/`, `supabase/grants/`), both guarded against accidental execution, **neither replay-tested nor executed**. See the migration ledger for phase status and the open-decisions file for what remains unresolved.
