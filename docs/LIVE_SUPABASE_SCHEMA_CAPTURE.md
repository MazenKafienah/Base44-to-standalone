# Live Supabase Schema Capture — A Reusable, Credential-Free Method

This describes a general method for reconciling a live Supabase (or any Postgres) database's actual schema against planning documents, **without ever giving a coding agent a database credential of any kind**. It is application-independent — the walkthrough below uses a synthetic `widgets`/`orders` example, not any real project's schema.

## Why credentials never go to the coding agent

A coding agent with a live database connection can, by design or by mistake, run something state-changing — and a database credential pasted into a chat session is a credential that has left your control the moment it's typed. Neither risk is worth taking for what is fundamentally a read task. This method sidesteps both: the agent never holds a connection string, an API key, or a password, and it never runs anything against the live database at all. A human runs one query, by hand, in the database's own SQL console, and hands the agent only the exported *result* — schema metadata, not credentials, not a live connection.

This also means the agent must never go looking for a credential as a workaround — not in a CLI's credential store, not in Keychain (or your OS's equivalent), not in environment variables, not in a `.env` file, not in shell history. If a required tool or authentication isn't already set up, the correct response is to stop and say so, not to probe for another way in.

## The four separated stages

Keep these distinct. Collapsing them is how a "just capture the schema" task turns into an accidental production change.

1. **Capture** — read-only. A single catalogue-only query, run by a human, against the live database. Produces metadata (table/column/constraint/index/policy/grant definitions), never application rows.
2. **Baseline generation** — offline. The agent turns the captured metadata into a DDL reconstruction and a normalized inventory, entirely from the exported file, with no further database contact. The baseline is a *reconstruction*, not a *replay-proven* script, until stage 3 happens.
3. **Replay validation** — a separate, later, explicitly authorised step. The baseline is executed against a disposable local or throwaway database (never the live one) to prove it actually runs end-to-end. Only after this should anyone trust the baseline as "this DDL definitely works."
4. **Production execution** — a separate, later, explicitly authorised step again. Even a replay-validated baseline or a well-reviewed grant script should not be applied to the live database in the same breath as capture-and-reconcile. Get a second look, understand the blast radius, and execute deliberately.

A capture-and-reconcile task should only ever touch stage 1 and 2. If a plan starts drifting into "let's just also run this against production while we're here," that's the moment to stop and get separate, explicit authorisation for that specific step.

## Step-by-step

### 1. Write the capture query (agent)

A single `SELECT` (with `WITH` CTEs as needed), reading only `pg_catalog` and `information_schema`, that:

- Never reads a row of application data — only reads about tables, not through them.
- Never mutates anything — no `INSERT/UPDATE/DELETE/ALTER/DROP/CREATE/GRANT/REVOKE` etc. anywhere in it.
- Returns one deterministic JSON object in one row, with every array sorted (e.g. `jsonb_agg(... ORDER BY ...)`), so two captures of an unchanged schema produce byte-identical output.
- Uses only `STABLE` catalogue-inspection functions (`pg_get_constraintdef`, `pg_get_functiondef`, `pg_get_triggerdef`, `has_table_privilege`, `has_schema_privilege`, `version()`, and similar) — never a `VOLATILE` function, and never one that touches or invokes application logic.
- Covers, where present: extensions, schemas, tables (with owner/comment/RLS flags), columns (types, nullability, defaults, identity, generated-column expressions), sequences and their ownership, enums, domains, constraints (primary key/unique/foreign key/check/exclusion, with full definitions), indexes, views and materialized views, functions (signature, language, volatility, security-definer status, full definition), triggers, RLS policies, and table/column/routine/schema-level grants for the roles that actually matter (the API-facing roles and any application role).

Before handing this query to a human to run, the agent should lexically re-check it for exactly the properties above — no mutating keyword outside a comment, no application-table `FROM`, a single deterministic result.

### 2. Run it (human, manually)

The human opens the database's own SQL console (Supabase's SQL Editor, or equivalent), pastes the query, eyeballs it for anything mutating, runs it once, and exports the single result row as CSV or JSON to a file **outside any Git repository**. No credential is ever typed into the coding session at this step.

### 3. Ingest and validate (agent)

Given the file path, the agent:

- Confirms the file exists at the stated path and computes its checksum (e.g. SHA-256) for provenance — recorded in every downstream artifact, never skipped.
- Parses the structure without dumping the complete raw contents into the conversation — inspect programmatically, report counts and key facts, not a wall of raw JSON.
- Confirms it looks like schema metadata (table/column/constraint arrays), not application records (no obvious user data, no row-count-sized data dumps).
- Confirms every section the capture query was supposed to produce is actually present.
- Runs a **bounded secret scan** specifically over free-text fields that could theoretically embed something sensitive — function bodies, policy expressions, column defaults, comments, view/trigger definitions. If anything matches a credential-shaped pattern, the agent does not print the match, does not commit the surrounding text, and instead reports only the object name and the suspected category, then asks the human what to do. Redacting silently and calling the result "exact" is not an option — either it's flagged for a decision, or it's clean.

### 4. Reconcile against documents (agent)

For every place the live capture and a planning document disagree, state four things together: what the live database actually shows, what each document claims, what the practical consequence of the mismatch is, and what the agent recommends — then let the live database win on *factual, already-existing* state, while leaving *not-yet-implemented product decisions* for a human to actually decide. "The database already works this way" is not the same claim as "the database should keep working this way" — don't let evidence about the former quietly settle the latter.

### 5. Generate the baseline and any change proposal (agent), guarded

Both the DDL baseline and any grant/permission change proposal should:

- Say plainly, at the top, what they are (a captured snapshot; a proposal) and what they are not (something that's been replay-tested; something that's been executed).
- Carry an execution guard — e.g. a `DO $$ BEGIN RAISE EXCEPTION ...; END; $$;` block at the very top — that aborts on an accidental run. The guard is removed only in the dedicated replay-validation or execution stage, by explicit later decision, never automatically.
- Separate schema shape from access grants into different files. They change for different reasons and get reviewed by different people at different times.

### Synthetic worked example

Say a capture reveals a live `orders` table with a `status` column whose `CHECK` constraint allows `'pending'`, `'shipped'`, `'cancelled'` — but the design doc only ever mentions `'pending'` and `'shipped'`. That's a live-vs-document conflict worth surfacing exactly as above: live fact (three allowed statuses), document claim (two), consequence (code written only against the doc's two values will reject or mishandle `'cancelled'` orders), and a recommendation (update the doc, or confirm `'cancelled'` was an intentional later addition) — not a silent pick of one over the other.

## What this method deliberately does not cover

Choosing *how* to fix a discovered conflict, deciding whether a proposed grant change should actually be executed, and picking a replay-validation environment are all judgement calls for the humans involved, informed by this method's output — not something the method itself resolves.
