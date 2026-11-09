---
name: ship
description: Get a change from a working branch to users — commits, PR body, stacked PRs with gh stack, merging and cleanup, then release scope, versions, tags and notes, app/npm artifacts, server deploys (blue-green, atomic release dirs, proxy config), and verifying what is live. Use when committing, opening or describing a PR, pushing to an open PR, waiting on CI or `gh pr checks`, reading a failed run, splitting work into a stack, merging, cleaning up branches, cutting a release, tagging, publishing a package or app build, deploying, switching traffic, rolling back, or checking what is running in production.
---

# Ship

The global rules (`AGENTS.md` / `CLAUDE.md`) already set the route to main: feature branch, ready PR, one dureview pass, merge only when told, clean up afterwards. This skill holds what goes into each step and the traps along the way, then everything from main to users.

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

## Waiting on CI

- Right after a push, confirm the run exists with `gh run list --commit <sha>` before watching. If the list is empty, check again a couple of times rather than sleeping a fixed number of seconds first.
- Wait with `gh pr checks <N> --watch --fail-fast --interval 30`, started once, in the background, then end the turn and let the exit notify you: the exit says the watch is done and whether it saw a failure; the buckets below say which checks failed. Without `--fail-fast`, `--watch` waits for the slowest job, so a web failure can sit unseen behind a 13-minute iOS job.
- Nothing before or after the watch: no `sleep` in front, no `| tail`, `| grep` or `| head`, no `; echo` or `|| true`, no second foreground copy under `timeout`, and don't poll the output file. A pipe reports the last command's exit code, and `head` kills the watch once it has its lines, so a run still in progress comes back as exit 0.
- A review bot's check (dureview with a P0/P1) is a failure, so `--fail-fast` stops before CI finishes. When the required checks cover every CI job you need to wait for, directly or through a summary check that depends on all of them and fails when any fails, add `--required`. `--required` filters by check, not by workflow: a required web job in the same workflow as an optional iOS job ends the watch before iOS finishes. Without required checks, `--required` has nothing to wait on. Otherwise watch the CI runs themselves: `gh run watch <run-id> --exit-status`, one per workflow, with the ids from `gh run list --commit <sha>`.
- Read the result by bucket, not by exit code: `gh pr checks <N> --json name,bucket --jq '.[] | select(.bucket == "fail")'`. `cancel` and `skipping` are not failures, but they are not passes either. A run can be cancelled by a newer push, by hand, or because no runner was allocated, and a job can be skipped by a path filter or by a failed dependency. Check with `gh run list --commit <sha>`: if another run on the same head passed, ignore it; if the head's only run was cancelled or skipped, nothing has verified that head.
- "no checks reported" means nothing ran on that head. Find out why with `gh run list --commit <sha>` rather than guessing.
- While a run is still going, `gh run view --log-failed` prints nothing. Fetch the failed job's log directly: `gh api repos/<owner>/<repo>/actions/jobs/<job-id>/logs`, then strip colour codes.

## After merge

- Squash-merged branches always look "not fully merged" to git. Before force-deleting, check with `git cherry -v main <branch>` (lines starting with `-` are already on main) or a diff against main.
- Merging is not deploying. When the deploy is triggered by a push to main (Workers Builds, Zeabur, Pages), confirm that a build actually started for the merge commit; at least one platform has missed a squash-merge push.

## Release 1: Scope the release by the deployable unit

- Ship only the units the change touches. A docs-only or frontend-only change gets no backend version bump, tag, or GitHub Release. What is live is each unit's current release (the server's release SHA, the page actually served), not the last merge.
- Before deploying, check what is actually running and how it got there: the image tag or SHA, and whether anyone deployed outside the normal path. Record the rollback target (previous image, compose or config backup) before changing anything.
- Deploy only when told. Once told, run the whole chain without stopping for further confirmation, and report each step's result. Scripts with interactive prompts get their non-interactive flag (e.g. an `ASSUME_YES` env var) or they abort silently mid-chain.

## Release 2: Versions, tags, notes

- The version bump, changelog, and release notes go in a `chore/release-x.y.z` PR. After it merges, tag **main** and push the tag; the tag-triggered workflow builds, signs, and publishes.
- Pre-1.0: features and breaking changes bump the minor; fixes bump the patch.
- Update every place the version lives in the same PR: manifests, the workspace version in `bun.lock` (then verify with `bun install --frozen-lockfile`), and version examples in docs and `AGENTS.md`.
- Release notes (`docs/releases/vX.Y.Z.md` where the project uses it) list only user-visible changes, never what did not change.
- A tag that produced a published release never moves. Delete and re-tag only if nothing was published. A stuck CI run on a tag is re-run on the same tag (manual dispatch), not fixed by a new commit.
- Tests that only run on tags (packaged end-to-end) are run manually before tagging whenever they changed.

## Release 3: Artifacts

- Verify the artifact as a user downloads it, not the local build: version, embedded permission descriptions (`Info.plist`), signature and notarization (`codesign`, `spctl`, `stapler`), first launch. A build with an entitlement but no permission description crashed on launch.
- A packaging step that degrades fails the release; it does not warn. Nothing in CI depends on a GUI (Finder scripting).
- Apple notarization failing with 403 "agreement missing" means the account holder must accept the latest agreement; it is not a code problem.
- npm: a brand-new package can resolve to a placeholder version for a minute or two; verify with `--prefer-online`. Publishing several packages together must be safe to re-run after a partial failure.

## Release 4: Servers

- **Order**: proxy config, then frontend, then backend, unless the change says otherwise. Proxy config: diff the live file against the repo first (hand hot-fixes), back it up, validate as the service user, then reload.
- **Blue-green**: deploy green, move real traffic to it and verify under that traffic, upgrade blue, then restore the normal split. A node with no traffic proves nothing. A health endpoint that the proxy does not route is not an outage signal.
- **Atomic frontend releases**: build into `releases/<sha>`, carry the previous release's hashed assets forward so open tabs keep loading, then switch a `current` symlink in one rename. Release directories live where the server runtime resolves its modules. Pruning old releases removes them as rollback targets; know how many are kept.
- **Building on a production host** caps CPU and memory and has swap. An out-of-memory build is a production incident.
- Frontend and backend that deploy separately are released and verified separately; a backend deploy says nothing about the page.

## Release 5: After deploy

Verify with the live system: the served version or SHA, a real request through the changed path, the error rate for a few minutes. UI releases get the pre-launch UI sweep from `review` before tagging, not after.
