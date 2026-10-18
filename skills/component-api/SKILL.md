---
name: component-api
description: "Design React component APIs that are composable, predictable, and hard to misuse, and write the design system docs (DESIGN.md or a docs folder) that tell humans and agents which component, variant, and token to use. Use when creating or refactoring components, building a component library, reviewing a props API, deciding how a component exposes customization, or creating or updating design system, component, or token docs. Triggers on: props design, compound components, composition vs configuration, controlled/uncontrolled, defaultValue, asChild, Slot, render props, forwardRef, variants, className override, boolean props, prop drilling, slots, Radix, shadcn, DESIGN.md, component guidelines, CLAUDE.md/AGENTS.md design rules, or an agent that hardcoded a color, invented a variant, or picked the wrong component."
---

# Component API

A component API is a contract: the common case is a one-liner, the uncommon case is possible without forking. Failure is always too rigid (users fork or hack around you) or too configurable (30 props nobody can hold in their head). Implement variant styles in the project's existing styling system; don't impose a new one.

Code for every pattern below: [references/implementation.md](references/implementation.md) — read when writing or refactoring a component.

## API decisions

1. **Composition over configuration.** Expose structure as JSX children (`<Card><CardHeader>…`), not configuration objects (`header={{ title }}`).
2. **Compound components for multi-part UI**, sharing implicit state through React context (root provides, parts consume) instead of prop-drilling through one root.
   - Use when: related elements share implicit state; the component has slots (header, body, footer); order or presence of children varies between usages; consumers need flexible composition.
   - Don't when: simple fixed structure; 1–3 props; the structure never changes.
3. **Goldilocks customization, layered in this order:**
   1. `variant` — predefined options (`primary`, `secondary`, `destructive`)
   2. `size` — predefined sizes (`sm`, `md`, `lg`)
   3. `className` — escape hatch for one-off overrides
   4. `asChild` — render as a different element (Radix pattern)

   Too little and users fork; per-style props (`backgroundColor`, `hoverBackgroundColor`, `paddingX`…) are a confusing maintenance nightmare.
4. **Name props like the platform.** Mirror HTML and stay consistent across every component (`disabled` everywhere, never `isDisabled` on one and `disabled` on another). Booleans are positive (`disabled`, `open`), never double negatives (`notEnabled`, `isNotClosed`). Event handlers start with `on` (`onChange`, `onOpenChange`), never `handleChange` / `blurHandler`.
5. **Every stateful component supports controlled and uncontrolled**: internal state from `defaultValue` by default; external when the consumer passes `value` + `onChange`.
6. **Play nice with the DOM.**
   - Forward refs on every component wrapping a DOM element; otherwise focus management, tooltips, and popover positioning break for composers.
   - Spread remaining props (`{...props}`) so `aria-*`, `data-testid`, and other HTML attributes pass through.
   - Defaults cover 80% of usages: `variant = "primary"`, `size = "md"`, `type = "button"` (not `"submit"`; the safer default).
7. **`asChild`** lets consumers swap the rendered element (`<a>`, Next.js `Link`) while keeping behavior and styles.
8. **Children for simple content, render props for data** (`renderItem={(user) => …}`), slot props (`header={<h2/>}`) for optional sections that take arbitrary JSX.
9. **Wrap complex components in error boundaries** so one broken widget doesn't take down the page.
10. **One folder per component** with its tests and an `index.ts` of public exports.

## Common mistakes

- **Prop explosion**: `leftIcon`, `rightIcon`, `iconSpacing`, `iconSize`… use children: `<Button><Icon /> Click <Arrow /></Button>`.
- **Boolean soup instead of variants**: `<Button primary large rounded>` invites invalid combinations; use `variant="primary" size="lg" radius="full"`.
- **Premature abstraction**: don't create a shared component until the code has been copy-pasted 2–3 times. Wait for the pattern, then extract.
- **Swallowing props**: destructuring everything and forgetting `{...props}` breaks `aria-*`, `data-*`, and testing attributes.
- **Missing ref forwarding**: breaks focus, tooltips, and popover positioning for anyone composing the component.

## Design system docs

Tokens and components define what exists; docs define when to use what. Write for a new engineer who joined yesterday and can never ask questions, which is exactly what an agent is. Every decision the docs leave out gets replaced by a guess, and a guess looks exactly like a decision.

Templates (structure tree, component template, flowchart, drift table): [references/design-docs-templates.md](references/design-docs-templates.md) — read when creating or updating DESIGN.md, component guidelines, or token docs.

Five steps, in order:

1. **Inventory the code first. Never skip.** Docs from memory are how agents learn variants that don't exist.
   - Tokens: find the token files (globals.css, theme files, Tailwind config); list every semantic token and the primitive backing it; note tokens that exist only as raw values (they need a rule, not a mention).
   - Components: list every shared component, its variant and size unions, and appearance-changing props. The union type is the source of truth.
   - Usage: open two or three real call sites per component to learn the de facto default and the patterns worth blessing or banning.
   - Every name in the docs must trace to a file line. A documented variant that doesn't exist teaches every future agent to invent it.
   - Nothing to document (no tokens, no shared components, ad-hoc styles)? Stop and say so; offer to set up semantic tokens and base components first.
2. **Pick the structure.** One token file and ≤5 components: single `DESIGN.md` at the repo root. Bigger: a docs folder with a root file (character in 2–3 sentences + index) and one file per topic. Root under 100 lines, topic files under 150; split by topic when one outgrows that, never trim rules. Small files get read; a 2000-line bible gets skimmed.
3. **Every component uses the four-part template, in order:**
   1. When to use it, naming the alternative ("dropdown for actions, combobox for dropdowns with search"). Without the alternative the agent has a description, not a choice.
   2. Which variant applies where, and which is the default, stated with "Nothing else exists." Most drift is an agent promoting everything to `primary`.
   3. One correct and one incorrect example, real code from the codebase. Models follow examples over prose; the incorrect one inoculates against the most common mistake.
   4. A decision flowchart when a choice has three or more options: a tree is walked the same way every run, a paragraph is interpreted.

   Token docs: components use semantic tokens, never primitives, never raw values; then a flowchart per role ("what background do I use").
4. **Language rules.**
   - Numbers and absolutes, no hedge words. Replace every "reasonably", "tasteful", "where appropriate", "try to avoid" with a number or a never/always.
   - Every non-obvious rule carries its why; a bare rule is applied blindly, a reasoned one generalizes.
   - Every line must change what the agent does. Don't explain what a button or token is; filler dilutes the rules that matter.
5. **Verify by prompting.** Docs are a hypothesis until an agent has run against them. Ask for a real screen in one sentence, read the drift against the drift table, and fix the docs, never just the output: fixing output fixes one screen, patching the doc fixes every screen after it. Repeat until a fresh prompt produces zero drift.
