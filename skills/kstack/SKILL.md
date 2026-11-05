---
name: kstack
description: Ask which kstack skill or sequence fits the situation. A router over the kstack skills.
disable-model-invocation: true
---

# kstack

A map of the kstack skills and the order they run in. Read a skill's SKILL.md before stating what it does or recommending a step be skipped; the summaries here are for orientation only.

## Setup (once per repo)

- **`setup`** (manual): write the GitHub issue-tracker conventions, add the `## Agent skills` pointer to the root rule file, and create the five labels. Run before the first issue or milestone; GitHub only.

## Flow: idea → merged → live

1. **`design`**: a concrete sketch (usage, shapes, one alternative, ≤3 questions) before code; large or layout work goes on a commentable Doc or canvas. Skip for small, obvious changes.
2. Build, pulling in the domain skills below as the work touches them.
3. **`review`**: standards + ADR/intent axes, plus the UI pass when the interface changed. Also triages dureview findings and runs the pre-launch UI sweep.
4. **`ship`**: commits, PR body, stacks, merge and cleanup, then release scope, tags, artifacts, deploy, and verifying what is live.

Side entries:

- Something broken, slow, or flaky → **`diagnose`**, then back to step 3.
- Work moves to another session, subagent, or machine → **`handoff`** (handoff doc, dispatch brief, verifying "done", resuming).

## Backend

- **`backend`**: domain (contexts, state ownership, layering, `CONTEXT.md`, ADRs), contracts (single source, error codes, auth at the boundary, changing a contract), data (schema traps, transactions around external calls, ledgers, migrations and rollout).

## Frontend (condensed from emil)

- **`ui-design`**: hierarchy, type, color, surfaces, icons, marketing pages, the visual unslop pass.
- **`ui-motion`**: whether and how to animate.
- **`ui-build`**: polish details, layout stability, states and focus, forms, touch/keyboard/screen readers, mobile web, rendering and loading performance.
- **`component-api`**: reusable component props and design-system docs.
- **`prototype`** (manual): several real variants behind a live picker when the direction is undecided.

## Cross-cutting

- **`unslop`**: strip AI tells from code and prose.
- **`write-skill`** (manual): create or change a kstack skill.

## Overlaps

| Torn between | Ask | Then |
|---|---|---|
| `design` / `prototype` | Can the decision be made from a sketch or static mockups, or does it need real materials and interaction? | Sketch or mockups → design (Doc or canvas). Real → prototype. |
| `design` / `backend` | Is the question how to build this feature, or where knowledge and state live? | Feature → design (it pulls in backend for ADRs). Placement → backend. |
| `ui-design` / `ui-build` | Is it the wrong look, or right but unfinished, janky, or awkward to use? | Look → ui-design. Unfinished or hard to use → ui-build. |
| `ui-motion` / `ui-build` | Does the motion look wrong, or do frames drop and things load slowly? | Looks wrong → ui-motion. Frames, lists, loading → ui-build. |
| `review` / `ship` | Judging the change, or getting it merged and live? | Judging → review. Merging, releasing → ship. |

## Not here

Discipline gates (design first, TDD, root cause, evidence) live in the global rules (`~/.agents/AGENTS.md`; `~/.claude/CLAUDE.md` for Claude Code) and apply everywhere; no skill restates them.
