---
name: release
description: Release and deploy — deciding what a change actually requires shipping, version bumps, tags and release notes, desktop/mobile/npm artifacts, server deploys (blue-green, atomic release directories, proxy config), and verifying what is really running in production. Use when cutting a release, bumping a version, tagging, writing release notes, publishing a package or app build, deploying to a server, switching traffic, rolling back, or checking what is live.
---

# Release

## 1. Scope the release by the deployable unit

- Ship only the units the change touches. A docs-only or frontend-only change gets no backend version bump, tag, or GitHub Release. What is live is each unit's current release (the server's release SHA, the page actually served), not the last merge.
- Before deploying, check what is actually running and how it got there: the image tag or SHA, and whether anyone deployed outside the normal path. Record the rollback target (previous image, compose or config backup) before changing anything.
- Deploy only when told. Once told, run the whole chain without stopping for further confirmation, and report each step's result. Scripts with interactive prompts get their non-interactive flag (e.g. an `ASSUME_YES` env var) or they abort silently mid-chain.

## 2. Versions, tags, notes

- The version bump, changelog, and release notes go in a `chore/release-x.y.z` PR. After it merges, tag **main** and push the tag; the tag-triggered workflow builds, signs, and publishes.
- Pre-1.0: features and breaking changes bump the minor; fixes bump the patch.
- Update every place the version lives in the same PR: manifests, the workspace version in `bun.lock` (then verify with `bun install --frozen-lockfile`), and version examples in docs and `AGENTS.md`.
- Release notes (`docs/releases/vX.Y.Z.md` where the project uses it) list only user-visible changes, never what did not change.
- A tag that produced a published release never moves. Delete and re-tag only if nothing was published. A stuck CI run on a tag is re-run on the same tag (manual dispatch), not fixed by a new commit.
- Tests that only run on tags (packaged end-to-end) are run manually before tagging whenever they changed.

## 3. Artifacts

- Verify the artifact as a user downloads it, not the local build: version, embedded permission descriptions (`Info.plist`), signature and notarization (`codesign`, `spctl`, `stapler`), first launch. A build with an entitlement but no permission description crashed on launch.
- A packaging step that degrades fails the release; it does not warn. Nothing in CI depends on a GUI (Finder scripting).
- Apple notarization failing with 403 "agreement missing" means the account holder must accept the latest agreement; it is not a code problem.
- npm: a brand-new package can resolve to a placeholder version for a minute or two; verify with `--prefer-online`. Publishing several packages together must be safe to re-run after a partial failure.

## 4. Servers

- **Order**: proxy config, then frontend, then backend, unless the change says otherwise. Proxy config: diff the live file against the repo first (hand hot-fixes), back it up, validate as the service user, then reload.
- **Blue-green**: deploy green, move real traffic to it and verify under that traffic, upgrade blue, then restore the normal split. A node with no traffic proves nothing. A health endpoint that the proxy does not route is not an outage signal.
- **Atomic frontend releases**: build into `releases/<sha>`, carry the previous release's hashed assets forward so open tabs keep loading, then switch a `current` symlink in one rename. Release directories live where the server runtime resolves its modules. Pruning old releases removes them as rollback targets; know how many are kept.
- **Building on a production host** caps CPU and memory and has swap. An out-of-memory build is a production incident.
- Frontend and backend that deploy separately are released and verified separately; a backend deploy says nothing about the page.

## 5. After deploy

Verify with the live system: the served version or SHA, a real request through the changed path, the error rate for a few minutes. UI releases get the pre-launch sweep from `ui-review` before tagging, not after.
