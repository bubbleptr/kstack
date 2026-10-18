---
name: prototype
description: "Explore an interface direction by building several genuinely different versions of one piece of UI behind a live picker, instead of one hardcoded guess, then record the decision and delete the prototype. Use when someone asks for a few options, variations, versions, or directions for a component or flow; says they aren't sure which approach to take (\"not sure if\", \"which feels better\", \"try a few\", \"show me some\"); wants two ideas compared side by side with real product data; wants a throwaway /proto route, playground, or sandbox; or wants the numbers behind an interaction (duration, easing, spring, blur) put on live sliders or a control panel with slow-motion replay. Not for a decision already made: then build one thing."
---

# Prototype

Take one described piece of UI, build several genuinely different versions, put them behind a live picker so the user can flip between them at full size, write down what they picked and rejected, then delete the prototype. This is the exploration phase: a rough description and a decision nobody has made. If the user has already decided, build one thing; a picker holding three answers to a settled question wastes everyone's time. Reviewing existing UI is `ui-review`; polishing a finished component is `ui-engineering`.

## Posture

The value is **divergence**: three tints of one idea teach the user nothing. Each variant must be a direction you could defend shipping on its own. Divergence doesn't lower the craft bar: a sloppy variant loses on execution, the user rejects a good direction because the type was ugly, and the run taught nothing.

## Hard rules

