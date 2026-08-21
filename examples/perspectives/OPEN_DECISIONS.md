# PERSPECTIVES — Open Decisions

Preserved conflicts and unresolved questions carried forward from the planning documents. None of these are resolved by MIG-000. Each requires Maz's explicit confirmation before the affected code or SQL is written.

## 1. `article_processing_log` access level: authenticated-read or worker-only?

- One handover statement describes authenticated `SELECT` access on this table.
- The execution runbook's grant procedure treats it as worker-only (no `anon`/`authenticated` grant at all).
- **Current recommendation (not yet confirmed):** worker-only, because the table holds internal pipeline audit/processing records rather than reader-facing content.
- **Gate:** MIG-001 must reconstruct all Data API grants against the verified live schema; this table's grant must not be written either way until Maz confirms.

## 2. `runMonthIngestion` — port, replace, or deliberately drop?

- Verified to exist in the legacy prototype (`base44/functions/runMonthIngestion/`) but undocumented in the earlier stale audit.
- Behaviour: month-targeted ingestion, distinct from the regular bi-daily broad pipeline.
- Must not disappear silently — a decision to drop it must be a conscious one, recorded here once made, not an omission discovered later.

## 3. `pruneOldArticles` — port, replace, or deliberately drop?

- Verified to exist in the legacy prototype (`base44/functions/pruneOldArticles/`) but undocumented in the earlier stale audit.
- Behaviour: storage-hygiene pruning that must preserve saved/followed articles rather than deleting them indiscriminately.
- Any replacement must explicitly carry forward the saved-article protection requirement; must not disappear silently.

## 4. Exact current Anthropic API model identifiers

- Planning documents disagree on the summariser split: the handover specifies "Claude Sonnet 4.6" for extraction and "Claude Haiku 4.5" for writing; the master document's code bodies hardcode `claude-3-5-sonnet-20241022` and `claude-3-5-haiku-20241022`.
- The `claude-3-5-*` strings are known-stale and must not be copied into new code.
- **Gate:** confirm the exact current Anthropic API model identifiers against official Anthropic documentation at implementation time, not from any planning document in this workspace.

## Context: how the live database bears on these

Per the workspace owner's confirmation during MIG-000, a live Supabase project already exists containing 11 `public`-schema tables: `api_cache`, `article_processing_log`, `article_specialties`, `articles`, `authors`, `content_fingerprints`, `interaction_signals`, `journals`, `specialties`, `subtopics`, `user_preferences`. This table set matches the **handover document's** schema shape (standalone `specialties` and `content_fingerprints` tables, no `article_authors`/`article_tags` join tables) rather than the master document's schema shape (which used `article_authors`/`article_tags` joins and a `content_fingerprint` column on `articles` instead of a standalone table). This is recorded here as a **user-confirmed starting fact for MIG-000**, not as something inspected directly — MIG-000 explicitly does not connect to Supabase. MIG-001's read-only schema capture is what formally verifies it and is the point at which decision 1 above can actually be resolved against real table names.
