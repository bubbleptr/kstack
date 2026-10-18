---
name: unslop
description: "Strip the tells that mark code or prose as AI-generated, then make it read like the surrounding codebase or like a person wrote it. Code: narrated or stale comments, commented-out code, try-catch theater, silent fallbacks, defensive null checks, `as any` / ts-ignore, debug logs, TODOs, dead code, duplicate helpers, one-call wrappers, V2/Enhanced names, compat shims, drive-by changes, weakened tests. Prose: delve/leverage/crucial, em dashes, rule of three, \"not just X but Y\", hedging, puffery, chatbot residue. Use when cleaning up agent-written code, before opening a PR, after any generated change, when drafting or reviewing READMEs, docs, release notes, commit messages, blog or marketing copy, UI microcopy, or when text \"sounds like AI\" / \"reads like ChatGPT\" or someone asks to de-slop or humanize it. Visual-design slop belongs to ui-design."
---

# Unslop

Nobody flags AI output for one bad line or one odd word. They recognize the accumulation: a comment narrating every statement, a try-catch around code that can't throw, the same fifty words and sentence shapes. Each choice is defensible alone; together they're a watermark. Pick the pass by what you're cleaning: code, prose, or both (comments and commit messages are prose inside code).

## Code pass

Why it happens: models were trained where code that *runs* wins, so they over-defend, over-comment, and over-scaffold; everything that makes a snippet survive in isolation makes it slop inside a codebase. Slop rarely breaks the build, but it buries intent, hides real failures behind quiet fallbacks, and doubles what the next reader holds in their head.

1. Scan the diff (or file) for the patterns in [references/code.md](references/code.md), in order: comments, defense, types, naming, structure, logging, tests, scope. Load it for every code pass.
2. **Fix by deleting first.** Rewrite only where behavior must survive. When removing a cast, catch, or fallback forces a choice about bad input, choose the loud failure (a throw or a failed build) over the quiet wrong answer.
3. **Match the room**: mirror the surrounding codebase's comment density, naming, and error-handling idiom. Slop is relative; a comment style normal in one repo is noise in another.
4. Final read: "What here would make a reviewer suspect a model wrote this?" Fix it. Repeat until the answer is nothing.

**The comment test**, which covers every comment: **does this comment state something the code cannot?** A constraint, a unit, a tradeoff, the bug it works around: keep. Everything else: delete.

Decisions that come up on nearly every pass:

- **Silent fallbacks are the most dangerous pattern**: `catch { return [] }` keeps tests green while the primary path is broken. Fail loudly unless degraded behavior is an explicit product decision, and then make the degraded state visible to the user.
- **Handle errors once, at a real boundary, adding context**, or don't catch. Validate untrusted input once at the boundary, then pass a trusted value through.
- **A cast survives only with a comment stating the invariant** that makes it safe.
- **Version-suffixed names**: replace the old implementation, update its callers, take its name. Keep a deprecated alias only for published public API with callers you can't update.
- **Before adding any utility, search for the existing owner of that job.** Build for the current caller; generalize on the second real use.
- **Never delete, skip, or weaken a test to pass it.** If the contract genuinely changed, say so and assert the new contract.
- **Keep the patch minimal**; propose unrelated cleanup as its own change.

Final audit:

- [ ] Every surviving comment states something the code cannot
- [ ] No catch swallows a failure; no fallback hides a broken primary path
- [ ] No cast silences the compiler without a stated invariant
- [ ] No name carries `V2`, `New`, or `Enhanced`; the old implementation is gone
- [ ] No helper duplicates an existing one; no wrapper just forwards
- [ ] No debug logs, no dead code, no TODO stubs, no commented-out blocks
- [ ] No test was weakened to pass
- [ ] The diff touches only what the task required
- [ ] The result reads like the surrounding codebase wrote it

## Prose pass

Two halves. First strip the tells. Then put a voice back in. The second half is not optional: text merely clean of tells is sterile, and sterile is its own tell.

1. Read the text once for what it's actually trying to say. You can't rewrite a sentence you haven't understood; you'll preserve the slop's shape.
2. Sweep in order using [references/writing.md](references/writing.md) (load it for every prose pass): words, then sentences, then punctuation and formatting, then claims. Each layer hides behind the one above it; a puffed-up claim is hard to see until the vocabulary around it is plain.
3. Put a voice back in ("Sounding like someone" in the same file): have a reaction, vary the rhythm, use "I" when it fits, be specific over evocative, let a little mess in.
4. Final read: "Which sentence here would make a reader suspect a model wrote this?" Fix it. Repeat until the answer is none.

Rules that hold regardless of context:

- **No em dashes.** Use periods or commas; not parentheses, en dashes, or spaced hyphens, which keep the same generated rhythm.
- **Count the real items** instead of grouping in threes. **Repeat the right word** instead of cycling synonyms.
- **Hedge once, at most.** Name the actor unless it's unknown or irrelevant.
- **Name the source or cut the claim.** Replace puffery with what happened; delete trailing "-ing" conclusions and generic endings.
- **Say what it does, not how it feels.** Every such sentence must restate as a concrete instruction, fact, or number, and must not fit unchanged into another project's docs. Fail either, cut it.

Final audit: read once more and ask "What here would make a reader think a model wrote this?" Check for survivors (a leftover "leverage", a rule-of-three that snuck back, a paragraph with no opinion in it). Fix and re-read until the question comes up empty.
