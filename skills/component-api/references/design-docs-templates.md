# Design System Docs: Templates

## Structure

```
How big is the system?
 ├── One token file and ≤5 components → a single DESIGN.md at the repo root
 └── Bigger → a docs folder
      ├── a root file → the system's character in 2–3 sentences,
      │   plus an index of the granular files
      └── one file per topic → colors.md, typography.md, buttons.md, forms.md
```

Root file under 100 lines; every topic file under 150. When a file outgrows that, split it by topic instead of trimming rules.

## Component template (four parts, this order)

````md
## Button

Use for actions. Navigation styled as a button is a Link.
Actions in tight spaces use IconButton.

Variants: `primary`, `secondary`, `danger`. Nothing else exists —
an unlisted variant is a bug, not an option.

- `primary` — the single most important action on a screen. One per view.
- `danger` — destructive or hard-to-undo actions only.
- `secondary` — everything else. This is the default.

```tsx
// Correct
<Button variant="secondary">Cancel</Button>

// Incorrect — ghost doesn't exist in this system
<Button variant="ghost">Cancel</Button>
```
````

1. When to use it, and the named alternative ("use a dropdown for actions, a combobox for dropdowns with search").
2. Which variant applies where, and which is the default.
3. One correct and one incorrect example, real code from the codebase.
4. A decision flowchart when a choice has three or more options.

## Decision flowchart

```md
Is it the single most important action on the screen?
 ├── Yes → variant="primary"
 └── No
      ├── Is it destructive, or hard to undo?
      │    └── Yes → variant="danger"
      └── Default → variant="secondary"
```

## Token docs

Load-bearing rule: components use semantic tokens (`--bg-surface`), never primitives (`--gray-100`), never raw values. Then one flowchart per role ("what background do I use", "what text color do I use"), built from the semantic tokens found in the inventory.

## Language rewrites

- Hedge: "Use small text sparingly" (what is sparingly?). Rule: "Body text is 16px, captions are 14px. Never go below 12px."
- Rule with its why: "One `primary` per view — two primary buttons means the screen has no hierarchy."

Delete every "reasonably", "tasteful", "where appropriate", "try to avoid"; replace each with a number or a never/always.

## Verification prompt and drift table

Prompt: "Build a settings screen for notification preferences. Use our existing components."

| The drift | The hole | The fix |
| --- | --- | --- |
| Hardcoded a color | Token rule isn't findable or isn't strict | Move the layer rule into the root file |
| Invented a variant | Docs list variants without "nothing else exists" | Close the set explicitly (and tell the user if the union type isn't strict) |
| Picked the wrong component | No "when to use it vs the alternative" line | Add the comparison for that pair |
