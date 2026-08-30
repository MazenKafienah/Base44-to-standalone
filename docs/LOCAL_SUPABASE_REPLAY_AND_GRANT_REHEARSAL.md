# Local Supabase Replay and Grant Rehearsal — A Reusable Method

A follow-on to [`LIVE_SUPABASE_SCHEMA_CAPTURE.md`](LIVE_SUPABASE_SCHEMA_CAPTURE.md), covering stages 2 and 3 of that document's four-stage separation (capture → **baseline generation** → **replay validation** → production execution) in detail, plus how to safely rehearse an access-control change before anyone touches production. Application-independent; examples below use a synthetic `orders`/`customers` schema.

## Why replay validation earns its own step

A baseline generated mechanically from captured catalogue metadata (constraint definitions, index definitions, function bodies) is *probably* correct, because it was built from the live objects' own DDL-generating functions rather than transcribed by hand. "Probably correct" and "proven to run" are different claims. The gap between them is exactly what replay validation closes — and it tends to close cleanly, because the source material was already exact.

## Getting a faithful local environment

For a plain Postgres schema, a local Postgres instance is enough. For a Supabase schema specifically — one whose RLS policies call `auth.uid()` or `auth.role()`, or whose access model depends on the `anon`/`authenticated`/`service_role` roles — a plain Postgres instance is not faithful, because those roles and functions don't exist there. The right tool is the **official Supabase CLI's local development stack** (`supabase init` + `supabase start`), which provisions a real Postgres instance alongside GoTrue (auth), PostgREST, and the standard roles and helper functions, all on loopback, with no project reference or login needed for local-only use.

That stack needs a container runtime. If avoiding a GUI dependency matters, a CLI-only Docker-compatible runtime (at the time of writing, Colima) avoids the manual "launch Docker Desktop" step entirely, at the cost of one extra tool to install. Either way: **the container images and CLI are downloaded once, locally** — no login, no `supabase link`, no project reference, no production credential enters this step at all. State the exact tools you intend to install and why before installing them, and stop for explicit human action if the platform ever requires a manual GUI step partway through.

A well-chosen local environment often turns out to be *more* representative than expected: in one worked case, a freshly-provisioned local Supabase Postgres instance exhibited the identical default table-privilege behavior as the live project being investigated (broad grants to `anon`/`authenticated` on every new table) — confirming that behavior was a platform default, not a project-specific misconfiguration, purely as a side effect of using the real local stack instead of a generic Postgres container.

## Replaying the baseline

1. **Never modify the committed, guarded baseline file.** Generate a mechanical, deterministic, untracked local copy with only the execution guard removed (a straightforward text substitution — e.g. stripping an exact `DO $guard$ ... $guard$;` block), and record the SHA-256 of both the source and the generated copy so the relationship is auditable. Never stage or commit the generated copy.
2. **Apply it to a completely empty schema**, with the SQL client set to stop on first error (e.g. `psql -v ON_ERROR_STOP=1`). A clean pass with zero errors is the actual pass condition — don't treat warnings-only output as ambiguous; read every line.
3. **If it fails, classify why** before touching anything: a genuine baseline defect, an environment/extension mismatch, a Supabase-platform-specific dependency the local stack doesn't provide, an ordering problem, or a syntax issue introduced by the mechanical generation step. Fix forward in a new, superseding artifact — never edit history that already shipped.
4. **Re-run the identical capture query against the replayed local database.** Comparing live-captured metadata against locally-replayed metadata, object by object, is the actual proof the baseline is faithful — not just that it executed. Classify every difference (exact match; expected environment noise such as a patch-version bump or a platform-internal schema name; a genuine mismatch worth investigating) rather than silently accepting or silently hiding any of them.

## Rehearsing an access-control (grant) change

Schema replay and grant rehearsal are different exercises with different risks, and deserve separate treatment even though they often happen in the same session.

1. **Capture the pre-change privilege state first**, on the replayed local database, before touching anything. This is your rollback target and your baseline for comparison.
2. **Build the target access matrix from evidence, not assumption** — one row per (table, role) pair, citing exactly which verified fact (a live RLS policy, a confirmed product decision, a live-verified command list) justifies each privilege. Don't grant a command "to be safe" if nothing in the evidence calls for it, and don't omit one that the evidence clearly requires.
3. **Apply the target grant script to the local replica and diff the result against the target matrix**, not against a vague sense of "looks about right."
4. **Test effective access with real role-switching, not just catalogue inspection.** `information_schema` tells you what's *granted*; it doesn't tell you what a request actually experiences. Simulate the real request context: assume the role (`SET LOCAL ROLE ...`), set the session variables the platform's own auth-context functions read (for Supabase, `request.jwt.claims` as a JSON string containing at least `role` and `sub`), and then actually run the query. Do this inside an explicit transaction — outside one, a `SET LOCAL` silently no-ops and any conclusion drawn from the resulting query is worthless. (This is an easy mistake to make once and a very easy one to catch: if the "test" appears to run as an unrestricted superuser, it didn't set the role at all.)
5. **Demonstrate the grant-vs-policy distinction empirically if you can, not just describe it.** Provisioning a table with a permissive row-level policy already in place, then testing access *before* and *after* revoking the underlying table privilege — with the policy left completely untouched both times — is a small, cheap experiment that turns "grants and policies are different layers" from an assertion into a proven fact, with a receipt.
6. **Test idempotency**: apply the target script twice, confirm zero errors and byte-identical resulting state both times. A script that isn't idempotent is either not ready, or needs `IF EXISTS`/`IF NOT EXISTS`-style guards added before it's considered done.
7. **Test rollback in both directions**: target → rollback → confirm it matches the original pre-change state exactly; then rollback → target again → confirm deterministic restoration. A rollback script that "restores the old state" is not automatically safe to use casually — if the old state was itself over-permissive, say so loudly in the rollback file's own header, not just in a separate report someone might not read before running it.

## A clarification worth stating explicitly, because it's a common point of confusion

**The type of client key an application holds and the role a request executes as are two different things.** A frontend that only ever holds a public/anon-type API key (never the privileged service key) is not the same claim as "unauthenticated requests can read data." A signed-in user's request still presents the public key plus their own session token; the platform's row-level security then evaluates them under their authenticated role, not as anonymous. Don't let evidence that "the app only uses the public key" be read as evidence about whether sign-in is required — those are independent facts, and conflating them can lead to either an unintended public data exposure or an unnecessary sign-in requirement, depending on which way the confusion runs.

## Reusable checklist

- [ ] Confirm the target environment is genuinely local before running anything against it (loopback host, disposable/ephemeral database or container, no production credential reachable from the session).
- [ ] Never mutate the committed, guarded baseline or grant files — generate untracked, guard-stripped local copies only, and record source/generated hashes.
- [ ] Replay the baseline against an empty schema; treat a completely clean run as the pass bar.
- [ ] Re-capture and structurally diff live vs. local; classify every difference, hide none.
- [ ] Build the target grant matrix from cited evidence per row.
- [ ] Test effective access via real role/claims simulation inside an explicit transaction, not catalogue inspection alone.
- [ ] Test idempotency (apply twice, expect identical state) and rollback (both directions) before calling a grant change "reviewed."
- [ ] Keep a clear, separate, review-only plan for the eventual production execution — pre-requisites, sequence, and explicit abort conditions — distinct from the artifacts that were actually rehearsed locally.
