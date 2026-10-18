---
name: design
description: Turn a feature request or change into a concrete sketch before implementing — the caller's usage, data shapes and signatures, one structurally different alternative, and at most three open questions — instead of an open-ended interview. Use when the work is large enough that CLAUDE.md calls for a fuller design, when the scope is vague ("it's kind of big… or small"), when the request touches layout or placement, when there are competing architectures, or when the user says design, plan, sketch, architect, or "how should we build this".
---

# Design

Sketch first, then let the human challenge the sketch. Challenging something concrete costs a reviewer far less attention than answering an abstract decision tree, and most decisions cannot be made before the code exists anyway.

## 1. Facts are your job

Read the code, the Accepted ADRs, `CONTEXT.md`, and nmem before asking anything. Questions about facts the repo can answer are noise.

- Check what the request **means** before how to build it. Read ambiguous words in the context of the current setup ("switch to flash" meant the model already in use, not a different vendor's). A page built as "best of each category" when it meant "all of them" was a semantic miss no layout could fix.
- Treat memory as a hypothesis. A remembered priority or decision gets checked against the current code or issue before it shapes the sketch.
- When a reference or mockup exists, the target is a 1:1 reproduction of it; free interpretation drifts.

## 2. The sketch

Write it in the reply, in this order:

1. **Usage**: what the caller writes — the call sites, the CLI invocation, the screen the user sees. The shapes below are derived from this.
2. **Shape**: data structures and types first, then signatures and module placement. Name the load-bearing decisions.
3. **Alternative**: one structurally different design (not a variant of the same shape), with one line on why it lost.
4. **Decided**: reversible choices you made, each with a one-line reason. Do not ask about these.
5. **Deferred**: what cannot be decided before implementation, marked as such rather than forced.
6. **Open questions**: at most three, only for irreversible choices or product direction, asked in plain text after the sketch.

For layout or placement work, the sketch shows where it goes, what form it takes, and the trade-off — options side by side if needed — before any UI code. Building from a one-line description led to a full redo.

## 3. Hold the line

- When the official way to do something does not exist, say so and stop. A workaround that imitates the missing feature (a cron job standing in for a platform's automation) is worse than not doing it.
- A decision the user declined stays declined. Later fixes do not reintroduce it.
- When the reviewer or critic you were told to use is unavailable, wait for it; do not substitute another.

## 4. After agreement

- Record the decision as an ADR only if it qualifies (hard to reverse, surprising, real trade-off); the format lives in `domain-design`. When implementation forces a change to an accepted decision, write the correction back into the ADR in the same PR.
- No spec file. The code and tests become the description of the feature; the sketch's job ends at agreement.
- Work too large for one session splits into briefs per `handoff`, one per reviewable PR, stacked when they depend on each other.
