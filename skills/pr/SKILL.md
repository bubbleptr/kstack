---
name: pr
description: Get a change from a working branch onto main — commit messages, the checks to run before opening a PR, the PR body, stacked PRs with gh stack, waiting on CI and reading its result, merging, and cleaning up afterwards. Use when committing, opening or describing a PR, splitting work into a stack, pushing to an open PR, waiting for or reading CI, merging, or deleting merged branches.
---

# PR

The global rules (`AGENTS.md` / `CLAUDE.md`) already set the route to main: feature branch, ready PR, one dureview pass, merge only when told, clean up afterwards. This skill holds what goes into each step and the traps along the way. Triage of reviewer findings is in `review`; everything from main to users is in `ship`.

## Commits

- Conventional Commits **with a scope**: `fix(billing): …`, `feat(composer): …`. Squash merges put the PR title on main, so the title follows the same rule.
- No session links, "Generated with …" lines, or co-author trailers from the harness, whatever its default.
- Small, ordered commits; each one builds.

## Before opening the PR

- Run the same build the release runs (`bun run build`, `go build ./...`), not only lint and unit tests. A type error that only the production build catches has reached release day before.
- Run the typecheck after adding or changing tests. Runners that strip types (vitest, `bun test`) pass a test file with a missing type import, and the PR then fails CI on the first job.
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

## Waiting on CI

- Right after a push or `gh pr update-branch`, the new head may have no checks yet, and `--watch` then exits 1 at once with "no checks reported", the same exit code as a failure. Confirm checks exist first: `gh pr checks <N> --json name` lists them for Actions and external CI alike; if it is empty, check again every few seconds rather than sleeping a fixed time. If none ever appear, nothing ran on that head: find out why (`gh run list --commit <sha>` for Actions) rather than guessing.
- Wait with `gh pr checks <N> --watch --fail-fast --interval 30 > <file> 2>&1`, started once, in the background, and let the exit notify you. Without `--fail-fast`, `--watch` waits for the slowest job, so a web failure can sit unseen behind a 13-minute iOS job.
- Nothing before or after the watch: no `sleep` in front, no `| tail`, `| grep`, `| awk` or `| head`, no `; echo` or `|| true`, no second foreground copy under `timeout`, and don't poll the output file. A pipe reports the last command's exit code, so a red run notifies as exit 0, and `head` kills the watch once it has its lines. Read the file after the exit.
- Start the wait after every push, including when the harness has its own PR watcher. Those watchers (Auto-fix in the Claude desktop app) wake you on failures, conflicts and comments only; a green run is silent, and the user ends up telling you CI passed.
- A review bot's check (dureview with findings) fails within minutes, so `--fail-fast` can stop while CI is still running. When the required checks cover every CI job you need, directly or through a summary check that fails when any of them fails, add `--required`; it filters by check, not by workflow, and a summary check only reports once every job is done. Otherwise watch the CI runs themselves: `gh run watch <run-id> --exit-status > <file> 2>&1`, one per workflow, with the ids from `gh run list --commit <sha>`.
- Read the result by bucket: `gh pr checks <N> --json name,bucket --jq '.[] | select(.bucket == "fail")'`. The exit says the watch is done and whether it saw a failure; the buckets say which. `cancel` and `skipping` are not failures, but they are not passes either. A run can be cancelled by a newer push, by hand, or because no runner was allocated, and a job can be skipped by a path filter or by a failed dependency. Check with `gh run list --commit <sha>`: if another run on the same head passed, ignore it; if the head's only run was cancelled or skipped, nothing has verified that head.
- `gh run rerun --failed` is refused with "This workflow is already running" until every job of the run has finished, and `--fail-fast` exits at the first failed job. Wait for the run with `gh run watch <run-id> > /dev/null` before rerunning.
- While a run is still going, `gh run view --log-failed` prints nothing. Fetch the failed job's log directly: `gh api repos/<owner>/<repo>/actions/jobs/<job-id>/logs`, then strip colour codes.

## After merge

- Squash-merged branches always look "not fully merged" to git. Before force-deleting, check with `git cherry -v main <branch>` (lines starting with `-` are already on main) or a diff against main.
- Merging is not deploying. When the deploy is triggered by a push to main (Workers Builds, Zeabur, Pages), confirm that a build actually started for the merge commit; at least one platform has missed a squash-merge push.
