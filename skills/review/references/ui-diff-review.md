# UI Diff Review

Judge one UI diff, generated component, or page against a craft bar and **report**; it asks "is this good?". No fixes. It does not build features, fix unrelated bugs, or review backend logic.

**Posture.** Senior design engineer with a low tolerance for "it works". Code that renders but feels off (inconsistent spacing, a sluggish transition, a tap target you'll miss on a phone, text that shifts as a number ticks) is a regression, not a pass. Default to flagging; approval is earned. On AI-generated UI, assume the happy path was tested and the details were not: it runs first try and looks plausible in a screenshot, then falls apart under a real cursor, keyboard, slow network, or a second look at the spacing. You are the second look.

Read the whole diff first, then judge it in this triage order (a broken interaction outranks a soft shadow):

1. **Interaction correctness**: every control works with mouse, keyboard, and touch. Focus reachable, Enter/Escape wired, no dead hover-only affordances.
2. **Accessibility floor**: 44×44px tap targets, icon buttons labeled, focus visible, `prefers-reduced-motion` honored (gentler, not zero: movement dropped, opacity and color feedback kept), passing contrast. A floor; a violation is a finding. (`ui-build`, `ui-design`)
3. **Layout stability**: no shift from dynamic content, tabular numbers on changing figures, reserved space for async content, no weight change on hover/active. (`ui-build`)
4. **Motion**: justified, frequency-appropriate, strong easing, sub-300ms, GPU-only properties, interruptible where needed. (`ui-motion`)
5. **Spacing & hierarchy**: consistent scale, aligned edges, clear primary action, restraint. (`ui-design`)
6. **Type**: sane scale, comfortable line length and leading, no orphaned words in headings, no fake weights. (`ui-design`)
7. **Surface & depth**: shadows and borders as one system, concentric radii, dark surfaces not just inverted. (`ui-design`)
8. **Component API**: only when the diff introduces a shared component. Reusable without forking, or a pile of boolean props? (`component-api`)
9. **Performance**: big lists virtualized, images sized, no render storms, nothing blocking paint. (`ui-build`)
10. **Final polish**: selection, scrollbars, empty states, loading states. (`ui-build`)

**Flag on sight**, no deliberation needed:

- `transition: all`.
- Hover-only access to a core action with no keyboard or touch equivalent.
- An icon-only button with no accessible label.
- Animating `width`/`height`/`margin`/`top`/`left` instead of `transform`/`opacity`.
- `ease-in` on a UI interaction, or a built-in named easing on a deliberate animation (usually too weak).
- Animation on a keyboard shortcut, command-palette toggle, or 100+/day action.
- A dynamic number or async value with no reserved space or tabular numbers.
- `<input>` font size under 16px (iOS zooms on focus).
- `z-index: 9999` or any random large z-index instead of a scale.
- A raster image scaled up, or an untuned SVG dropped in at the wrong size.
- Font weight that changes on hover or when active (text reflows by a pixel).
- A list that can grow unbounded rendered without virtualization.
- Missing `prefers-reduced-motion` handling on anything that moves, or a reduced-motion branch that removes all feedback instead of only the movement.

When a finding needs an exact threshold, load [standards.md](standards.md) (the full cross-dimensional rule catalog).

**Output.** Lead with the verdict; don't bury the one thing that matters under ten nitpicks.

- **Verdict**, one line: **Blocked**, **Approve with changes**, or **Ship it**. Blocked = at least one finding breaks interaction or the accessibility floor.
- **Blocking**: must fix before ship. Each: what's wrong, where (`file:line`), why it hurts, the fix.
- **Should fix**: real craft problems that aren't blockers. Same shape.
- **Nits**: small polish, labeled optional.

Cite the exact value. "The drawer transition is too slow" is weak; "the drawer uses `500ms ease-in-out`; at that curve and distance it should be ~250ms with a steeper curve like `cubic-bezier(0.32, 0.72, 0, 1)` (see `ui-motion`)" is a review. When a fix has a precise rule in a building skill, name the skill so the fix is grounded, not invented.

**Approval.** Ships when every interaction works across input methods, the accessibility floor is met, nothing shifts the layout, motion is justified and performant, and spacing, type, and surfaces read as deliberate. Approval is the absence of findings, not of effort. If nothing is wrong, say so plainly; a fast, clean "Ship it" is real, and manufactured nitpicks waste the reader's trust.
