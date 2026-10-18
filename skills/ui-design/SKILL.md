---
name: ui-design
description: "Visual design judgment for interfaces: hierarchy, spacing, restraint, typography, OKLCH color and contrast, surfaces (radii, shadows, borders, gradients, dark mode), icons, and marketing pages, plus an unslop pass that strips AI-generated tells. Use when building, styling, or reviewing any UI or landing page; when choosing type, palette, borders, shadows, spacing, icons, or button hierarchy; when converting colors to oklch, building a scale or Tailwind v4 theme, fixing contrast, or deriving dark mode; when text wraps, truncates, or reads badly; when building a hero, pricing page, blog, docs site, or changelog. Triggers: a UI that \"looks off\", \"feels cheap\", \"looks AI-generated\" or \"vibe-coded\", purple/indigo gradients, glassmorphism, emoji feature grids, centered-hero sameness, scroll animations, parallax, carousels, design audit or critique."
---

# UI Design

Most details are invisible individually but compound; the aggregate of invisible correctness is the differentiator. Apply these calls when producing UI, cite them when reviewing it. The rule is the default; context decides (several rules are deliberate trick questions: tabs, baseline grids, contrast specs, outlined-vs-filled icons, alert shapes).

**Work in the project's styling system.** Detect Tailwind, plain CSS, CSS Modules, CSS-in-JS, vanilla-extract, StyleX, whatever is there, and express every fix in it. Tokens go wherever the project keeps tokens. A design fix never justifies a second styling approach.

## 1. Hierarchy and restraint

- **One primary action per view.** Three equally weighted buttons means no next step. Demote: primary, secondary ghost, text link.
- **Hierarchy is subtraction.** Make the important thing prominent by making everything else recede, not by making everything louder.
- **Restraint reads as intent.** Fewer colors, fewer font weights, one icon library, three illustration colors instead of six. A limited palette looks decided; a full one looks indecisive.
- **Specific beats generic.** "Save changes" not "Submit"; "Your email must include an @ symbol" not "Invalid input". Empty states say why and what to do next. Sentence case, front-load the key word, no "please".
- **Every state is designed.** Default, hover, focus, active, disabled, loading, empty, error, each visually distinct. A missing pressed state makes a button feel dead.
- **Accessibility is a floor, not a polish pass.** Focus rings (never bare `outline: none`), labels wired to inputs, real `<button>`, color + icon + text for errors.
- **Space with system, not eyeballing.** `gap` on the parent over per-child margins; paragraph spacing = 1× line-height; body capped at 65ch; content containers capped ~1200px; breakpoints where content breaks, not at 768/1024.
- Never put "Delete" beside "Confirm": proximity implies equivalence. Text goes beside or below full-bleed images by default; text over them needs scrim, color-matching, and a contrast check.

The full right/wrong lists for layout, IA, interaction, motion timing, a11y, copy, and components, plus the "looks off" checklist: [references/design-pairs.md](references/design-pairs.md). Read it when building or auditing any of those areas, or when a UI looks off and you can't name why.

## 2. The unslop pass (run after generating any screen)

The tells are mechanical: a model picks the most common pattern for every decision at once, and the accumulation is a watermark. Stripping tells is half the job; putting a point of view back in is not optional, because a scrubbed screen with no opinionated choice is still generic.

1. **Squint test.** Could this screen belong to any of ten thousand products? Find what ties it to *this* product. If the answer is "the logo", everything else is up for review.
2. **Subtract first.** Walk every icon, line, badge, caption, container, and word and ask *what breaks if this is gone?* If nothing, delete it. "Adds visual interest" and "fills the space" are not answers. Walk again: deletions expose each other. Do this before restyling; there's no point recoloring something you're about to delete.
3. **Sweep in order: color, typography, layout, decoration, iconography, copy, motion.** Color first because it's the loudest tell. The tells:
   - Unchosen indigo/violet/blue-purple. More than one gradient per screen, or one with no job. Dark-and-neon as default "premium" costume.
   - Gradient text (`bg-clip-text text-transparent`); headlines get a solid color. Inter `font-black tracking-tight` centered as a non-decision.
   - The template hero (pill badge, centered headline, subhead, button + "Learn more", gradient orb). Icon-title-description card grids and bento cells sized before the content existed.
   - Glassmorphism with nothing behind it. Orbs and blobs behind the hero (if it feels empty after deleting them, the foreground is too weak). More than one depth cue per surface; `rounded-2xl` everywhere.
   - Emoji as icons; ✨ sparkles as "AI". Generic headlines a competitor could paste unchanged; fabricated proof (invented user counts, stock-avatar testimonials, unearned logo walls): real or cut.
   - `fade-in-up` on every section, `hover:scale-105` on non-clickable cards.
