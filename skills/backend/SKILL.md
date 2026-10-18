---
name: backend
description: "Shape a backend: the domain model (bounded contexts, domain logic versus transport and persistence, state ownership, CONTEXT.md vocabulary, ADRs), the contract with callers (HTTP/RPC, IPC and WebSocket messages, auth headers, error shapes, shared type packages), and persistent data (schema, transactions, money/ledger writes, idempotency, migrations, backfills, deploy order). Use when adding a feature with new concepts or state, splitting or merging modules/contexts, deciding which layer owns a rule, naming a domain term, writing CONTEXT.md or an ADR, adding or changing an endpoint, message, field or header, designing errors or auth checks, writing frontend mocks, frontend/backend disagreeing on a field, altering tables, writing a migration or backfill, touching billing, balances or quotas, calling external services from code that writes the DB, or planning a deploy with a schema change."
---

# Backend

The code and its tests are the only description of what the system does. This skill decides where domain knowledge lives in that code, what callers may rely on, and how persistent data changes safely. It also records the two things code cannot hold: the vocabulary (`CONTEXT.md`) and why a hard-to-reverse choice was made (ADRs).

Which part applies: new concepts, state, or module boundaries → Domain; anything a caller sees (endpoint, message, header, error) → Contracts; tables, transactions, money, migrations, deploys → Data. Most features touch all three.

## Domain

Apply DDD in proportion to the domain. A CRUD admin endpoint needs a handler and a query, not an aggregate. Reach for the structure below when rules, state transitions, or money are involved.

### Find the context before writing code

1. Read the repo's `CONTEXT.md` (one per context in multi-context repos) and the Accepted ADRs that touch the area. Also check what layout the repo actually uses — some repos keep decisions in `docs/decisions/` instead of `docs/adr/`. Follow the repo; create files lazily, only when there is something to write.
2. Name the bounded context the change belongs to. If the new concept's meaning differs between two parts of the system ("account" as login vs. as billing balance), those are two contexts with two terms, not one shared model.
3. Pick the owner of each piece of state. Exactly one module writes it; others read it by id or receive events. Framework storage (workflow engines, agent frameworks, caches) owns only its own execution state; business data stays in the domain's own tables.

Done when you can say, for every new piece of state: which context owns it, which module writes it, and how other contexts learn about it.

### Layering and types

Dependencies point inward: transport → application → domain ← infrastructure. Map this to the stack's own names (`handler/ service/ repository/ domain/` in Go; a domain package plus a thin server app in a TS monorepo) instead of forcing folders. The test for a misplaced rule: can it be unit-tested without starting the server or the database? If not, move it inward.

Three modelling choices that do not happen by default:

- Brand identifiers and units (`UserId`, `Credits`, `Cents`, `Millis`), and name the unit wherever the value crosses a boundary: in the wire field (`durationMs`, `amountCredits`, `paidCents`) and the column (`amount_credits`, `paid_cents`). Real bugs came from confusing credits with paid money and seconds with milliseconds: a field called `use_time` was read as milliseconds while holding seconds; `amount` was read as money while holding credits. The type system catches that, review does not reliably. Store integer amounts in the smallest unit; one column never mixes credits and currency. When a requested field name has no unit, propose the unit-bearing name; the product's wording belongs in the UI label, not the wire.
- Make "unknown" a distinct state. "No rows" is not zero: an empty ledger that means "not loaded yet" or "unknown" must not render or be published as `$0`.
- Name the capability, not the vendor: `BrowserContext`, not `Chrome`; `PaymentProvider`, not `Stripe`. The vendor is an adapter.

### Between contexts

- Reference other contexts by id, never by importing their entities.
- Shared types live in an explicit shared kernel package (ids, money, event envelopes, contract shapes). A backend never imports from a frontend app; both import from the kernel.
- Prefer events for "something happened" and direct calls for "I need an answer now". Cancelling a multi-step operation does not undo steps that already had external effects; record each completed effect so compensation can run idempotently.
- The product owns its contracts. When integrating a plugin system, SDK, or upstream, adapt it to your domain model at the edge instead of reshaping the domain to match it.

### CONTEXT.md — vocabulary only

Business language for one context. Format per term:

```md
**Term**:
One or two sentences: what it is in this domain.
_Avoid_: synonyms that must not be used
```

- A definition says what the thing **is**. Behaviour (what the UI shows, which button does what, edge-case handling) belongs in code and tests; once a definition grows into behaviour it is a spec, and it will drift.
- Implementation words stay out, even when ADRs use them.
- Update it in the same change that introduces or renames the term. A term that exists only in someone's memory or chat does not exist for the next agent.

### ADRs — decisions, not specs

Write one only when all three hold: hard to reverse, surprising without context, and a real trade-off between alternatives. Write it when the decision is made, not as pre-implementation planning.

Follow the repo's existing ADR language and headings. For a new repo, read [`references/adr-template.md`](references/adr-template.md) when writing the first ADR.

- An ADR is history: dated, with a status (`Accepted`, `Superseded by ADR-NNNN`). It is allowed to describe a world that no longer exists. Treat only Accepted ADRs as constraints.
- Keep implementation detail out — file paths, config keys, and step lists belong in the PR. Keep the decision, the rejected alternatives, and the consequences someone would otherwise "fix".
- To change a decision, write a new ADR that supersedes the old one and flip the old status. Do not rewrite history.

