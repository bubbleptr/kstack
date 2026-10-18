# Changing a contract

Decide first who the callers are:

- **Internal only, deployed together** (same monorepo, same release): change the contract, migrate every caller in the same change, delete the old shape. No compatibility layer.
- **Separately deployed** (frontend and backend released independently, mobile apps, multi-node rollouts): expand/contract — the server accepts old and new, clients move, then the old is removed once a metric shows nothing sends it. A one-shot header rename caused production-wide 401s because frontend and backend did not ship at the same instant. For headers, the CORS allow/expose lists and the proxy config in `deploy/` change in the same release as the server.
- **Public** (open-source CLI, third-party integrators): version it. SemVer for clients, a new endpoint or version for semantic changes.

Changing what an existing field or endpoint **means** is never an in-place edit for separately deployed callers. Add a new field or endpoint and retire the old one.

Watch partial updates: a `PUT` that replaces the whole resource silently resets any field the caller did not send. Use `PATCH` semantics, or make the full-replace explicit in the contract.

When a contract changes, update in the same PR whatever documents restate it (OpenAPI, architecture doc, user-facing API doc). A stale contract doc is worse than none.
