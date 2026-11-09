---
name: design
description: Turn a feature request or change into a concrete sketch before implementing — the caller's usage, data shapes and signatures, one structurally different alternative, and at most three open questions — instead of an open-ended interview. Use when the work is large enough that the global rules call for a fuller design, when the scope is vague ("it's kind of big… or small"), when the request touches layout or placement, when there are competing architectures, or when the user says design, plan, sketch, architect, or "how should we build this".
---

# Design

Sketch first, then let the human challenge the sketch. Challenging something concrete costs a reviewer far less attention than answering an abstract decision tree, and most decisions cannot be made before the code exists anyway.

## 1. Facts are your job

Read the code, the Accepted ADRs, `CONTEXT.md`, and nmem before asking anything. Questions about facts the repo can answer are noise. When something cannot be read, state the assumption at the top of the sketch instead of asking.

- Check what the request **means** before how to build it. Read ambiguous words in the context of the current setup ("switch to flash" meant the model already in use, not a different vendor's). A page built as "best of each category" when it meant "all of them" was a semantic miss no layout could fix.
- Treat memory as a hypothesis. A remembered priority or decision gets checked against the current code or issue before it shapes the sketch.
- When a reference or mockup exists, the target is a 1:1 reproduction of it; free interpretation drifts.

## 2. The sketch

Write it in this order; a Doc splits it across two tabs (section 3):

1. **Usage**: what the caller writes — the call sites, the CLI invocation, the screen the user sees. The shapes below are derived from this.
2. **Shape**: data structures and types first, then signatures and module placement. Name the load-bearing decisions.
3. **Alternative**: one structurally different design (not a variant of the same shape), with one line on why it lost.
4. **Decided**: reversible choices you made, each with a one-line reason. Do not ask about these.
5. **Deferred**: what cannot be decided before implementation, marked as such rather than forced.
6. **Open questions**: at most three, only for irreversible choices or product direction, asked in plain text after the sketch (on a Doc, as the round). Prefer one question with a recommended answer over three open ones.

Then stop for the user when the work is large or an open question is irreversible; otherwise proceed on the sketch and let the user course-correct.

For layout or placement work, the sketch shows where it goes, what form it takes, and the trade-off — options side by side if needed — before any UI code. Building from a one-line description led to a full redo.

## 3. Where the sketch goes

Small work: the reply. Large or layout work, when the `Artifact` tool is available: a private page the user comments on, with the link and one line in the reply. Build each page by its type's own instructions; this section only says what goes on it.

**Doc** (`quickstart` intent `document`). The Doc is a decision surface, not the design: a page that carries the whole sketch reads like a spec, and the user spends attention on parts that need no answer.

- **First tab, the decision**, top to bottom:
  1. One line on how to answer: comment on a question to change it; anything left untouched counts as agreed.
  2. What is being built and why, at most three sentences, your guesses marked as yours.
  3. This round's questions. Each is a heading that is the question itself, then your recommendation, then one line of why, so a one-line comment ("agree", "B") settles it.
  4. **Settled**: each decision on one line, marked by who made it: the user, accepted recommendation, or your call. Section 2's Decided items start here as your calls.
  5. **Deferred**: one line each.
- **Second tab, the sketch**: usage, shape and the alternative from section 2, drawn for the current answers. The first tab never repeats them. Redraw it whenever an answer changes the design.
- **Rounds**: one round by default. A second exists only for a fork: when more than three questions qualify and one decides whether others apply at all, ask it first, with any independent questions up to three in total, and name the held-back questions in one line under it so the user sees what their answer opens. When the user answers, move each answer into Settled as one line, delete the questions it made moot, add any it raised, and reply with one line on what changed and how many questions remain. The page gets shorter each round, not longer.
- **After agreement** (no questions left), the first tab becomes the dated summary: what gets built, in one sentence; what the user will see, as numbered steps; a decisions table (decision, choice, who made it); **Check these**, your calls worth a second look; Deferred, one line each; and the size in PRs. This summary is for the user; the builder works from the code and briefs (section 5).

**Canvas** (`quickstart` intent `design`): layout options as artboards side by side, the recommended one marked in its title, each trade-off in a sticky beside it. It picks a direction from mockups; real fonts, tokens and interaction are `prototype`. A Doc question settled on the canvas names the chosen artboard in Settled.

Both pages:

- **Mock data only**: fixtures or story data, never credentials, secrets or real user data. A private page is one Share click from public.
- **Reading answers**: when the user says they have commented, read every thread on both pages in one pass. Name the canvas threads you could not answer (not sent to Claude); Doc threads have no such gate.
- **After agreement**: apply what the comments asked, write the Doc summary, and stop maintaining the pages. The repo stays the source of truth (section 5); the pages are the record of how it was decided.

## 4. Hold the line

- When the official way to do something does not exist, say so and stop. A workaround that imitates the missing feature (a cron job standing in for a platform's automation) is worse than not doing it.
- A decision the user declined stays declined. Later fixes do not reintroduce it.
- When the reviewer or critic you were told to use is unavailable, wait for it; do not substitute another.

## 5. After agreement

- Record the decision as an ADR only if it qualifies (hard to reverse, surprising, real trade-off), citing the page link when one was used; the format lives in `backend`. When implementation forces a change to an accepted decision, add a superseding ADR in the same PR and flip the old one's status to `Superseded by ADR-NNNN`; never rewrite the old decision.
- No spec file. The code and tests become the description of the feature; the sketch's job ends at agreement.
- Work too large for one session splits into briefs per `handoff`, one per reviewable PR, stacked when they depend on each other.
