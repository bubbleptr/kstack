---
name: write-skill
description: Create, condense, or revise a skill in the kstack repo (~/code/kstack) — deciding whether it should exist at all, what goes in the body versus references, how to write the description, and how to prove it changes behavior with an A/B run. Use when adding a kstack skill, folding an upstream skill into kstack, pruning or rewriting an existing one, or when a skill seems to have no effect.
---

# Write a kstack skill

A skill earns its place only by changing what the agent does. Most candidate content does not: Opus already follows textbook practice, and `~/.claude/CLAUDE.md` already carries the discipline gates. The repo's design record is `docs/design.md`; the criteria below are its section 2 in procedure form.

## 1. Should it exist?

Walk this for every rule, then for the skill as a whole:

1. **Already in CLAUDE.md?** A one-line preference there (stack choices, TDD, root cause, evidence) already drives behavior. Do not restate it. `frontend-architecture` was dropped for exactly this.
2. **Which domain?**
   - Domains with production experience (backend, flow, shipping): the rule must be a personal preference, a decision, or a lesson from a concrete incident. Mine nmem for these; cite the incident in the why.
   - Domains still being learned (frontend design): the chosen teacher (emil) is the source of truth. Do not mine nmem for taste. Keep the teacher's judgments and exact values; remove only routing text, duplication, and preamble.
3. **Would the model do it anyway?** (Experienced domains only; teacher content from step 2 is kept even when a baseline would match it.) If yes, delete it, however true it is. A/B results in `docs/design.md` show which kinds of rules turned out to be no-ops (transactions without external calls, unique-key idempotency, expand/contract, FSD layout).

If nothing survives, the skill does not get written. Record why in `docs/design.md`.

## 2. Shape

- `skills/<name>/SKILL.md`, English, body ≤150 lines. Lookup material (tables, recipes, code, checklists, exact values) goes in `skills/<name>/references/`.
- Every reference is linked from the body with the condition for reading it ("Read X when …"). An unlinked reference is never read.
- Each rule pairs with a one-line why when the why is not obvious. Strict words ("always", "never", numbers) over hedges.
- State the target behavior rather than listing what to avoid, except for hard guardrails.
- Cross-reference other kstack skills by name. No pointers to uninstalled upstream skills.
- Docs about the skill (rationale, verification) go in `docs/` in Chinese; the skill itself stays in English.

## 3. Description

- What it does, when to use it, and the distinct situations that should trigger it. One trigger per situation; synonyms for the same situation are one trigger.
- Under ~900 characters.
- Must parse as YAML. A description containing `: ` must be quoted. Check: `python3 -c "import yaml,sys;yaml.safe_load(open(sys.argv[1]).read().split('---')[1])" skills/<name>/SKILL.md`.
- A router or anything only the user should fire gets `disable-model-invocation: true` and a one-line human-facing description.

## 4. Prove it

Run an A/B before merging when the skill could plausibly be a no-op:

1. Write one realistic scenario from the skill's domain into a scratch file, including the relevant CLAUDE.md lines so the baseline is fair.
2. Two subagents in parallel: one answers the scenario cold, the other reads the skill first and also reports "skill friction" (unclear, contradictory, missing, or would-have-done-anyway).
3. Fix the friction. In experienced domains, keep only the rules that produced a difference and delete the rest; in teacher-sourced domains, use the run to find friction and errors, not to delete the teacher's judgments.
4. Record the result, including its limits (one scenario, same source as the material), in `docs/design.md`'s verification notes.

Skills built almost entirely from concrete incidents may skip the A/B; say so in the PR.

## 5. Land it

- Feature branch, ready PR, dureview once, fix-or-ignore each finding, merge when told (per CLAUDE.md and `ship`).
- After merge, symlink the new skill into both `~/.agents/skills` and `~/.claude/skills`. An installed skill with the same name blocks the link.
- Removing an upstream skill changes the global install, so it needs the user's explicit go-ahead for that removal; approval to merge is not approval to uninstall. For a teacher's skill set (emil), wait until its whole migration is done. Once approved: back up, remove (`npx skills remove -g -y <name>`; untracked copies by hand), and clear dangling links under `~/.pi/agent/skills` and `~/.grok/skills`.
- Update the `kstack` router when a skill is added, renamed, or removed.
