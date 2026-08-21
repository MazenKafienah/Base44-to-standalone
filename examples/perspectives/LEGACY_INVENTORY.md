# PERSPECTIVES — Legacy Prototype Inventory (Verified)

All facts below were verified directly against the `Perspectives_Prototype` repository working tree during MIG-000 (2026-08-21). Where a fact differs from an earlier planning-document claim, the discrepancy is recorded explicitly rather than silently forcing the older number. No private data, credentials, or secrets are recorded here.

## Stack

- **Build tool:** Vite 6
- **Framework:** React 18, single-page application (`react-router-dom` 6)
- **Package name:** `base44-app` (from `package.json`)
- **Styling:** Tailwind CSS 3.4 + shadcn/ui (Radix primitives)
- **Data fetching / forms:** TanStack Query 5, react-hook-form + zod
- **Other notable deps:** framer-motion, recharts, lucide-react, sonner
- **Auth:** `requiresAuth: false` confirmed at `src/api/base44Client.js:12`
- **Backend:** Deno serverless functions via `@base44/sdk`, Base44-managed Postgres

## Entities — verified: 7 (matches prior audit)

Source: `base44/entities/*.jsonc`

1. `Article.jsonc`
2. `Author.jsonc`
3. `InteractionSignal.jsonc`
4. `Journal.jsonc`
5. `Subtopic.jsonc`
6. `User.jsonc`
7. `UserPreferences.jsonc`

## Backend (Deno) functions — verified: **10**, not 9

Source: `base44/functions/*/`

1. `backfillAuthorPhotos`
2. `enrichAuthor`
3. `findAuthorPhoto`
4. `pruneOldArticles`
5. `refreshTopJournals`
6. `runDeepIngestion`
7. `runIngestionPipeline`
8. `runMonthIngestion`
9. `seedSubtopics`
10. `suggestSubtopic`

**Discrepancy:** the MIG-000 authorisation's "verified nine functions" list (items 1–8 and 10 above) omits `seedSubtopics` (item 9). The repository actually contains 10 functions. `seedSubtopics` is a one-shot taxonomy-seeding script already mapped in the master planning doc to `scripts/seed_subtopics.py`; it does not carry the same "port-or-drop" ambiguity as `runMonthIngestion` or `pruneOldArticles` (see `OPEN_DECISIONS.md`), but the count itself should be corrected wherever "9 functions" is cited.

## Frontend source footprint

- **Custom components** (`src/components/*.jsx`, excluding the `ui/` subdirectory): **27**, not the 22 previously cited. Verified list includes `AdminAccessCard`, `ArticleTypeChips`, `EditorArticleTable`, `EditorCustomRun`, and `MonthPicker` in addition to the 22 covered by the earlier audit — these five appear related to an admin/editor surface not previously inventoried.
- **shadcn/ui primitives** (`src/components/ui/*`): **49** files, not the 45 previously cited.
- `AuthContext.jsx`, `base44Client.js` present as documented.
- Root config: Vite, Tailwind, ESLint, `jsconfig.json`, `components.json` all present as documented.

## Two behaviours requiring an explicit port-or-drop decision

Neither may disappear silently during migration (see `OPEN_DECISIONS.md`):

1. **`runMonthIngestion`** — month-targeted ingestion mode, distinct from the regular bi-daily broad pipeline.
2. **`pruneOldArticles`** — storage-hygiene pruning that must protect saved/followed articles from deletion.

## Verification method note

Counts were obtained by direct filesystem enumeration of the repository working tree (`find`/`ls` over `base44/entities`, `base44/functions`, `src/components`, `src/components/ui`) and by grep of `package.json` and `src/api/base44Client.js`, not by re-reading planning documents. This inventory should be re-verified if the prototype repository changes before later migration phases begin.
