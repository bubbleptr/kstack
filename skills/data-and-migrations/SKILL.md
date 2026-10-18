---
name: data-and-migrations
description: Design persistent data and change it safely — schema and constraints, transaction boundaries, money/ledger writes, idempotency, schema migrations, backfills, and the order schema and code ship in. Use when adding or altering tables/columns, writing a migration or backfill, touching billing, balances, quotas or other money-like state, calling external services from code that also writes to the database, or planning a deploy that includes a schema change.
---

# Data and Migrations

Data outlives code: a bad row, a lost charge, or a half-run migration stays until someone repairs it by hand. Default to Postgres when choosing freely; follow the project's existing database otherwise.

The standard moves — constraints for invariants, unique keys for idempotency, expand/contract for schema changes — are assumed. This skill holds the choices and traps that are specific to how these systems have actually failed.

## Schema traps

- **Test on the production engine.** A 44-character dedup key in a `varchar(10)` rolled back a whole transaction and lost a bonus; the SQLite fixtures never enforced the length. When production is Postgres, integration tests run on Postgres (testcontainers, PGlite).
- **Soft delete is a domain decision.** A `deleted_at IS NULL` added to a billing `UPDATE` let in-flight requests skip charging after their key was deleted. For every query that gains the filter, ask what in-flight work it strands.
- **One owner per default.** GORM re-issues `ALTER` on every start for a boolean `default:` tag. Set a default in the migration or in code, not in both.
- **One tool runs migrations.** When an ORM generates schema (Ent, Drizzle, GORM), it produces the numbered SQL file; the numbered file is what runs in every environment. ORM auto-migrate stays off in shared environments, or is limited to the one node that is documented to run it.

## Transactions and side effects

- When a write must wrap an external call, the house shape is tx1 (`PENDING` + reservation) → call → tx2 (settle or compensate). The `PENDING` row is what a reconciler finishes after a crash.
- One outbox row per notification channel, retried independently, so a failing email never blocks or duplicates the push.
- Multi-account writes lock with `FOR UPDATE` ordered by account id.

## Money and ledgers

- Balances come from an append-only ledger, or are updated in lockstep with it in the same transaction. Ledger and audit tables reject `UPDATE`/`DELETE` with a trigger or permissions; corrections are reversing entries.
- Integer amounts in the smallest unit, with the unit in the column name (`amount_credits`, `paid_cents`). One column never mixes credits and currency.
- "No rows" is not zero. An empty ledger that means "not loaded" or "unknown" must not be published as `$0`.

## Retries and resumption

- Resumed or leased work carries a lease token, so a stale worker cannot commit over a newer one.
- Cancelling a multi-step operation does not undo steps that already had external effects. Record each completed effect so compensation can run idempotently.

## Migrations and rollout

- Numbered, forward-only, never edited after running anywhere shared — deploys may verify checksums.
- Backfills are separate from schema migrations, run in bounded batches, and can restart from the top.
- Filesystem or data-dir migrations take the target path as a parameter and run only from the app entry point. A test once ran one against the developer's real home directory.
- **Know which nodes migrate.** When only the primary runs migrations, applying the migration is an explicit deploy step, verified (schema version or table check) before any other node takes traffic. Breaking changes that cannot be split into expand/contract need a declared downtime window.

## Operations

- Backups copy, never mirror: `rclone copy`, not `sync`, so an upstream deletion doesn't erase history.
- `deploy/` in the repo is the source of truth for server config. A hot-fix made on a server is committed back the same day.
- Before changing infrastructure to fix a database symptom (dropping a view, demoting a node), check driver and client versions. A pg driver bug once looked like a schema problem.
