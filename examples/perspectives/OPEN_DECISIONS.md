# PERSPECTIVES — Open Decisions

Preserved conflicts and unresolved questions carried forward from the planning documents and from live-schema evidence. Each requires Maz's explicit confirmation before the affected code or SQL is written. Resolved items are kept below with their outcome for the audit trail — resolution here does not mean implementation is complete.

## Open

### 1. `runMonthIngestion` — port, replace, or deliberately drop?

- Verified to exist in the legacy prototype (`base44/functions/runMonthIngestion/`) but undocumented in the earlier stale audit.
- Behaviour: month-targeted ingestion, distinct from the regular bi-daily broad pipeline.
- Must not disappear silently — a decision to drop it must be a conscious one, recorded here once made, not an omission discovered later.

### 2. `pruneOldArticles` — port, replace, or deliberately drop?

- Verified to exist in the legacy prototype (`base44/functions/pruneOldArticles/`) but undocumented in the earlier stale audit.
- Behaviour: storage-hygiene pruning that must preserve saved/followed articles rather than deleting them indiscriminately.
- Any replacement must explicitly carry forward the saved-article protection requirement; must not disappear silently.

### 3. Exact current Anthropic API model identifiers

- Planning documents disagree on the summariser split: the handover specifies "Claude Sonnet 4.6" for extraction and "Claude Haiku 4.5" for writing; the master document's code bodies hardcode `claude-3-5-sonnet-20241022` and `claude-3-5-haiku-20241022`.
- The `claude-3-5-*` strings are known-stale and must not be copied into new code.
- **Gate:** confirm the exact current Anthropic API model identifiers against official Anthropic documentation at implementation time, not from any planning document in this workspace.

### 4. `articles.embedding` vector dimension (new, surfaced by MIG-001)

- The live capture confirms `articles.embedding` is an `extensions.vector` column, but `information_schema.columns` does not expose the vector's dimension (typmod) the way it exposes a `varchar`'s length.
- The planning documents assume 1536 dimensions (OpenAI `text-embedding-3-small`), but this has **not** been verified against the live catalogue.
- **Gate:** confirm the true live dimension (e.g. via `\d+ public.articles` in the SQL Editor, or a follow-up targeted capture of `pg_attribute.atttypmod`) before any embedding-writing code is built or before the baseline is replay-validated.

## Resolved

### 5. `article_processing_log` access level — RESOLVED: worker-only

- **Original conflict:** one handover statement described authenticated `SELECT` access; the execution runbook's grant procedure treated it as worker-only.
- **Live evidence at decision time:** the live database already had an RLS policy (`processing_log_authenticated_read`) permitting authenticated `SELECT` — the same policy shape used on the six agreed public-read tables, not the deny-all shape used on the other two worker-only tables. This evidence pointed toward the table already being configured for authenticated access, i.e. toward the *other* option.
- **Decision (Maz, during MIG-001):** worker-only. Chosen deliberately notwithstanding the live evidence above, because the table holds internal pipeline processing records rather than reader-facing content.
- **Required follow-up (not yet done, needs a later separately-authorised phase):** the live RLS policy `processing_log_authenticated_read` predates this decision and must eventually be dropped for full hygiene. The proposed grant SQL (`Perspectives-worker/supabase/grants/20260821_PROPOSED_DATA_API_GRANTS.sql`) revokes the table-level grant, which is sufficient to make the table unreachable via the Data API immediately once that file is executed — independent of when the stale policy itself is removed.

### 6. V1 anonymous read access to reader-facing tables — RESOLVED: no anon reads

- **New conflict surfaced by MIG-001:** the live database's RLS policies on `articles`, `authors`, `journals`, `specialties`, `subtopics`, and `article_specialties` already restrict `SELECT` to `auth.role() = 'authenticated'` — meaning unauthenticated (`anon`) requests get zero rows today, contrary to what a "public discovery feed" reading of the product vision might have assumed.
- **Decision (Maz, during MIG-001):** intentional. V1 requires sign-in before any reader-facing content is visible. No anonymous browsing.
- **Clarification recorded:** this does not conflict with the settled architecture point that "the frontend uses the anon key only" — that describes which *type* of Supabase client key the frontend holds (never the service-role key), not whether unauthenticated requests can read data. A signed-in user's browser still uses the anon key plus their own session token; RLS evaluates them as `authenticated` regardless.
- **Consequence:** no RLS policy change is needed for these six tables — the live configuration already matches the decision. Only the redundant `anon` table-level grant needed removing, which the proposed grant SQL does.

## Context: how the live database resolved the schema-shape question

MIG-001's read-only capture (2026-08-27/28) confirmed the live database's 11-table `public` schema matches the **handover document's** shape exactly (standalone `specialties` and `content_fingerprints` tables, no `article_authors`/`article_tags` join tables) rather than the master document's shape. This was previously recorded as a user-confirmed starting fact pending formal verification; it is now formally verified. See `MIG001_SCHEMA_RECONCILIATION.md` and the full report in `Perspectives-worker` for details, including two facts neither document anticipated: authors relate to articles via a single `author_id` foreign key plus an unstructured JSON co-authors field (not a join table), and tags/subtopics relate to articles via plain array columns (also not a join table).