## Contracts

A contract is every fact a caller relies on: field names, units, meanings, error codes, headers, and ordering. Most of the production incidents this section exists for were not crashes. They were two sides that both compiled and disagreed about meaning.

### One source of truth for the shape

- The contract lives in exactly one place and both sides derive from it: a shared contract package in a monorepo (schema + inferred types), or a generated client from the server's schema. Hand-written parallel types on the client are where drift starts.
- Every message on a multiplexed channel (IPC, WebSocket, event bus) carries an explicit `type` discriminator. Never infer the kind of a message from which fields happen to be present.
- Every persisted or cross-process envelope (events, manifests, job payloads, cache snapshots) carries a `schemaVersion` from the first version. Adding it later means guessing the version of everything already written.

### Parse at the edge, trust inside

- Each handler parses the raw request into domain types once, at the boundary, and rejects with a structured error. Inside the application, types are trusted; no re-validation.
- Map domain results to the wire in the handler. Do not expose ORM rows or internal structs directly; the day the table changes, the API silently changes with it.

### Errors are part of the contract

- Return a stable machine-readable code (`error_code` / `reason` enum) plus a human message. Callers branch on the code, never on message text or on the HTTP status alone. Mapping every 401 to "session expired" hid bad-credential errors from users.
- Sanitize errors where they leave the backend (or the proxy in front of it). Hiding internals in the frontend does not hide them from anyone calling the API directly.

### Auth at the boundary, every time

- Authorization checks run before any side effect: before account selection, upstream forwarding, quota reservation, or billing. Inbound webhooks authenticate by provider signature over the raw body.
- Long-lived connections re-check authorization per turn/message, not only at connect.
- Anything that billing or authorization reads from a cache must be in the cache snapshot. When adding a field that affects billing or limits, add it to the snapshot and bump the snapshot version in the same change, otherwise cache hits use the old shape (one such miss billed zero).

### Changing a contract

Read [`references/contract-changes.md`](references/contract-changes.md) when renaming, removing, or changing the meaning of a field, header, endpoint, or message, or switching between full-replace and partial updates. It decides between same-change migration, expand/contract, and versioning by who the callers are.

### Mocks and tests use real shapes

- Frontend mocks and fixtures are captured from, or typed against, real backend responses. A mock written from imagination encodes the misunderstanding the real API will contradict.
- Contract tests assert the wire shape a caller sees (field names, codes, units), through the real handler.

## Data

Data outlives code: a bad row, a lost charge, or a half-run migration stays until someone repairs it by hand. Default to Postgres when choosing freely; follow the project's existing database otherwise. The standard moves — constraints for invariants, unique keys for idempotency, expand/contract for schema changes — are assumed. This section holds the choices and traps specific to how these systems have actually failed.

### Schema traps

- **Test on the production engine.** A 44-character dedup key in a `varchar(10)` rolled back a whole transaction and lost a bonus; the SQLite fixtures never enforced the length. When production is Postgres, integration tests run on Postgres (testcontainers, PGlite).
- **Soft delete is a domain decision.** A `deleted_at IS NULL` added to a billing `UPDATE` let in-flight requests skip charging after their key was deleted. For every query that gains the filter, ask what in-flight work it strands.
- **One owner per default.** GORM re-issues `ALTER` on every start for a boolean `default:` tag. Set a default in the migration or in code, not in both.
- **One tool runs migrations.** When an ORM generates schema (Ent, Drizzle, GORM), it produces the numbered SQL file; the numbered file is what runs in every environment. ORM auto-migrate stays off in shared environments, or is limited to the one node that is documented to run it.

### Transactions, side effects, retries

- When a write must wrap an external call, the house shape is tx1 (`PENDING` + reservation) → call → tx2 (settle or compensate). The `PENDING` row is what a reconciler finishes after a crash.
- One outbox row per notification channel, retried independently, so a failing email never blocks or duplicates the push.
- Multi-account writes lock with `FOR UPDATE` ordered by account id.
- Resumed or leased work carries a lease token, so a stale worker cannot commit over a newer one.
- Cancellation and compensation: see Domain › Between contexts.

### Money and ledgers

- Balances come from an append-only ledger, or are updated in lockstep with it in the same transaction. Ledger and audit tables reject `UPDATE`/`DELETE` with a trigger or permissions; corrections are reversing entries.
- Units and "no rows is not zero": see Domain › Layering and types.

### Migrations and rollout

- Numbered, forward-only, never edited after running anywhere shared — deploys may verify checksums.
- Backfills are separate from schema migrations, run in bounded batches, and can restart from the top.
- Filesystem or data-dir migrations take the target path as a parameter and run only from the app entry point. A test once ran one against the developer's real home directory.
- **Know which nodes migrate.** When only the primary runs migrations, applying the migration is an explicit deploy step, verified (schema version or table check) before any other node takes traffic. Breaking changes that cannot be split into expand/contract need a declared downtime window.

### Operations

- Backups copy, never mirror: `rclone copy`, not `sync`, so an upstream deletion doesn't erase history.
- `deploy/` in the repo is the source of truth for server config. A hot-fix made on a server is committed back the same day.
- Before changing infrastructure to fix a database symptom (dropping a view, demoting a node), check driver and client versions. A pg driver bug once looked like a schema problem.
