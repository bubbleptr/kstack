# PR body

Adapted from Dex Horthy's show-me skill (HumanLayer), via mattpocock/skills `pr`.

```markdown
## Summary

<one or two sentences>

<diagram, diff sketch, or tree>

## Evidence

- **Before:** <screenshot / output / failing test run>
  **After:** <screenshot / output / passing test run>

## Merge danger

**Door:** <one-way or two-way>
**Blast radius:** <one word, then one line if needed>

Closes #N
```

Skip preambles; keep prose short. Use the project's `CONTEXT.md` vocabulary.

## Summary: pick the smallest view that makes the point

- Logic or an algorithm → pseudocode:

```text
on(save)
  if content is unchanged
    return cached result
  write new content
  return fresh result
```

- Runtime control flow → call tree:

```text
submitForm
  createSession
    persistPrompt
    launchAgent
  navigateToSession
```

- UI structure → component tree, with the state and module boundaries that matter:

```text
<SessionPage> (apps/web/src/routes/session.tsx)
  useSessionEvents()
  <SessionToolbar>
    <RunSkillButton> (packages/ui)
```

- File responsibility or a broad refactor → shallow file tree:

```text
src/
├── commands/       # parses user actions
├── sessions/       # owns session state
└── transport/      # sends API requests
```

- Interaction between components or processes → Mermaid sequence diagram.

- What changes inside an existing shape → `diff` of that shape:

```diff
 submitForm
   createSession
     persistPrompt
+    expandSkillMention
     launchAgent
```

- Most of it is new, or omitted context would hide ownership → show the whole block.

Place each visual next to the sentence it supports. Keep only the calls, files, props, and states the reader needs. Usually one visual; rarely more than two.

## Evidence

Screenshots are best when the change is visual. Otherwise execution: the exact test or command that failed before and passes now, with its output. "Tests pass" with no command is not evidence.

## Merge danger

- **Door**: two-way if a revert fully undoes it; one-way if it migrates data, deletes anything, publishes, changes a public contract, or sends something external.
- **Blast radius**: who or what could break — consumers of a changed API, layout on mobile, every page that uses a shared component, billing.
