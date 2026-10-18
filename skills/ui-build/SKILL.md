---
name: ui-build
description: "Make UI feel finished, fast, and right under every input: font rendering, layout-shift prevention, interaction states, focus, hit areas, z-index, optical alignment, scrollbars, empty states, refresh flash; forms and controls; touch, keyboard, and screen-reader interaction; native-feeling mobile web; list virtualization, preloading, lazy loading, off-screen work, static generation. Use when building or reviewing any component, form, input, button, modal, tooltip, menu, carousel, bottom sheet, video, timer, PWA, or full-screen mobile layout, doing a polish pass, or fixing UI that feels off, slow, laggy, reflows, or \"works in Chrome but feels wrong on my phone\". Triggers: tabular-nums, CLS, skeleton, focus trap, tap target, validation, double submit, autocomplete, iOS input zoom, aria-label, reduced motion, sticky hover, 300ms delay, dvh, safe areas, virtualization, loading=lazy, FOUT."
---

# UI Build

A finished interface is mostly invisible work: text that renders crisply, numbers that don't jiggle, hover states that don't shift layout, hit areas you never miss. Fast UI is mostly work you *don't* do: don't render nodes nobody sees, don't fetch at request time what you could build at build time, don't move layout after the user started reading. Most jank is self-inflicted; the fix is usually deleting or deferring work. And many users reach the interface through a thumb, a keyboard, or a screen reader, never a mouse: UI that only works with a fine pointer and perfect vision is broken, however polished. These rules are the baseline, not enhancements; apply them as a pass over every component you build or review.

**Match the project's styling system** (Tailwind, plain CSS, CSS Modules, CSS-in-JS) and express every fix in it; the values matter, the syntax doesn't. Don't add a global stylesheet to a Tailwind codebase or utility classes to a styled-components one. Tokens (z-index scale, font weights) go wherever the project keeps design tokens.

Motion (easing, durations, press `scale(0.96)`, hover transitions, `transition: all`, `will-change`, theme-switch transitions, per-frame React animation) lives in `ui-motion`. Shadows, borders, gradients, elevation, image outlines, and dark-mode surfaces live in `ui-design`.

## Hard rules

1. **Hover enhances, never enables.** Anything reachable only through hover doesn't exist on a phone.
2. **Media queries over device sniffing.** Gate by capability: `(hover: hover)`, `(pointer: fine|coarse)`, `env()`, `dvh`. Never branch on user agent or screen width to guess touch.
3. **Touch and mouse are not exclusive.** iPads with trackpads, touchscreen laptops: write for both at once.
4. **Never disable zoom.** `user-scalable=no` / `maximum-scale=1` lock low-vision users out (Safari ignores them for pinch anyway). Fix the 16px input size, the actual cause.
5. **CSS or a meta tag before JavaScript.** Almost every mobile fix is one declaration; a `useIsTouchDevice()` hook to hide hover is the wrong tool.
6. **Every fix ships with its why, applied where the why applies.** `user-select: none` on a button is correct; on body text it's a defect.
7. **Real hardware before "done".** Emulation can't reproduce sticky hover, tap delay, rubber-banding, safe areas, or the keyboard. Fixing only what desktop Chrome shows you ships all of them.
8. **Build on accessible primitives.** Base UI by default; if it doesn't fit the codebase use what does, but focus management, ARIA, and keyboard support are non-negotiable.

## Text and layout stability

Read [references/polish-recipes.md](references/polish-recipes.md) when extending a hit area, styling a scoped scrollbar, adding anchor offsets, safe areas, truncation, or a z-index scale. Read [references/review-checklist.md](references/review-checklist.md) when reviewing a component or page, or hunting why it "feels off".

- **Antialiased font smoothing, always:** `-webkit-font-smoothing: antialiased` on `body`. Subset custom fonts to the characters, alphabets, and languages actually used; nothing else ships.
- **Never change font weight on interaction.** Weight changes on hover or selection reflow text and shift layout. Keep weight constant; signal state with color (`.tab { font-weight: 500 } .tab.selected { color: var(--color-primary) }`). Define weights as variables (`--font-weight-normal: 400; --font-weight-medium: 500; --font-weight-semibold: 600; --font-weight-bold: 700;`) so they're adjustable globally.
- **Tabular numbers for anything that changes.** Counters, prices, timers get `font-variant-numeric: tabular-nums` so digits keep equal width and don't jiggle.
- **Real typographic characters:** `…` not `...`, `’` not `'`, `“` `”` not straight quotes. `text-wrap: balance` on `h1`–`h3`. Larger text wants tighter letter spacing, smaller text looser: pair each font size with its tracking (a `Text` component is the right home; exact values are font-dependent).
- **Preload fonts** so text doesn't reflow when they land; fonts loaded without preload cause visible FOUT on every cold load.
- **Zero layout shift from dynamic content.** Skeletons, image placeholders, and dynamic areas get hardcoded dimensions (or `aspect-ratio`) matching the loaded state. Images and videos always declare dimensions. Reserve space for async content with skeletons.
- **No flash on refresh.** Interactive components render their real state on first paint: persist state to `localStorage`/`sessionStorage`, hydrate correctly under SSR, set initial state before render. Never default-then-correct.
- **Empty states teach.** An empty list is a first-run screen, not an error: short headline, one line of explanation, primary action. Size it to the filled state's dimensions so completing the action doesn't shift layout.
- **Fixed z-index scale.** No arbitrary `z-index: 9999`; define a scale and use only its steps. Better: avoid z-index where possible with `isolation: isolate` (or `position: relative`) so children can't leak above unrelated UI.
- **Align by eye, not by math.** Mathematical centering often reads off-center:
  - Button with trailing icon: icon-side padding = text-side padding − 2px (`pl-4 pr-3.5`).
  - Play triangles: shift `2px` toward the pointed side (`margin-left: 2px`).
  - Asymmetric icons (stars, arrows, carets): fix the SVG viewBox/path itself; a `1px` margin is the fallback.
