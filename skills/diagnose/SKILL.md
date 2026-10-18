---
name: diagnose
description: Diagnosis loop for hard bugs, production incidents, flaky tests, and performance regressions — build a red-capable feedback loop, reproduce, rank hypotheses, instrument, fix with a regression test, and grade the evidence behind every conclusion. Use when something is broken, throwing, failing intermittently, slow, or wrong in production, when a fix "didn't take", or when the user says debug, diagnose, investigate, or "why is this happening".
---

# Diagnose

CLAUDE.md already requires root cause before fix. This skill is how to get there without guessing, and how to tell a proven cause from a plausible one.

## 0. Production incident: stop the bleeding, read the error

When something broke in production, two steps come before the loop:

1. **What changed?** Line the symptom's start time up against deploys, config changes, dependency bumps, and upstream announcements. A change minutes before the symptom is the first suspect.
2. **Read the actual error text** in the service's logs (see grading in §4). If a recent deploy is the likely cause, roll back first and keep diagnosing; the rollback is itself a test of that hypothesis.

Then build the loop below against a non-production target, or read-only against production.

## 1. Build the loop first

The skill is a **red-capable** command: one command you have already run, that drives the real code path, asserts the user's exact symptom, is deterministic, takes seconds, and runs unattended. Until it exists, do not form theories from reading code.

Ways to get one, roughly in order: failing test at the seam that reaches the bug; curl script against a dev server; CLI run diffed against a known-good output; headless browser script; replay of a captured request or event log; a throwaway harness around one function; a fuzz loop; `git bisect run` between a good and bad state; the same input through old and new versions.

For intermittent bugs the goal is a higher reproduction rate, not a clean repro: loop the trigger, parallelize, add stress. 50% is debuggable, 1% is not.

If no loop is possible, stop and say so: list what you tried and ask for environment access, a redacted artifact, or permission to add instrumentation.

Redact secrets in everything you show; build loops against env vars.

## 2. Rule out the boring causes

Before chasing a logic bug, check that you are looking at the code you think is running:

- A stale process: a backend or utility process that does not hot-reload, a cached build, an old worker still serving.
- State that bypassed its owner: rows written directly to the database are not seen by in-memory schedulers or caches; send the change through the owning API.
- Passing tests in one file say nothing about the full suite, and unit tests passing do not mean the feature runs end to end.

## 3. Hypothesize, then probe

- Write 3–5 ranked, falsifiable hypotheses ("if X, then changing Y makes it disappear") before testing any. Show them to the user, but keep working unless they redirect.
- Each probe tests one prediction and changes one variable. Prefer a debugger; otherwise targeted logs tagged with a unique prefix (`[DEBUG-a4f2]`) so cleanup is one grep.
- Performance: measure a baseline first, then bisect or profile. Logs are the wrong tool.

## 4. Grade the evidence

Every conclusion states what it rests on. The grades that have been confused before:

- **Server logs and error bodies outrank metrics.** Read the actual error text first; use metrics to narrow scope. A conclusion drawn only from metrics is labeled as such ("metrics-only inference").
- **Your own test traffic or telemetry proves the pipeline works, not that a user's failure was the same.**
- **Blaming upstream requires proof**: the decoded output compared against the upstream source, or a minimal repro against upstream alone. Until then it is a hypothesis.
- **A status code from a bad request is ambiguous.** A 404 for an invalid payload cannot distinguish a missing route from an upstream rejection; check the error-body format or the route table.
- A retry or timeout that makes the symptom go away is not a root cause.

## 5. Fix, lock, clean up

- Regression test first, at a seam that reproduces the real call pattern. If no honest seam exists, say so; that is an architecture finding, not a reason to write a shallow test.
- Re-run the original (un-minimized) loop after the fix.
- Remove every tagged debug line and throwaway harness.
- The PR states the hypothesis that proved correct.
- When the cause is perceived latency, ship the perceived fix (defer, stream, show progress) first and leave the deep fix until it still feels slow.

## 6. When the cause is not found

Say it is not found. Record in the issue or PR: the hypotheses ruled out and how, the instrumentation now in place, and what signal would confirm the next hypothesis. A mitigation that hides the symptom is labeled as a mitigation.
