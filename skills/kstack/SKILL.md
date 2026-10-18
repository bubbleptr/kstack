---
name: kstack
description: Ask which kstack skill or sequence fits the situation. A router over the kstack skills.
disable-model-invocation: true
---

# kstack

A map of the kstack skills and the order they run in. Read a skill's SKILL.md before stating what it does or recommending a step be skipped; the summaries here are for orientation only.

## Flow: idea → merged → live

1. **`design`**: a concrete sketch (usage, shapes, one alternative, ≤3 questions) before code. Skip for small, obvious changes.
2. Build, pulling in the domain skills below as the work touches them.
3. **`review`**: standards + ADR/intent axes, with `ui-review` as a third section when UI changed. Also triages dureview findings.
4. **`ship`**: commits, PR body, stacks, merge, cleanup.
5. **`release`**: version, tag, artifacts, deploy, verify what is live.

Side entries:

- Something broken, slow, or flaky → **`diagnose`**, then back to step 3.
- Work moves to another session, subagent, or machine → **`handoff`** (handoff doc, dispatch brief, verifying "done", resuming).

## Backend

- **`domain-design`**: bounded contexts, state ownership, layering, `CONTEXT.md` vocabulary, ADRs.
- **`api-contract`**: endpoints, messages, error codes, auth at the boundary, changing a contract safely.
- **`data-and-migrations`**: schema traps, transactions around external calls, ledgers, migrations and rollout order.

## Frontend (condensed from emil)

- **`ui-design`**: hierarchy, type, color, surfaces, icons, marketing pages, the visual unslop pass.
- **`ui-motion`**: whether and how to animate.
- **`ui-engineering`**: polish details, layout stability, rendering and loading performance.
- **`ui-interaction`**: forms, touch, keyboard, screen readers, mobile web.
- **`component-api`**: reusable component props and design-system docs.
- **`prototype`**: several real variants behind a live picker when the direction is undecided.
- **`ui-review`**: judge a UI diff, or sweep the whole app before launch.

## Cross-cutting

- **`unslop`**: strip AI tells from code and prose.
- **`write-skill`**: create or change a kstack skill.

## Overlaps

| Torn between | Ask | Then |
|---|---|---|
| `design` / `prototype` | Can the decision be made by reading a sketch, or does it need to be seen or run? | Read → design. Seen → prototype. |
| `design` / `domain-design` | Is the question how to build this feature, or where knowledge and state live? | Feature → design (it pulls in domain-design for ADRs). Placement → domain-design. |
| `review` / `ui-review` | Any diff, or interface craft? | Any diff → review, which calls ui-review for UI. Whole-app pre-launch → ui-review sweep mode. |
| `ui-design` / `ui-engineering` | Is it the wrong look, or right but unfinished/janky? | Look → ui-design. Unfinished → ui-engineering. |
| `ui-motion` / `ui-engineering` | Does the motion look wrong, or drop frames? | Looks wrong → ui-motion. Frames, lists, loading → ui-engineering. |
| `ui-interaction` / `ui-engineering` | Is someone locked out or does it feel wrong on a phone, or is it merely unpolished? | Locked out / phone → ui-interaction. Unpolished → ui-engineering. |
| `ship` / `release` | Getting it onto main, or getting it to users? | Main → ship. Users → release. |
| `api-contract` / `data-and-migrations` | Does the change cross a process boundary, or persist? | Wire → api-contract. Storage → data-and-migrations. Often both, contract first. |

## Not here

Discipline gates (design first, TDD, root cause, evidence) live in `~/.claude/CLAUDE.md` and apply everywhere; no skill restates them. Upstream skills still installed separately (for example matt's `tdd`, `research`, `triage`) are not part of kstack.
