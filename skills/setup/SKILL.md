---
name: setup
description: "Configure a GitHub repo for issue tracking with `gh`: write the tracker conventions, add the rule-file pointer, and create the five kstack labels. Run once per repo before its first issue or milestone."
disable-model-invocation: true
---

# Set up a repo's issue tracker

Run this once per repo. It writes the issue-tracker conventions, adds a short pointer to the rule file the repo already uses, and creates the five labels those conventions expect. It creates no issues or milestones and deletes no labels.

This is prompt-driven, not a script: explore, show a draft, let the user edit it, then write.

Read [references/issue-tracker.md](references/issue-tracker.md) before drafting. It is the seed for `docs/agents/issue-tracker.md` and the source of the label set.

## 1. Explore

Report what is present, missing, and already correct:

- `git remote -v` and `gh repo view --json nameWithOwner,hasIssuesEnabled`. This skill supports only a GitHub remote with Issues enabled. If there is no GitHub remote, stop; do not fall back to GitLab, local markdown, or another tracker. If `gh` is missing or not authenticated, say so and stop.
- `CLAUDE.md` and `AGENTS.md`: which exists? Does either already have an `## Agent skills` block, and which subsections (`### Issue tracker`, `### Triage labels`, `### Domain docs`)?
- `docs/agents/`: an existing `issue-tracker.md`, `triage-labels.md`, or `domain.md`.
- `docs/roadmap.md`: whether it exists and what it says is current.
- Existing labels and milestones: `gh label list --limit 200` and `gh api 'repos/{owner}/{repo}/milestones?state=all' --jq '.[].title'`.

A repo configured by Matt's `setup-matt-pocock-skills` may have a `## Agent skills` block with `### Triage labels` and `docs/agents/triage-labels.md`. If present, both encode the old five-role triage vocabulary. This setup uses the five labels in the reference instead; leaving the old doc would give agents two vocabularies.

## 2. Draft and confirm

Show the user in one message:

- The rule file to edit and the exact `## Agent skills` block to add or replace: a `### Issue tracker` line naming GitHub Issues, `gh`, and `docs/agents/issue-tracker.md`, in the repo's documentation language. Use `CLAUDE.md` when it exists, otherwise `AGENTS.md`; if neither exists, ask which to create. Never create both.
- The complete `docs/agents/issue-tracker.md` to write, with repo-specific details filled in.
- The labels to create and the labels already present. Create only the five missing ones; list every other label as left untouched.
- Any old Matt sections or files to remove or replace, one line each.
- Any `docs/roadmap.md` correction, only when a closed milestone is still marked current.
- Milestones: none are created or changed.

Wait for the answer before writing, and apply the user's edits to the draft first.

## 3. Write

1. Write `docs/agents/issue-tracker.md` from the reference. When the file already exists, merge the reference's conventions into it and show the diff; keep repo-specific commands instead of overwriting them. Drop sections that only serve skills kstack no longer has (triage, wayfinder, to-tickets, to-spec). Keep the file in Chinese.
2. Edit the rule file in place: add the `## Agent skills` block, or add or update its `### Issue tracker` subsection. Do not append a second `## Agent skills`.
3. Replace Matt's leftovers as confirmed: remove the `### Triage labels` subsection and delete `docs/agents/triage-labels.md`. Leave an existing `### Domain docs` subsection and `docs/agents/domain.md` alone; this setup does not create them, and domain docs belong to `backend`.
4. Create each missing label with `gh label create <name> --description "<meaning>"`. Use the five names and meanings from the reference; never invent a sixth. Do not use `--force`; an existing label is left as it is.
5. If `docs/roadmap.md` marks a closed milestone as current, correct that one pointer. Otherwise leave the file alone; do not copy issue detail into it.

## 4. Verify

- `git diff` shows only the intended files, and the reference link resolves.
- `gh label list` shows the five labels and the untouched extras.
- Read the new `docs/agents/issue-tracker.md` as the agent that will use it: can it find where issues live, how to read one, what a milestone means, and where a spec goes?
- Report the files changed, labels created, and labels left alone.
