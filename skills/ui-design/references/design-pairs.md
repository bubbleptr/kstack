# Design pairs: right/wrong calls by area

Paired decisions with the reason the better choice wins. Several are trick questions on purpose: the rule is the default, context decides. Typography, color, and surface pairs live in their own references; this file covers the rest.

## Layout and spacing

1. Space siblings with `gap` on the parent, not `margin-bottom` on every child: margin leaves a trailing gap after the last item and bleeds past the component boundary; gap stays contained.
2. Baseline grids are an editorial tool: they shine in long-form layouts and are usually overkill in dense product UI. (Trick question: know which context you're in.)
3. Cap content containers (~1200px, centered) on wide monitors. Uncapped layouts produce unreadable line lengths and sparse, drifting compositions.
4. Set breakpoints where the content breaks, not at device sizes (768/1024). Device-named breakpoints routinely miss the actual failure point.
5. A simple marketing page doesn't need a 12-column grid; 8 columns (or fewer) does the job without configuration overhead.
6. Constrain sticky headers with `max-height` and `dvh` units. An 80px header on a 667px landscape viewport leaves almost no room for content.
7. Symmetric two-column layouts are static; a wider column offset by a narrower one creates tension and interest.
8. Fixed bottom CTAs on mobile must account for `env(safe-area-inset-bottom)` or they sit on top of the last form field on phones with home indicators.
9. Text over full-bleed images requires a scrim, color-matching, and contrast checks. Text in a column beside or below the image just works; default to that.

## Information architecture

1. Forty settings on one flat scrollable page buries everything equally. Group with a sidebar nav and progressive sections.
2. Never place "Delete" directly beside "Confirm". Proximity implies equivalence; separate destructive actions with whitespace and a visual break.
3. Name things by the user's mental model, not internal jargon: "Send messages in the background", not "Enable async relay mode".
4. Tabs trick question: "Account / Billing / Notifications" tabs are arguably fine; "This week / This month / This year" is the textbook-correct use. Tabs are for views of the same thing.
5. Empty states explain and act: "You haven't created a project yet" + a "Create project" button, not "No projects found".
6. Onboarding that creates the user's first item in one step beats an 8-stop feature tour. The tour teaches the product; the shortcut delivers value.
7. Three levels of nested dropdowns lose people. Prefer a flatter structure with breadcrumbs where depth is unavoidable.
8. Search data is IA feedback: if users keep searching "invoices" while the nav says "Billing", rename the nav item.
9. Error pages explain what went wrong and offer a specific recovery action. "Something went wrong" + a back button ends the journey.
10. Don't bury the primary action at the bottom of a long page; anchor it persistently in view. Most users never reach the bottom.

## Interaction

1. Hover states confirm affordance, not just decorate: pair the color change with a cursor change and a subtle underline where appropriate.
2. Never ship `outline: none` bare. Replace the default focus ring with a custom `outline` plus `outline-offset: 2px`.
3. Buttons need a pressed state (a slight depress/scale on click). Without it the button feels unresponsive.
4. Long list loads get skeletons, not spinners: the skeleton holds the page shape; a spinner collapses it.
5. Prefer optimistic updates that revert on error over waiting for the server. Optimistic feels instant.
6. Debounce search input at ~300ms. A request per keystroke hammers the server and creates race conditions.
7. Touch targets: 44×44px minimum. 28×28px causes mis-taps.
8. Trigger actions on `mouseup`, not `mousedown`. Mousedown fires before the user commits; they lose the ability to cancel by dragging away.
9. Auto-advancing inputs (verification codes) feel clever until someone needs to correct a digit. Waiting for tab is safer.
10. Copy-to-clipboard buttons confirm: swap to a checkmark for ~1.5s. Without feedback users click three more times.
11. Sliders update their value live during drag. On-release-only feels disconnected and makes precise values hard to hit.
12. Labels toggle their checkboxes: proper `<label>` association is one attribute and fixes a too-small hit target.
13. Show validation errors inline next to the offending field, not stacked above the form.

Depth on forms, touch, and mobile: `ui-interaction`.

## Motion timing

1. Hover transitions: 150ms feels native; 400ms feels like the UI is thinking.
2. Eight list items entering at once reads as a flash; staggered at 40ms intervals reads as arrival.
3. Any wait over 400ms needs an indicator (spinner or skeleton); silence past that reads as broken.
4. Entrances use `ease-out`, never `ease-in`: ease-in delays the initial movement, exactly when the user is watching.
5. Exits accelerate away (`ease-in`), they don't coast off screen (`ease-out`).
6. Exit-and-enter is asymmetric: an exit is departure, not an entrance played in reverse.
7. Modals and popovers scale up from the element that triggered them (shared origin), not from screen center. Exception: viewport-centered modals with no anchored trigger stay centered.
8. `transition: all 0.3s` is a bug factory: it silently animates width, height, and color, triggering layout recalculation. List the properties: `transition: opacity 0.3s, transform 0.3s`.
9. Choreograph: when three elements would move simultaneously and clash, let one move at a time.
10. Disable transitions during theme switches, or the change becomes a ragged cascade. With `next-themes`, set `disableTransitionOnChange` on the `ThemeProvider`.
11. Respect `prefers-reduced-motion`: movement-heavy UI triggers vestibular symptoms for some users.

Full motion decisions: `ui-motion`.

## Accessibility

1. `aria-label` describes the action, not the element: `aria-label="Search"`, never `aria-label="icon"`.
2. Error states need three signals: red border + error icon + message text. Color alone is invisible to colorblind users.
3. `<button>` over `<div role="button" onClick>`: the div isn't keyboard-focusable without extra work.
4. Pages with long navs need a visually hidden "Skip to content" link that appears on focus.
5. Wire `<label for>` to the input `id`. Unconnected labels don't focus their input when clicked.
6. Modals trap focus. If focus escapes to the page behind, background content becomes reachable and disorienting for screen reader users.
7. Keep DOM order matching visual order. CSS reordering (`order: -1`) desynchronizes what screen readers announce from what sighted users see.

Contrast thresholds: [color-contrast.md](color-contrast.md).

## Copywriting

1. Label buttons with the specific outcome: "Save changes", not "Submit".
2. Error messages tell users how to fix it: "Your email must include an @ symbol", not "Invalid input".
3. Placeholder text is not a label: it disappears the moment typing starts. Use a persistent label above the field.
4. Empty states give context and a next step: "You haven't added anything yet — start by creating a project", not "No items".
5. Sentence case for UI, not Title Case Throughout The Whole Product.
6. Front-load the important word: "Export ready, download it", not a 20-word sentence with "export" buried in the middle.
7. Drop "please" from imperative UI copy: "Confirm", not "Please confirm your selection before proceeding".

## Components

1. Button hierarchy: one primary, one secondary ghost, one text link. Three equally weighted primaries means no clear next action.
2. Inputs need visually distinct default, hover, focus, and error states.
3. Badge vs tag: a badge implies a count, a tag implies a category. A "New" feature marker is neither; it needs its own treatment.
4. Tooltips can't contain interactive elements; a "Learn more" link inside a hover tooltip is unreachable. Use a click-triggered popover.
5. In data tables, right-aligned tabular numbers do the work zebra striping was compensating for. Fix alignment before adding stripes.
6. Toast duration scales with content: compute it from word count (~200–250 wpm reading speed) instead of a flat 2s.
7. Don't wrap an entire card in one `<a>`: it kills text selection and screen readers announce the whole card as one link. Give the card a distinct CTA link.

## "Looks off" checklist

- [ ] More than one visually primary button in view
- [ ] Solid hex borders in light mode / glowing alpha borders in dark mode
- [ ] Raw hex values instead of semantic tokens
- [ ] Body text lines wider than 65ch
- [ ] Uppercase labels without loosened tracking
- [ ] Proportional figures in number columns
- [ ] Per-child margins instead of parent `gap`
- [ ] Inner radius equal to outer radius on nested cards
- [ ] Focus outline removed without replacement
- [ ] Placeholder used as the only label
- [ ] "Submit" / "Invalid input" / "Something went wrong" generic copy
- [ ] Color-only error signaling
- [ ] Two icon libraries mixed in one surface
- [ ] `transition: all` anywhere
- [ ] No `prefers-reduced-motion` handling