4. **Put a point of view back in.** Derive the palette from something real (domain, brand artifact, content). Make one distinctive type decision (one; three is a costume party). Show the actual product. Let content break symmetry. Repeat one signature element. Restraint is a feature: adding "personality effects" is re-slopping.
5. **Final audit.** Ask "what here would make a designer suspect a model built this?" and fix survivors. Then "name the one choice a competitor couldn't copy-paste"; if nothing answers, go back to step 4.

Full rationale and the fix for each tell: [references/unslop-tells.md](references/unslop-tells.md). Read it when a tell is present and the fix isn't obvious. Generic copy goes through `unslop`.

## 3. Typography

- **Two typefaces cover most products; three is the ceiling.** Pair across categories (serif display over sans text), never two lookalike sans-serifs. Marketing gets more latitude than product UI. A typography fix never requires a new typeface unless the task asks.
- **Every size comes from a scale.** One-off values rot. On teams, name steps by role (`text-caption`), not size.
- **Leading by role**, unitless: ~1.1 display, 1.5–1.6 body. **Tracking by size**: slightly negative on display, slightly positive on small uppercase, zero on body.
- **Measure** 45–75 characters, ~65ch. **Wrap by role**: `balance` on headings, `pretty` on short blurbs, neither on long-form.
- `tabular-nums` on anything that updates or stacks in columns. `font-synthesis: none`. `woff2` only, only weights you use. High-level properties (`font-weight`, `font-variant-*`) over raw tags.
- Truncation must leave a path to the full text. Casing via `text-transform`, not baked into strings. Real characters (`…`, en/em dashes, curly quotes). Never underline non-links; bold for UI emphasis, italic for citations.
- **Floors:** body 16px, dense UI 14px, captions 13px, almost never below 12px; mobile inputs ≥16px (never `maximum-scale=1`); light weights never small; contrast 4.5:1 (3:1 at ≥ ~24px).

Read [references/typography-text.md](references/typography-text.md) when setting scale, leading, tracking, measure, wrapping, truncation, underlines, selection, RTL, or auditing text (includes the symptom table and checklist). Read [references/typography-fonts.md](references/typography-fonts.md) when choosing, pairing, loading, or configuring fonts (formats, variable axes, OpenType features, small caps, fallback stacks). Read [references/typography-tailwind.md](references/typography-tailwind.md) when writing any typography fix in a Tailwind codebase.

## 4. Color

- **Everything new is oklch.** When editing legacy hex/rgb/hsl, swap values only; keywords, gradient shape, and hex-expecting third-party config stay. HSL lies about lightness and drifts hue.
- **Semantic tokens** (`--color-border-subtle`) over raw hex. Hover via `color-mix(in oklch, var(--color) 85%, black)`. Tints by reducing chroma, not opacity.
- **L answers light vs dark:** surface L > 0.6 takes dark text; ≤ 0.6 takes light text.
- **Contrast lives in the L gap.** Repair by moving L only; raising chroma does nothing. APCA |Lc| 60 body (75 comfortable), 45 large, 30 non-text; WCAG 4.5:1 / 7:1 body, 3:1 large and UI when compliance is named.
- **One hue per ramp;** >10° spread across a scale is visible drift, rebuild it. Sibling hues share L per step and the same *fraction* of each hue's chroma ceiling, never one absolute C.
- **Check gamut before shipping high chroma.** Clamp C holding L and H; serve P3 chroma behind `@media (color-gamut: p3)` over an sRGB base.
- **Dark mode is a remap, not a repaint, and not inverted light mode.** Swap which scale steps feed which role; keep the layering hierarchy (the topmost surface stays the lightest dark grey). Desaturate brand colors 20–30% for dark mode or they vibrate.
- **Neutrals agree with the brand hue** (blue UI, cool greys). Never pure `#808080`: tint neutrals with a slight hue bias. If green is the brand primary, success gets a different color. Disabled uses a dedicated muted token, not `opacity: 0.4`.
- Tailwind v4: `@theme` tokens in oklch, no hex.

