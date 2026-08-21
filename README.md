# Base44-to-standalone

A reusable methodology and toolkit for migrating **any** Base44 no-code application to a standalone, self-hosted stack (frontend + database + backend worker).

This repository is **application-independent**. It is not a home for any single product's production code.

## What belongs here

- Generic templates, scripts, and validators for auditing a Base44 prototype and planning its migration.
- Documentation of the general migration methodology (see [`docs/MIGRATION_PLAYBOOK.md`](docs/MIGRATION_PLAYBOOK.md)).
- Sanitised, reference examples — currently **PERSPECTIVES**, tracked under [`examples/perspectives/`](examples/perspectives/), as the first real-world case this methodology was developed against.

## What does not belong here

- **Production PERSPECTIVES code.** The standalone frontend lives in [`Perspectives-app`](https://github.com/MazenKafienah/Perspectives-app); the worker and canonical database migrations live in [`Perspectives-worker`](https://github.com/MazenKafienah/Perspectives-worker).
- **Canonical PERSPECTIVES SQL.** This repository must never become a second canonical location for production schema. `Perspectives-worker/supabase/migrations/` is the only canonical copy.
- **Credentials, secrets, private user records, or production data of any kind.** Every example in this repository is sanitised. Reusable scripts and templates operate on synthetic fixtures, not real data.
- **Executable production files duplicated as "canonical copies."** If a script or SQL file is genuinely production code for an application, it is referenced or summarised here, not pasted in as a second source of truth.

## Repository map

See [`examples/perspectives/PROJECT_MAP.md`](examples/perspectives/PROJECT_MAP.md) for the full four-repository PERSPECTIVES workspace layout, ownership boundaries, and deployment targets.

## Status

MIG-000 governance and initial PERSPECTIVES reference documentation (project map, legacy inventory, migration ledger, open decisions). No generic scripts, templates, or validators have been written yet — those follow in later, separately authorised phases as reusable lessons are extracted from each PERSPECTIVES migration phase.