- **Native scrollbar on the page.** Never replace it. Customize only inside small scrollers like code blocks: `8px` wide, `rgba(0,0,0,0.2)` thumb, `4px` radius.

## Interaction states and focus

- **Hover must not change layout** (see font weight above). Transition details are in `ui-motion`.
- **Gate every `:hover` behind `@media (hover: hover) and (pointer: fine)`.** Touch applies `:hover` on tap and leaves it stuck. Hover-only row actions render always at lower emphasis, or reveal on `:focus-within` too.
- **Focus:** style `:focus-visible`, not bare `:focus`; never remove the outline (no `outline: none` without a replacement), and don't recolor it to anything but grey, black, or white; colored outlines clash with the interface.
- **Selection:** if you style `::selection` at all, keep a subtle tint; don't make selected text unreadable or brand-colored for its own sake.
- **Minimum hit areas:** `44×44px` for touch/mobile (a finger pad is ~44px), at least `40×40px` on desktop. Extend small controls with a pseudo-element (`inset: -10px` on a 24px icon) or `min-width/min-height: 44px`; don't scale the visual. Two hit areas must never overlap: on collision shrink both to the largest non-colliding size, or add spacing.
- **Press feedback on press, not release.** `:active` (or `pointerdown` if JS drives the visual), `transform: scale(0.97)`, 100–160ms `ease-out`, never below `scale(0.95)`. The action still fires on `click` so users can slide off to cancel. Use existing easing tokens (ui-motion).
- **Decorative elements are inert:** `pointer-events: none` on anything purely visual so it can't steal clicks; `user-select: none` on code-built illustrations.
- **Tooltips open after a 200ms delay** so incidental cursor travel doesn't flicker them on. **Warm state**: once one tooltip is open, neighbors open instantly with no delay and no entrance animation; clear ~300ms after the last closes.
- **Submenus get a triangular safe area** (`clip-path`) so diagonal travel toward the submenu doesn't close it.

## Forms

Read [references/forms.md](references/forms.md) when writing any form, input, button, or validation.

- **Every input has an associated label**; clicking the label focuses the input.
- **Right `type` and `inputmode`** (`email`, `tel`, `url`, `number`, `search`, `password`; `numeric` for codes, `decimal` for amounts): correct mobile keyboard, validation, and autofill for free. `autocomplete="one-time-code"` on verification inputs; `autocapitalize="none"` + `autocorrect="off"` on usernames and codes; `enterkeyhint` says what return does.
- **Input font-size ≥ 16px, always** (inputs, textareas, selects). Under 16px iOS Safari zooms on focus and never zooms back. If the design wants smaller text, keep it 16px under `(pointer: coarse)` (Tailwind `text-base pointer-fine:text-sm`, not a width breakpoint like `md:`, which still zooms on an iPad) and never go below 16px on touch; `text-sm` alone on an input is the bug.
- **`spellcheck="false"` / `autocomplete="off"`** on usernames, search, codes, slugs. Keep autocomplete on for email, name, address, payment. Disable password-manager overlays (`data-1p-ignore`, `data-lpignore`) where they don't belong.
- **Decorations overlay the input** (absolute, `pointer-events: none`, padding makes room), never siblings, or clicking them does nothing and the border lies about the hit area. Clickable decorations (clear button) refocus the input.
- **Autofocus the first input in a modal, desktop only.** On coarse pointers it pops the keyboard and shoves the layout before the user has read it.
- **Always wrap inputs in a `<form>`; Enter must submit.** Textareas submit on Cmd+Enter (Mac) / Ctrl+Enter.
- **Buttons are `<button>`**, never `<div onClick>` / `<span onClick>` (invisible to keyboards and screen readers).
- **Disable while submitting, with loading text** ("Submitting..."): prevents duplicate requests and says what's happening.
- **Validate late: reward early, punish late.** Validate on blur or submit, never on every keystroke during the first attempt. Once a field has shown an error, validate on change so it clears the moment it's fixed.
- **Errors sit next to the field** that caused them (`aria-invalid`), not in a summary at the top.
- **No dead zones on checkboxes and radios**: label, control, and the gap between them are all clickable; one `<label>` wraps the row.
- **Destructive actions require confirmation.** `confirm()` is the floor; real products use a proper confirmation modal.
- **Prefill everything you can** from the logged-in user and from link context (a "Change username" link lands on a form prefilled with "I'd like to change my username to:").
- **A fixed bottom CTA must not cover the last field**: pad the scroll container by CTA height + safe-area inset.