Read [references/color-oklch.md](references/color-oklch.md) when converting colors, building a tonal ramp or multi-hue family, deriving dark mode, or auditing a scale for drift. Read [references/color-contrast.md](references/color-contrast.md) when measuring or repairing a contrast pair. Read [references/color-gamut-p3-tailwind.md](references/color-gamut-p3-tailwind.md) when chroma is high, targeting P3, supporting pre-2023 browsers, or defining/porting a Tailwind v4 theme.

When colors change, report them by file: every touched declaration, old value beside new, plus the rule it violated (drifting hue, insufficient L gap, out-of-gamut chroma).

## 5. Surfaces

- **Derive nested radii:** inner = outer − padding. Copying the parent's radius pinches the corners, the most frequent giveaway of unpolished UI. Past a 24px gap the layers read as independent.
- **Shadows for depth, borders for separation.** Cards, buttons, menus, popovers, dialogs: a stacked translucent shadow token (it composites on any background; a solid border is tuned to one). Dividers, table gridlines, form-field edges: keep real borders (on inputs a visible edge is accessibility).
- **Separation borders recede:** light mode alpha (`rgba(0,0,0,0.08)`), dark mode solid (`#2a2a2a`-style; alpha white glows). Hairline 0.5px on high-DPI.
- **Dark mode flattens shadows** to one low-opacity white 1px ring.
- **Gradients are eased**, never plain linear (banding). Fades use `mask-image`, not overlays. Never fade scrollable content.
- **Every image gets a 1px inset outline:** black 10% light, white 10% dark. Never tinted neutrals or the accent.
- **Themes flip variables,** not per-component `dark:` classes. One depth cue per surface.
- Hover transitions only `box-shadow`, never `all`.

Read [references/surfaces.md](references/surfaces.md) when writing radius, shadow, border, gradient, image-outline, or dark-theme CSS; it holds the exact tokens, symptoms, and checklist. Non-surface finishing (font rendering, layout shift, z-index, scrollbars): `ui-engineering`.

## 6. Icons and illustrations

One icon library per surface, stroke matched to adjacent text weight, round caps, optical (not mathematical) centering, a text label for ambiguous metaphors, one icon one meaning. Illustrations: ~3 colors, recolored to tokens, stroke matched to icons, redrawn rather than scaled up. Read [references/icons-and-illustrations.md](references/icons-and-illustrations.md) when drawing, choosing, sizing, or placing any icon, favicon, or illustration.

## 7. Marketing pages

A visitor sees a landing page once, so it earns more animation budget than product UI, but the budget is still small.

- **Motion maps to user input.** Never: scroll-triggered fade-ups, scroll hijacking, parallax not 1:1 with scroll, auto-advancing carousels. If the user didn't cause it, cut it.
- **Intro animations play once per session** (`sessionStorage`, not `localStorage`).
- **Pre-render** blog, docs, changelog with revalidation; never fetch at request time. **Preload** fonts and above-the-fold images.
- **Content lives in the DOM:** hover nav submenus are visually hidden, not mounted on hover.
- **CTAs follow auth state:** logged out "Get Started", logged in "Go to Dashboard".
- **Docs are for copying:** copy button on every snippet, every page exportable as markdown (`.md` URLs), visual examples for every concept. **Blogs and changelogs are feeds:** RSS, balanced headings.

Read [references/marketing-pages.md](references/marketing-pages.md) when implementing any of the above or before shipping a marketing page (pre-ship checklist).

## Reviewing UI

Present findings as one markdown table, `| Before | After | Why |`, one row per issue with a one-line reason. Sweep against the real roles on screen (body, headings, labels, live numbers, inputs), not just the stylesheet. Stay silent on rules already followed; fix with the smallest diff, and don't restructure markup unless an animated underline or expandable truncation needs an extra element. Review with fresh eyes when possible: flaws invisible at the end of a build session are obvious the next day. Full review workflow: `ui-review`.