1. **Never touch production code during exploration.** Prototypes live in their own throwaway directory, imported by nothing; prototype code left in a real component stays as dead props and unused variants. The in-place branch (Phase 5) is the single exception, and still keeps variants outside the real component: it adds only one marked mount block that Phase 7 reverts. Integration happens only in Phase 7, only for the chosen variant.
2. **Real materials, worst content.** Real fonts, tokens, components (different materials answer a question about an interface you aren't building). Fill it with the content that breaks it: the longest real name, the two-line title, forty rows, the empty state. Pretty placeholders make every variant look fine and hide the decision.
3. **Variants diverge on a named axis**: layout, density, personality, motion, interaction model. You must be able to state each axis in a phrase before building. Sharing the project's tokens is not convergence; variants should feel native.
4. **Every variant fully works.** Real interactions, motion, and states. No dead buttons, no lorem ipsum, no "imagine this part".
5. **The picker is chrome, not a contestant.** Copy [references/picker.md](references/picker.md) verbatim; its look never adapts to the project.
6. **The output is a decision, not code.** Every run ends with chosen values and rejected options written down (Phase 7), then the surface deleted.

## Workflow

### Phase 1: Scope

One thing per run. If the description spans several components ("the dashboard"), pick the single highest-leverage piece, say which and why, and offer the rest as follow-up runs. Restate the brief in one sentence: what it is, where it lives, what it must do.

### Phase 2: Recon

- **Stack**: framework, styling system, motion library. Write every variant in that system.
- **Materials**: colors, radii, spacing, fonts, easing and duration tokens, reusable components.
- **Personality**: playful consumer app or crisp dashboard? This bounds how far the boldest variant may go.
- **Context**: what it renders against, beside what neighbors, at what sizes.
- **Baseline**: if the piece exists, the current implementation is variant 1, unchanged, so every other variant is measured against something real.

No project at all? Take the standalone branch in Phase 5 with a restrained default: neutral grays, one accent, system font stack.

### Phase 3: Loadout

Load the skills the brief needs **now**, before variant code; retrofitting craft after the picker is built means editing every variant.

| When the brief involves | Load |
| --- | --- |
| Text as the design (headlines, editorial, docs, data tables, a type scale); palette, accent, theme, dark mode, contrast; cards, modals, dropdowns, elevation, borders, glass, anything that stacks; a hero, pricing table, landing page, blog, or changelog | `ui-design` |
| Motion: entrances, exits, gestures, drag, press and hover feedback | `ui-motion` |
| Inputs, buttons, validation, signup, checkout, settings | `ui-interaction` |
| Long lists, heavy media, anything that could feel slow | `ui-engineering` |
| The API of the component being promoted (Phase 7 only) | `component-api` |

**Every run, no exceptions:** `ui-design` before building (hierarchy, spacing, restraint decide whether any of it reads as designed), then `ui-engineering` (polish) and `ui-interaction` (touch and accessibility) before the user sees the picker. Invoke each skill by name; if one isn't installed, say so and proceed rather than inventing its rules. The loadout applies to every variant equally, or the comparison is rigged.

**Done when** you can name each loaded skill and the dimension it owns in this run, and no variant was written from memory of a rule you could have looked up.

### Phase 4: Choose directions

Default **3 variants**; up to 5 when asked or the design space is genuinely wide. More than 5 dilutes the comparison. List the set first: a name and axis for each. Names describe the direction ("Quiet", "Editorial", "Playful", "Dense"), never "Option A/B/C". Two directions differing only in accent color or copy are one direction; replace one with a different layout, interaction model, or motion story.

**Done when** every variant has a name and stated axis and no two share an axis position.

### Phase 5: Build the harness

- **Project with a dev server** (default): an isolated route (`/proto/<slug>` or equivalent), one file per variant plus a small harness file. No types, no tests, no careful naming. Nothing in production imports from it.
- **No project / static context**: one self-contained HTML file with inline CSS and JS.
- **Context that can't be faked**: the in-place picker, below.

Load [references/picker.md](references/picker.md) now and build exactly that. Render **one variant at a time, full size, in realistic surrounding context** (a toast needs a page behind it, a card needs siblings, a button needs a form); thumbnails distort spacing and scale. Switching is **instant**, no animation: it happens a hundred times a session.

**In-place branch.** Only when the surrounding state genuinely can't be reproduced in an isolated route (a step deep in an authenticated flow, a cell in a live data table, layout dependent on real parent or server state); a variant judged against faked context is worse than no prototype. State the reason out loud first ("the empty state only appears after a real sync, so this runs in place"). Then hold all four:

1. **Variants still live outside production**: same throwaway directory, one file each; the real component imports none.
2. **One mount block, clearly marked**: a single dev-only block at the call site swapping the real element for the picker, fenced in comments naming the prototype so it's greppable and revertible in one edit. If you're editing the real component to accommodate a variant, you're on the wrong branch.
3. **Dev-only, fails closed**: gate on the environment, not a flag someone could ship enabled. If the gate is unavailable, the real component renders.
4. **Real neighbors, still worst content**: drive the flow to the longest name, the forty rows, the empty state.

**When the brief has numbers in it** (duration, easing, spring, blur, radius, offset), add the control panel from [references/controls.md](references/controls.md). A number you would hardcode is an unmade decision; on a slider the user settles it in ten seconds instead of ten prompts.

### Phase 6: Verify and hand off

Run the harness and flip through every variant yourself: each renders, every interaction responds, the console is clean. Check each against the `ui-review` flag-on-sight list (`transition: all`, unlabeled icon buttons, sub-16px inputs, animating layout properties, missing reduced-motion): exactly the mistakes a fast prototype ships. Screenshot each variant if browser tooling is available. Then present the set and **stop; the choice belongs to the user**:

| # | Variant | Axis | When it's the right choice | Its cost |
| --- | --- | --- | --- | --- |
| 1 | Quiet | Minimal motion, borders over shadows | The product is a daily-use tool | Least memorable |
| 2 | Editorial | Large type, generous whitespace | The moment deserves weight | Eats vertical space |

Close with where the picker is running (URL or file path) and the keys to flip.

**Done when** every variant is reachable and behaves correctly, no console errors, and the table names each cost honestly.

### Phase 7: Promote and write the decision

Write the decision first; it's the only artifact that survives:

```md
Dropdown entrance — decided from /proto/dropdown
- Direction: scale + fade, not slide
- scale from 0.96, opacity 0 → 1
- 180ms enter, 140ms exit, cubic-bezier(0.32, 0.72, 0, 1)
- Rejected: slide-down felt heavy at 200ms and jittery below it
```

The rejected options save the most time; without them the same discussion recurs in three weeks. Integrate the winner following project conventions (`component-api` if it becomes shared), then tear down: delete the prototype directory and route, revert the in-place mount block, grep for the fence comments and confirm nothing is left. If the user wants another round, keep the harness and rerun Phase 4 diverging *around* the direction they gravitated to.

## Invocation variants

| Invocation | Behavior |
| --- | --- |
| `<description>` | Full workflow: scope → recon → loadout → 3 variants → picker → wait for choice |
| `<description> x5` | Same, with that many variants (capped at 5) |
| `tune <variant>` | Keep one variant, put its numbers on the control panel ([references/controls.md](references/controls.md)) |
| `riff <variant>` | New round: keep the harness, fresh set diverging around that variant's direction |
| `keep <variant>` | Write the decision, promote that variant, delete the prototype surface |
| `keep <variant>, leave the picker` | Promote, but keep the prototype surface |

## Tone

Sell each variant honestly: one line on when it wins, one on what it costs. Never pre-pick a favorite in the table; if asked, answer from the product's personality and how often the thing is used, not aesthetics alone. If two variants converged while building, cut one and say so: two truly distinct directions beat one padded to three.
