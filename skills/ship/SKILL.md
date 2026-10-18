---
name: ship
description: Get a change from a working branch to merged on main — commit messages, the PR body, stacked PRs with gh stack, merging, and post-merge cleanup. Use when committing, opening or updating a PR, writing a PR description, splitting work into a stack, merging a PR or a stack, or cleaning up branches after a merge.
---

# Ship

CLAUDE.md already sets the route: feature branch, ready PR, one dureview pass, merge only when told, clean up afterwards. This skill holds what goes into each step and the traps along the way.

## Commits

- Conventional Commits **with a scope**: `fix(billing): …`, `feat(composer): …`. Squash merges put the PR title on main, so the title follows the same rule.
- No session links, "Generated with …" lines, or co-author trailers from the harness, whatever its default.
- Small, ordered commits; each one builds.

## Before opening the PR

- Run the same build the release runs (`bun run build`, `go build ./...`), not only lint and unit tests. A type error that only the production build catches has reached release day before.
- Check that CI actually ran. A workflow that fails within seconds is usually billing, permissions, or a runner problem, not your code.
- Fixes for problems that already exist on main go in their own PR, not folded into a feature.
- When cherry-picking an upstream fix described as "zero-conflict", apply it onto current main and run it; missing earlier commits can turn it into a no-op.

## The PR body

Write in the project's documentation language. Use [references/pr-body.md](references/pr-body.md) for the template and the visual forms (pseudocode, call tree, component tree, file tree, diff, Mermaid). Read it every time you write or rewrite a PR body. In short:

1. **Summary**: the smallest visual that makes the change clear, next to one or two sentences.
2. **Evidence**: before and after. Screenshots for visual changes; otherwise the exact command and its output.
3. **Merge danger**: one-way or two-way door, and the blast radius.
4. `Closes #N` for every issue it resolves, so the tracker closes itself.

## Stacked PRs

- `gh stack submit` creates **drafts** unless told otherwise, and drafts trigger no reviewer. Use `gh stack submit --auto --open`, or `gh pr ready` each PR. When a tool default conflicts with a repo rule, follow the rule and say so.
- Merge a stack with `gh stack merge --merge --yes` from the top branch: it is atomic, and plain `gh pr merge` refuses stack members.
- Never `--delete-branch` on a lower PR of a stack: GitHub closes every PR based on it. To recover: push the merged PR's last commit (`headRefOid`) back to the deleted branch, `gh pr reopen` the closed PR, then `gh pr edit --base main`.

## After merge

- Squash-merged branches always look "not fully merged" to git. Before force-deleting, check with `git cherry -v main <branch>` (lines starting with `-` are already on main) or a diff against main.
- Merging is not deploying. When the deploy is triggered by a push to main (Workers Builds, Zeabur, Pages), confirm that a build actually started for the merge commit; at least one platform has missed a squash-merge push.
