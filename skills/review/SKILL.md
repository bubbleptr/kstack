---
name: review
description: Review a branch, PR, or work-in-progress diff along two axes — repo standards and the Accepted ADRs/intent it must honor — plus a UI pass when the interface changed; and triage findings from automated reviewers (dureview, CodeRabbit, Codex) into fix-or-ignore calls. Use when asked to review code, a PR, or "my changes", before opening or merging a PR, after a subagent delivers work, or when reviewer comments arrive on a PR.
---

# Review

## 1. Pin what is being reviewed

- **Branch or PR**: diff against an explicit base, `git diff <base>...HEAD` (three dots, merge-base). Run from the main workspace without a base, a review comes back empty and looks like a pass.
- **Work in progress**: committed changes are not the whole picture. Review `git diff <base>` (working tree against the base, staged and unstaged) and list untracked files with `git status --porcelain`; an empty `...HEAD` diff does not mean there is nothing to review.
- Confirm the ref resolves and the diff is non-empty before reviewing.
- Problems that already exist on the base branch are out of scope for this PR. Note them as a separate issue instead of blocking.

## 2. Two axes, reported separately

Run them as two independent passes (parallel subagents for anything beyond a small diff) so one does not mask the other:

- **Standards**: the repo's documented conventions (`AGENTS.md`, `CLAUDE.md`, `CONTRIBUTING.md`, lint config, design-system docs). Cite the rule for each violation. Fowler's code smells are judgment calls, labeled "possible", and a documented repo rule always overrides them. Skip what tooling already enforces.
- **Decisions and intent**: the Accepted ADRs touching the changed area, the PR description or brief, and `CONTEXT.md` vocabulary. Report what was asked but missing, what was built but not asked (scope creep), and what contradicts a recorded decision. There is no spec to diff against; the decisions are the contract.

When the diff touches UI, add the `ui-review` pass as a third section.

Diffs of roughly 20+ files or several hundred lines also get a pattern-level pass: repeated workarounds, the same shape copied across files, a new abstraction with one caller. Line-level checks miss these.

Present each axis under its own heading. Do not merge or re-rank across axes; end with the count and the worst finding per axis.

## 3. Adversarial pass (opt-in)

When the design is contested or the change is risky (concurrency, money, migrations, auth), get an independent challenge before shipping: a fresh subagent or the advisor, told to break the change rather than approve it. Concurrency and reconciliation designs are settled before dispatching an implementer, not discovered in review.

## 4. Triage automated reviewer findings

- A green check is not zero findings. dureview reports a summary in the check-run output and findings as inline comments: read both (`gh api repos/<owner>/<repo>/commits/<branch>/check-runs`, `gh api repos/<owner>/<repo>/pulls/<n>/comments`).
- Draft PRs trigger no automated reviewer. Mark the PR ready (`gh pr ready`) when the review is wanted.
- For each finding, check whether it holds in this platform's semantics before accepting it (a suggested SQL fix grouped by UTC where the product meant local hours). Reviewer tools have no intent context and flag deliberate behavior.
- When a finding is inherited from vendored or upstream content, say so in the call; it may still be worth fixing.
- When fixing one finding, fix the same pattern everywhere it occurs in the change, not just the cited line.

## 5. Who judges what

Agents handle mechanical and pattern-level checks. The human's attention goes to architecture, product behavior, and taste; surface those explicitly and keep the rest out of their way.
