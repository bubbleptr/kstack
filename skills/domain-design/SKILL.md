---
name: domain-design
description: Shape a backend's domain model — bounded contexts, where domain logic lives versus transport and persistence, state ownership, and the two documents that record it (CONTEXT.md vocabulary and ADRs). Use when adding a backend feature that introduces new concepts or state, splitting or merging modules/contexts, deciding which layer or package owns a rule, naming a domain term, or writing or editing CONTEXT.md or an ADR.
---

# Domain Design

The code and its tests are the only description of what the system does. This skill decides where domain knowledge lives in that code, and records the two things code cannot hold: the vocabulary (`CONTEXT.md`) and why a hard-to-reverse choice was made (ADRs).

Apply DDD in proportion to the domain. A CRUD admin endpoint needs a handler and a query, not an aggregate. Reach for the structure below when rules, state transitions, or money are involved.

## 1. Find the context before writing code

1. Read the repo's `CONTEXT.md` (one per context in multi-context repos) and the Accepted ADRs that touch the area. Also check what layout the repo actually uses — some repos keep decisions in `docs/decisions/` instead of `docs/adr/`. Follow the repo; create files lazily, only when there is something to write.
2. Name the bounded context the change belongs to. If the new concept's meaning differs between two parts of the system ("account" as login vs. as billing balance), those are two contexts with two terms, not one shared model.
3. Pick the owner of each piece of state. Exactly one module writes it; others read it by id or receive events. Framework storage (workflow engines, agent frameworks, caches) owns only its own execution state; business data stays in the domain's own tables.

Done when you can say, for every new piece of state: which context owns it, which module writes it, and how other contexts learn about it.

## 2. Layering and types

Dependencies point inward: transport → application → domain ← infrastructure. Map this to the stack's own names (`handler/ service/ repository/ domain/` in Go; a domain package plus a thin server app in a TS monorepo) instead of forcing folders. The test for a misplaced rule: can it be unit-tested without starting the server or the database? If not, move it inward.

Three modelling choices that do not happen by default:

- Brand identifiers and units (`UserId`, `Credits`, `Cents`, `Millis`). Real bugs came from confusing credits with paid money and seconds with milliseconds; the type system catches that, review does not reliably.
- Make "unknown" a distinct state. An empty ledger that means "not loaded yet" must not render as zero.
- Name the capability, not the vendor: `BrowserContext`, not `Chrome`; `PaymentProvider`, not `Stripe`. The vendor is an adapter.

## 3. Between contexts

- Reference other contexts by id, never by importing their entities.
- Shared types live in an explicit shared kernel package (ids, money, event envelopes). A backend never imports from a frontend app; both import from the kernel.
- Prefer events for "something happened" and direct calls for "I need an answer now". A cancellation does not undo side effects already sent; record them and run idempotent compensation.
- The product owns its contracts. When integrating a plugin system, SDK, or upstream, adapt it to your domain model at the edge instead of reshaping the domain to match it.

## 4. CONTEXT.md — vocabulary only

Business language for one context. Format per term:

```md
**Term**:
One or two sentences: what it is in this domain.
_Avoid_: synonyms that must not be used
```

- A definition says what the thing **is**. Behaviour (what the UI shows, which button does what, edge-case handling) belongs in code and tests; once a definition grows into behaviour it is a spec, and it will drift.
- Implementation words stay out, even when ADRs use them.
- Update it in the same change that introduces or renames the term. A term that exists only in someone's memory or chat does not exist for the next agent.

## 5. ADRs — decisions, not specs

Write one only when all three hold: hard to reverse, surprising without context, and a real trade-off between alternatives. Write it when the decision is made, not as pre-implementation planning.

Follow the repo's existing ADR language and headings. For a new repo, use [`references/adr-template.md`](references/adr-template.md).

- An ADR is history: dated, with a status (`Accepted`, `Superseded by ADR-NNNN`). It is allowed to describe a world that no longer exists. Treat only Accepted ADRs as constraints.
- Keep implementation detail out — file paths, config keys, and step lists belong in the PR. Keep the decision, the rejected alternatives, and the consequences someone would otherwise "fix".
- To change a decision, write a new ADR that supersedes the old one and flip the old status. Do not rewrite history.
