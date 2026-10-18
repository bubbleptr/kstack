---
name: api-contract
description: Define and change the contract between a backend and its callers — HTTP/RPC endpoints, IPC and WebSocket messages, auth headers, error shapes, shared type packages. Use when adding or changing an endpoint or message, renaming a field or header, designing error responses, wiring auth checks, writing frontend mocks against a backend, or when a frontend and backend disagree about what a field means.
---

# API Contract

A contract is every fact a caller relies on: field names, units, meanings, error codes, headers, and ordering. Most of the production incidents this skill exists for were not crashes. They were two sides that both compiled and disagreed about meaning.

## 1. One source of truth for the shape

- The contract lives in exactly one place and both sides derive from it: a shared contract package in a monorepo (schema + inferred types), or a generated client from the server's schema. Hand-written parallel types on the client are where drift starts.
- The backend never imports from a frontend app. Shared shapes go in a kernel package both depend on.
- Every message on a multiplexed channel (IPC, WebSocket, event bus) carries an explicit `type` discriminator. Never infer the kind of a message from which fields happen to be present.
- Every persisted or cross-process envelope (events, manifests, job payloads, cache snapshots) carries a `schemaVersion` from the first version. Adding it later means guessing the version of everything already written.

## 2. Parse at the edge, trust inside

- Each handler parses the raw request into domain types once, at the boundary, and rejects with a structured error. Inside the application, types are trusted; no re-validation.
- Map domain results to the wire in the handler. Do not expose ORM rows or internal structs directly; the day the table changes, the API silently changes with it.
- Name units in the field or the type: `durationMs`, `amountCredits`, `paidCents`. A field called `use_time` was read as milliseconds while holding seconds; `amount` was read as money while holding credits. When a requested field name has no unit, propose the unit-bearing name; the product's wording belongs in the UI label, not the wire.

## 3. Errors are part of the contract

- Return a stable machine-readable code (`error_code` / `reason` enum) plus a human message. Callers branch on the code, never on message text or on the HTTP status alone. Mapping every 401 to "session expired" hid bad-credential errors from users.
- Sanitize errors where they leave the backend (or the proxy in front of it). Hiding internals in the frontend does not hide them from anyone calling the API directly.

## 4. Auth at the boundary, every time

- Authorization checks run before any side effect: before account selection, upstream forwarding, quota reservation, or billing. Inbound webhooks authenticate by provider signature over the raw body.
- Long-lived connections re-check authorization per turn/message, not only at connect.
- Anything that billing or authorization reads from a cache must be in the cache snapshot. When adding a field that affects billing or limits, add it to the snapshot and bump the snapshot version in the same change, otherwise cache hits use the old shape (one such miss billed zero).

## 5. Changing a contract

Decide first who the callers are:

- **Internal only, deployed together** (same monorepo, same release): change the contract, migrate every caller in the same change, delete the old shape. No compatibility layer.
- **Separately deployed** (frontend and backend released independently, mobile apps, multi-node rollouts): expand/contract — the server accepts old and new, clients move, then the old is removed once a metric shows nothing sends it. A one-shot header rename caused production-wide 401s because frontend and backend did not ship at the same instant. For headers, the CORS allow/expose lists and the proxy config in `deploy/` change in the same release as the server.
- **Public** (open-source CLI, third-party integrators): version it. SemVer for clients, a new endpoint or version for semantic changes.

Changing what an existing field or endpoint **means** is never an in-place edit for separately deployed callers. Add a new field or endpoint and retire the old one.

Watch partial updates: a `PUT` that replaces the whole resource silently resets any field the caller did not send. Use `PATCH` semantics, or make the full-replace explicit in the contract.

When a contract changes, update in the same PR whatever documents restate it (OpenAPI, architecture doc, user-facing API doc). A stale contract doc is worse than none.

## 6. Mocks and tests use real shapes

- Frontend mocks and fixtures are captured from, or typed against, real backend responses. A mock written from imagination encodes the misunderstanding the real API will contradict.
- Contract tests assert the wire shape a caller sees (field names, codes, units), through the real handler.
