---
name: data-and-migrations
description: Design persistent data and change it safely — schema and constraints, transaction boundaries, money/ledger writes, idempotency, schema migrations, backfills, and the order schema and code ship in. Use when adding or altering tables/columns, writing a migration or backfill, touching billing, balances, quotas or other money-like state, calling external services from code that also writes to the database, or planning a deploy that includes a schema change.
---

# Data and Migrations

Data outlives code. A bad function is fixed by the next deploy; a bad row, a lost charge, or a migration that half-ran stays until someone repairs it by hand. Default to Postgres when choosing freely; follow the project's existing database otherwise.

## 1. Schema encodes invariants

- Put invariants in the database where it can hold them: `NOT NULL`, foreign keys, `CHECK`, and unique constraints. A unique constraint is the cheapest correct idempotency mechanism there is.
- Size columns for the data, and test against the production engine. A 44-character dedup key in a `varchar(10)` rolled back a whole transaction and lost a bonus; the SQLite test fixtures never enforced the length. Use real Postgres in integration tests (testcontainers, PGlite) when production is Postgres.
- Defaults live in one place. Some ORMs re-issue `ALTER` on every start when a default is declared in a struct tag (GORM booleans); set those in code or in the migration, not both.
- Soft delete is a domain decision, not a free filter. A `deleted_at IS NULL` added to a billing `UPDATE` let in-flight requests skip charging after the key was deleted. For every query that gains the filter, ask what in-flight work it strands.

## 2. Transactions

- A transaction contains database work only. Never call an upstream HTTP API, payment provider, or message send inside one: it holds locks for network time, and a rollback cannot un-send.
- For "reserve, call out, settle" flows, use two transactions:
  1. tx1: lock rows, write a `PENDING` record and the reservation entries.
  2. Call the external service, outside any transaction.
  3. tx2: write back the result, or write a compensating entry.
  A crash between steps leaves a `PENDING` row a reconciler can finish. That is the point.
- Lock in a consistent order (e.g. `SELECT … FOR UPDATE` ordered by account id) to avoid deadlocks between concurrent transfers.
- Persist a result together with the state change it triggers, in the same transaction. Two writes that must agree are one transaction or one row.
- Side effects after commit (notifications, webhooks, emails) go out after the transaction, each channel retried independently, so one failing channel never blocks or duplicates another.

## 3. Money and ledgers

- Balances are derived from an append-only ledger, or kept in lockstep with it in the same transaction. Ledger and audit rows are never updated or deleted; corrections are new reversing entries. Enforce it with a trigger or permissions, not convention.
- Store amounts as integers in the smallest unit with the unit in the column or type name. Never mix credits and currency in one column.
- "No rows" and "zero" are different. Do not publish an unknown balance or empty ledger as `$0`.

## 4. Idempotency

Every operation that can be retried — webhook handlers, job steps, payment callbacks, migrations, backfills — answers two questions: what happens if it runs twice, and what happens if the previous run died halfway?

- Key retried work with an idempotency key under a unique constraint, and treat a duplicate-key error as "already done".
- Resumed or leased work carries a lease token; a stale worker cannot commit over a newer one.
- Cancelling a multi-step operation does not undo steps that already had external effects. Record each completed effect so compensation can run.

## 5. Migrations

- Migrations are forward-only, numbered files checked into the repo, applied by one tool. Never edit a migration that has run anywhere shared; some deploys verify checksums and will refuse.
- Separate schema migrations from data backfills. A backfill runs in bounded batches, is idempotent (safe to restart from the top), and logs progress.
- Migrations that touch the filesystem or app data directories take the target path as a parameter and run only from the app entry point. A test once ran a data-dir migration against the developer's real home directory.

### Ordering schema and code (expand / contract)

When old and new code run at the same time — rolling or blue-green deploys, multiple nodes, a frontend released separately — every step must be compatible with both:

1. **Expand**: add the new column/table, nullable or with a default. Old code ignores it.
2. Deploy code that writes both (or writes new, reads with fallback).
3. Backfill.
4. Deploy code that reads only the new.
5. **Contract**: drop the old column in a later release.

Renames and type changes are expand/contract, never a single `ALTER`. A breaking change that cannot be split needs a declared downtime window.

Know which nodes run migrations. When secondary or green nodes skip auto-migration, the schema change must be applied before they take traffic, or they serve requests against a schema they don't know about.

## 6. Operations

- Backups copy, never mirror: `rclone copy`, not `sync`, so a deletion upstream does not delete the history.
- Deploy configuration in the repo (`deploy/`) is the source of truth. A hot-fix made on a server is committed back the same day.
- Before changing infrastructure to fix a database symptom (dropping a view, demoting a node), check the driver and client versions; a driver bug once looked like a schema problem.