## Touch, keyboard, and screen readers

Read [references/a11y-patterns.md](references/a11y-patterns.md) when building modals, menus, tooltips, timers, shortcuts, or icon buttons, or running an a11y review.

- **`touch-action: manipulation`** on buttons, links, inputs, `[role="button"]`: removes the 300ms double-tap delay.
- **`touch-action` on gesture surfaces names what the browser keeps**: `pan-y` on a horizontal carousel, `pan-x` on a vertical sheet handle, `none` only on surfaces that own every axis (canvas, slider, drag-to-dismiss), or users can't scroll past them. Native-scroll carousels use `scroll-snap` instead.
- **`-webkit-tap-highlight-color: transparent` once on `html`**, then give every tappable element its own `:active`.
- **`user-select: none` + `-webkit-touch-callout: none` on controls only**, never on `body`: users copy addresses, errors, order numbers.
- **Every icon-only button gets `aria-label`**; illustrations built from divs/CSS get `role="img"` + `aria-label`.
- **Tab order includes only visible elements.** Hide inactive panels with `visibility: hidden` or `inert`; `opacity: 0` / `transform` keep them focusable.
- **Focus never lands off-screen**: `scrollIntoView({ block: 'nearest' })` on focus.
- **Modals move focus in on open and restore it to the trigger on close.** Never strand focus on a removed node.
- **Every animation has a `prefers-reduced-motion` path.**
- **Time-limited actions pause on `visibilitychange`**: a countdown running in a background tab punishes switching away.
- **Show and bind shortcuts per OS**: `Cmd` on macOS, `Ctrl` elsewhere. If a button has a shortcut, surface it in its tooltip.
- **Feedback (errors, confirmations, status) is visible on the page**, never behind hover or inside a modal.

## Mobile web

Read [references/mobile-fixes.md](references/mobile-fixes.md) when building for or reviewing on mobile, a PWA, a sheet, a carousel, a full-screen layout, or when matching a symptom; it is the full fix catalog with code.

- **Height**: `100dvh` for app shells, drawers, bottom-pinned UI; `100svh` for heroes (no mid-scroll layout shift). `100vh` / `h-screen` overflows by the URL bar.
- **Safe areas**: `viewport-fit=cover` + `env(safe-area-inset-*)` padding on fixed headers, tab bars, toasts, sheets, landscape sidebars, for notches and home indicators. Without the meta tag every `env()` is `0`. Fallback `0px` inside `calc`.
- **Keyboard**: `interactive-widget=resizes-content` in the viewport meta so Android resizes like iOS and `dvh` bottom composers stay above the keyboard with no JS.
- **Overscroll**: `overscroll-behavior: none` on `html, body` in apps, `contain` on inner scrollables. Never `touchmove` + `preventDefault()`.
- **Scroll lock**: a dialog primitive that locks scroll (Base UI, Radix, vaul), not `overflow: hidden` on `body`, which iOS ignores for touch.
- **Video**: autoplay needs `muted` + `playsinline` (else no autoplay, or fullscreen). Under reduced motion, no autoplay; show controls. Pause off-screen (see Rendering and loading performance).
- **`theme-color` per `prefers-color-scheme`**, matching the top-of-page color, read from tokens, never invented.
- **`scroll-margin-top`** = sticky header height on anchor targets; cap the header's height.
- **Ship the Baseline** (viewport meta, theme-color, tap highlight, text-size-adjust, overscroll, 16px inputs, touch-action, hover gate) before the first component.

**Output (mobile fixes).** Apply the fixes; the code is the deliverable. Opinionated and brief: say the line and the reason, move on. Then at most a few lines: **What was wrong** (the matched symptom and its one-line why); **What changed** (file and declaration, one line each); **What needs a phone** (which fixes were verified from code and which the user must confirm on hardware, with what to look for). If it can't be verified without a device, say so instead of claiming it's fixed.

## Rendering and loading performance

Read [references/performance-recipes.md](references/performance-recipes.md) when virtualizing a list, preloading fonts/images, pausing off-screen work, or generating static content.

- **Virtualize long lists.** Don't render hundreds of DOM nodes when a handful are visible; lists over ~50 items are virtualized. Rendering a 2,000-row table and wondering why scroll is choppy: virtualize.
- **Preload critical, lazy-load the rest.** Preload fonts and above-the-fold (hero) images; below-the-fold images get `loading="lazy"` so they don't compete with critical resources.
- **Stop work that's off-screen.** Pause looping animations, video, and any resource-intensive operation when not visible (`IntersectionObserver`). Videos playing in background tabs and below the fold waste CPU and battery.
- **Generate static content at build time.** Blog posts, changelogs, docs, anything that doesn't change per request is never fetched at request time.

## Related

`ui-motion` (animation and its performance), `ui-design` (layout, type, color, surfaces), `component-api`, `review`.
