# PERSPECTIVES — Project Map

The PERSPECTIVES workspace consists of **four independent Git repositories**, each with a distinct role. No repository is a subdirectory or submodule of another; each has its own `.git` history and its own GitHub `origin`.

## Repository 1 — Legacy source (protected, read-only)

- **Local path:** `Perspectives_Prototype/`
- **GitHub:** `https://github.com/MazenKafienah/Perspectives_Prototype`
- **Role:** The existing Base44-connected prototype. Immutable migration source and behavioural reference. Home of the original PERSPECTIVES planning documents (historically; the current canonical documents now live in a sibling workspace folder, `Perspectives Migration Project/`, not inside this repository).
- **Not** the destination for any standalone-application code.
- **Protection rule:** No branch, commit, push, rename, or configuration change to this repository during migration phases unless a phase explicitly authorises it. As of MIG-000, it remains untouched.

## Repository 2 — Standalone frontend

- **Local path:** `Perspectives-app/`
- **GitHub:** `https://github.com/MazenKafienah/Perspectives-app`
- **Role:** New standalone Next.js (App Router) frontend. Reader-facing specialty feed, article/author pages, search, settings, auth.
- **Deployment target:** Vercel (later phase).
- **Data access:** Supabase Data API using the **anon key only**. Never the service-role key.

## Repository 3 — Worker and database migrations

- **Local path:** `Perspectives-worker/`
- **GitHub:** `https://github.com/MazenKafienah/Perspectives-worker`
- **Role:** Python discovery/enrichment/summarisation worker. Scheduled via GitHub Actions cron (target `0 6,18 * * *`), not a standing HTTP server.
- **Database ownership:** The **canonical, executable** location for PERSPECTIVES Supabase SQL migrations (`supabase/migrations/`). No other repository holds a second canonical copy.
- **Connection:** Direct PostgreSQL connection (`DATABASE_URL`, `postgres` role) — independent of Supabase Data API grants.
- **Secrets:** Holds the Supabase service-role key and worker-only API keys (Anthropic, OpenAI). Never shared with `Perspectives-app`.

## Repository 4 — General migration toolkit (this repository)

- **Local path:** `Base44-to-standalone/`
- **GitHub:** `https://github.com/MazenKafienah/Base44-to-standalone`
- **Role:** Application-independent methodology, templates, and validators for migrating any Base44 application. PERSPECTIVES is the first sanitised reference case, tracked under `examples/perspectives/`.
- **Never:** a second canonical location for PERSPECTIVES production SQL, and never a location for credentials, secrets, or real user/production data.

## Deployment boundaries

| Concern | Owning repository | Notes |
|---|---|---|
| Reader-facing web app | `Perspectives-app` | Deploys to Vercel |
| Ingestion/enrichment pipeline | `Perspectives-worker` | Runs via GitHub Actions cron |
| Canonical Supabase schema/migrations | `Perspectives-worker` | `supabase/migrations/` |
| Supabase project itself | External (Supabase, EU region) | Not a repository; inspected read-only starting MIG-001 |
| Generic migration methodology | `Base44-to-standalone` | No production code or secrets |
| Legacy Base44 prototype | `Perspectives_Prototype` | Read-only reference during migration |

## Database-migration ownership rule

`Perspectives-worker/supabase/migrations/` is the **only** canonical, executable location for PERSPECTIVES production SQL. Any schema documentation elsewhere (including in this toolkit) is a reference or summary, never a second source of truth, and must never be run directly against production.

## Local-to-remote correspondence

Each local path above tracks its own `origin/main` on its own GitHub repository — there is no shared remote and no cross-repository branch relationship. Work on one repository (e.g. a `bootstrap/mig-000` branch) has no effect on the git history of the other three; commit SHAs referenced across documents (see `MIGRATION_LEDGER.md`) are cross-references for traceability only, not shared history.
