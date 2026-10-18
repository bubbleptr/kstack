---
name: ui-engineering
description: "The invisible details and rendering work that make UI feel finished and fast: font rendering, layout-shift prevention, interaction states, focus, hit areas, z-index, optical alignment, scrollbars, empty states, refresh flash, plus list virtualization, preloading, lazy loading, off-screen work, and static generation. Use when building or reviewing UI components, doing a polish pass, fixing 'feels off/unfinished', or UI that is slow, laggy, or reflows. Triggers: polish, font-smoothing, tabular-nums, layout shift, CLS, jumping text, skeleton, placeholder, z-index, stacking context, focus ring, hit area, tap target, optical alignment, truncation, line-clamp, text-wrap balance, safe-area, empty state, pointer-events, flash on refresh, virtualization, long list, infinite scroll, re-render, preload, loading=lazy, font loading, FOUT, IntersectionObserver, SSG."
---

# UI Engineering

A finished interface is mostly invisible work: text that renders crisply, numbers that don't jiggle, hover states that don't shift layout, hit areas you never miss. Fast UI is mostly work you *don't* do: don't render nodes nobody sees, don't fetch at request time what you could build at build time, don't move layout after the user started reading. Most jank is self-inflicted; the fix is usually deleting or deferring work. Apply this as a pass over every component you build or review.

**Match the project's styling system** (Tailwind, plain CSS, CSS Modules, CSS-in-JS) and express every fix in it. Don't add a global stylesheet to a Tailwind codebase or utility classes to a styled-components one. Tokens (z-index scale, font weights) go wherever the project keeps design tokens.

## References

| Read when | File |
| --- | --- |
| Extending a hit area, styling a scoped scrollbar, adding anchor offsets, safe areas, truncation, or a z-index scale | [references/polish-recipes.md](references/polish-recipes.md) |
| Virtualizing a list, preloading fonts/images, pausing off-screen work, or generating static content | [references/performance-recipes.md](references/performance-recipes.md) |
| Reviewing a component or page, or hunting why it "feels off" | [references/review-checklist.md](references/review-checklist.md) |

Motion (easing, durations, press `scale(0.96)`, hover transitions, `transition: all`, `will-change`, theme-switch transitions, per-frame React animation) lives in `ui-motion`. Shadows, borders, gradients, elevation, image outlines, and dark-mode surfaces live in `ui-design`.

## Text

1. **Antialiased font smoothing, always:** `-webkit-font-smoothing: antialiased` on `body`. Subset custom fonts to the characters, alphabets, and languages actually used; nothing else ships.
2. **Never change font weight on interaction.** Weight changes on hover or selection reflow text and shift layout. Keep weight constant; signal state with color (`.tab { font-weight: 500 } .tab.selected { color: var(--color-primary) }`). Define weights as variables (`--font-weight-normal: 400; --font-weight-medium: 500; --font-weight-semibold: 600; --font-weight-bold: 700;`) so they're adjustable globally.
3. **Tabular numbers for anything that changes.** Counters, prices, timers get `font-variant-numeric: tabular-nums` so digits keep equal width and don't jiggle.
4. **Real typographic characters:** `…` not `...`, `’` not `'`, `“` `”` not straight quotes. `text-wrap: balance` on `h1`–`h3`. Larger text wants tighter letter spacing, smaller text looser: pair each font size with its tracking (a `Text` component is the right home; exact values are font-dependent).
5. **Preload fonts** so text doesn't reflow when they land; fonts loaded without preload cause visible FOUT on every cold load.

## Layout stability

6. **Zero layout shift from dynamic content.** Skeletons, image placeholders, and dynamic areas get hardcoded dimensions (or `aspect-ratio`) matching the loaded state. Images and videos always declare dimensions. Reserve space for async content with skeletons.
7. **No flash on refresh.** Interactive components render their real state on first paint: persist state to `localStorage`/`sessionStorage`, hydrate correctly under SSR, set initial state before render. Never default-then-correct.
8. **Empty states teach.** An empty list is a first-run screen, not an error: short headline, one line of explanation, primary action. Size it to the filled state's dimensions so completing the action doesn't shift layout.

## Interaction states

9. **Hover must not change layout** (see 2). Transition details are in `ui-motion`.
10. **Focus:** never remove the focus outline, and don't recolor it to anything but grey, black, or white; colored outlines clash with the interface.
11. **Selection:** if you style `::selection` at all, keep a subtle tint; don't make selected text unreadable or brand-colored for its own sake.
12. **Minimum hit areas:** `44×44px` for touch/mobile, at least `40×40px` on desktop. Extend small controls with a pseudo-element; don't scale the visual. Two hit areas must never overlap: if an extended area collides with a neighbor, shrink it to the largest non-colliding size.
13. **Decorative elements are inert:** `pointer-events: none` on anything purely visual so it can't steal clicks; `user-select: none` on code-built illustrations.

## Structure

14. **Fixed z-index scale.** No arbitrary `z-index: 9999`; define a scale and use only its steps. Better: avoid z-index where possible with `isolation: isolate` (or `position: relative`) so children can't leak above unrelated UI.
15. **Align by eye, not by math.** Mathematical centering often reads off-center:
    - Button with trailing icon: icon-side padding = text-side padding − 2px (`pl-4 pr-3.5`).
    - Play triangles: shift `2px` toward the pointed side (`margin-left: 2px`).
    - Asymmetric icons (stars, arrows, carets): fix the SVG viewBox/path itself; a `1px` margin is the fallback.
16. **Native scrollbar on the page.** Never replace it. Customize only inside small scrollers like code blocks: `8px` wide, `rgba(0,0,0,0.2)` thumb, `4px` radius.
17. **Anchors and safe areas:** `scroll-margin-top` equal to the sticky header height on anchor targets; `env(safe-area-inset-*)` padding for notches and home indicators.

## Rendering and loading performance

18. **Virtualize long lists.** Don't render hundreds of DOM nodes when a handful are visible; lists over ~50 items are virtualized. Rendering a 2,000-row table and wondering why scroll is choppy: virtualize.
19. **Preload critical, lazy-load the rest.** Preload fonts and above-the-fold (hero) images; below-the-fold images get `loading="lazy"` so they don't compete with critical resources.
20. **Stop work that's off-screen.** Pause looping animations, video, and any resource-intensive operation when not visible (`IntersectionObserver`). Videos playing in background tabs and below the fold waste CPU and battery.
21. **Generate static content at build time.** Blog posts, changelogs, docs, anything that doesn't change per request is never fetched at request time.

## Related

`ui-motion` (animation and its performance), `ui-design` (layout, type, color, surfaces), `ui-interaction` (forms, touch, a11y, mobile), `component-api`, `ui-review`.
