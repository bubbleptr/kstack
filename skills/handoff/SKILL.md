---
name: handoff
description: Move work between sessions, agents, and machines without losing it — write a handoff document for a fresh session, write a dispatch brief for a subagent or worker, verify a worker's claim of completion, and resume someone else's in-flight work. Use when the user says handoff, hand this off, continue in a new session, dispatch/delegate this, write a brief, pick up where X left off, or resume, and whenever you are about to spawn a worker for implementation.
---

# Handoff

Every handoff crosses a context boundary. What is not written down on the far side does not exist, and what the receiver claims is not true until shown.

## 1. Handoff document (to a fresh session)

Save it outside the repo (`$TMPDIR`). Contents:

- **Goal and current state**: what is done, what is in flight, what is next. State the delivery status precisely: local only, pushed, PR open (with link), reviewed, or merged. "Reviewed" is not done; merged is.
- **Decisions made and why**, and decisions explicitly declined, so they are not reopened.
- **Open questions** that need the user.
- **Pointers, not copies**: ADRs, PRs, issues, branches, commits by path or URL.
- **Suggested skills** for the next session.
- Redact secrets and personal data.

Use a fork-style handoff to start new work from this context; use compaction to keep going in the same one. Keep design discussion in one context and start each implementation in a fresh one.

## 2. Dispatch brief (to a subagent or worker)

The worker starts from nothing and does not read follow-up messages, so every mandatory requirement is in the brief itself. Long briefs go in a file the task points to.

Beyond goal, acceptance commands, files, constraints, and out-of-scope (the template is `~/code/harness/templates/task-brief.md`), state:

- **Cross-layer constraints** the worker cannot infer from the files it opens (whether an event must reach the journal, which process owns a piece of state).
- **The language of user-facing text.** A worker copied the brief's Chinese into the UI.
- **The branch, already pushed** to where the worker runs. Dispatch to a branch that does not exist there fails silently.
- Design decisions on concurrency, reconciliation, and data shape are made before dispatch. Fast implementers follow the brief faithfully and do not question it.
- Real-device or other user-visible actions need the user's consent each time, asked in plain text.

## 3. Verifying a worker's "done"

Agents report completion before their tool calls have actually landed. Done requires evidence in the report:

- `git diff --stat` (or `git diff origin/main -- <file>`) showing the change exists.
- Each verification command with its exit code.
- The branch pushed and, if requested, the PR URL.

An orchestrator showing a worker as idle or finished is not evidence; check for the commit on the branch. Send fixes back to the same worker rather than starting a fresh one.

## 4. Resuming in-flight work

Read the handoff, the linked PR/issue, and the branch state first. Then, before writing code, summarize to the user: background, what was found, what was decided, what is still open — and get agreement on the next step. Anything recalled from memory is checked against the current code before it is acted on.
