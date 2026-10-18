---
name: ui-interaction
description: "Make forms, controls, and touch/keyboard/screen-reader interaction feel right, and make a web app feel native on a phone. Use when building or reviewing any form, input, textarea, search field, login/signup, settings or checkout page, button, checkbox, submit handler, modal, tooltip, menu, carousel, bottom sheet, video, timer, PWA, or full-screen mobile layout. Triggers on: validation timing, error messages, double submit, Enter/Cmd+Enter submit, autofocus, autocomplete, 1Password, iOS input zoom, focus ring, focus trap, tab order, aria-label, a11y, screen reader, reduced motion, keyboard shortcuts, tap targets, sticky hover, tap highlight, 300ms delay, 100vh/dvh, notch and safe areas, keyboard covering inputs, pull-to-refresh, long-press selection, theme-color, playsinline, or \"works in Chrome but feels wrong on my phone\"."
---

# UI Interaction

Many users reach the interface through a thumb, a keyboard, or a screen reader, never a mouse. UI that only works with a fine pointer and perfect vision is broken, however polished. These rules are the baseline, not enhancements. Write all CSS in the project's existing styling system; the values matter, the syntax doesn't.

## Hard rules

1. **Hover enhances, never enables.** Anything reachable only through hover doesn't exist on a phone.
2. **Media queries over device sniffing.** Gate by capability: `(hover: hover)`, `(pointer: fine|coarse)`, `env()`, `dvh`. Never branch on user agent or screen width to guess touch.
3. **Touch and mouse are not exclusive.** iPads with trackpads, touchscreen laptops: write for both at once.
4. **Never disable zoom.** `user-scalable=no` / `maximum-scale=1` lock low-vision users out (Safari ignores them for pinch anyway). Fix the 16px input size, the actual cause.
5. **CSS or a meta tag before JavaScript.** Almost every mobile fix is one declaration; a `useIsTouchDevice()` hook to hide hover is the wrong tool.
6. **Every fix ships with its why, applied where the why applies.** `user-select: none` on a button is correct; on body text it's a defect.
7. **Real hardware before "done".** Emulation can't reproduce sticky hover, tap delay, rubber-banding, safe areas, or the keyboard. Fixing only what desktop Chrome shows you ships all of them.
8. **Build on accessible primitives.** Base UI by default; if it doesn't fit the codebase use what does, but focus management, ARIA, and keyboard support are non-negotiable.

## Forms

Code: [references/forms.md](references/forms.md) — read when writing any form, input, button, or validation.

- **Every input has an associated label**; clicking the label focuses the input.
- **Right `type` and `inputmode`** (`email`, `tel`, `url`, `number`, `search`, `password`; `numeric` for codes, `decimal` for amounts): correct mobile keyboard, validation, and autofill for free. `autocomplete="one-time-code"` on verification inputs; `autocapitalize="none"` + `autocorrect="off"` on usernames and codes; `enterkeyhint` says what return does.
- **Input font-size ≥ 16px, always** (inputs, textareas, selects). Under 16px iOS Safari zooms on focus and never zooms back. If the design wants smaller text, keep it 16px under `(pointer: coarse)` (Tailwind `text-base pointer-fine:text-sm`, not a width breakpoint like `md:`, which still zooms on an iPad) and never go below 16px on touch; `text-sm` alone on an input is the bug.
- **Visible focus**: style `:focus-visible`, not bare `:focus`; never `outline: none` without a replacement.
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

## Touch and pointer

Full fix catalog with code: [references/mobile-fixes.md](references/mobile-fixes.md) — read when building for or reviewing on mobile, a PWA, a sheet, a carousel, a full-screen layout, or when matching a symptom.

- **Gate every `:hover` behind `@media (hover: hover) and (pointer: fine)`.** Touch applies `:hover` on tap and leaves it stuck. Hover-only row actions render always at lower emphasis, or reveal on `:focus-within` too.
- **Tap targets ≥ 44px hit area.** A finger pad is ~44px; the visual can stay small (pseudo-element `inset: -10px` on a 24px icon, or `min-width/min-height: 44px`). Two hit areas never overlap; on collision shrink both to the largest non-colliding size, or add spacing.
- **Press feedback on press, not release.** `:active` (or `pointerdown` if JS drives the visual), `transform: scale(0.97)`, 100–160ms `ease-out`, never below `scale(0.95)`. The action still fires on `click` so users can slide off to cancel. Use existing easing tokens (ui-motion).
- **`touch-action: manipulation`** on buttons, links, inputs, `[role="button"]`: removes the 300ms double-tap delay.
- **`touch-action` on gesture surfaces names what the browser keeps**: `pan-y` on a horizontal carousel, `pan-x` on a vertical sheet handle, `none` only on surfaces that own every axis (canvas, slider, drag-to-dismiss), or users can't scroll past them. Native-scroll carousels use `scroll-snap` instead.
- **`-webkit-tap-highlight-color: transparent` once on `html`**, then give every tappable element its own `:active`.
- **`user-select: none` + `-webkit-touch-callout: none` on controls only**, never on `body`: users copy addresses, errors, order numbers.

## Mobile platform layer

- **Height**: `100dvh` for app shells, drawers, bottom-pinned UI; `100svh` for heroes (no mid-scroll layout shift). `100vh` / `h-screen` overflows by the URL bar.
- **Safe areas**: `viewport-fit=cover` + `env(safe-area-inset-*)` on fixed headers, tab bars, toasts, sheets, landscape sidebars. Without the meta tag every `env()` is `0`. Fallback `0px` inside `calc`.
- **Keyboard**: `interactive-widget=resizes-content` in the viewport meta so Android resizes like iOS and `dvh` bottom composers stay above the keyboard with no JS.
- **Overscroll**: `overscroll-behavior: none` on `html, body` in apps, `contain` on inner scrollables. Never `touchmove` + `preventDefault()`.
- **Scroll lock**: a dialog primitive that locks scroll (Base UI, Radix, vaul), not `overflow: hidden` on `body`, which iOS ignores for touch.
- **Video**: autoplay needs `muted` + `playsinline` (else no autoplay, or fullscreen). Under reduced motion, no autoplay; show controls. Pause off-screen (ui-engineering).
- **`theme-color` per `prefers-color-scheme`**, matching the top-of-page color, read from tokens, never invented.
- **`scroll-margin-top`** = sticky header height on anchor targets; cap the header's height.
- **Ship the Baseline** (viewport meta, theme-color, tap highlight, text-size-adjust, overscroll, 16px inputs, touch-action, hover gate) before the first component.

## Keyboard and screen reader

Patterns with code: [references/a11y-patterns.md](references/a11y-patterns.md) — read when building modals, menus, tooltips, timers, shortcuts, or icon buttons, or running an a11y review.

- **Every icon-only button gets `aria-label`**; illustrations built from divs/CSS get `role="img"` + `aria-label`.
- **Tab order includes only visible elements.** Hide inactive panels with `visibility: hidden` or `inert`; `opacity: 0` / `transform` keep them focusable.
- **Focus never lands off-screen**: `scrollIntoView({ block: 'nearest' })` on focus.
- **Modals move focus in on open and restore it to the trigger on close.** Never strand focus on a removed node.
- **Every animation has a `prefers-reduced-motion` path.**
- **Time-limited actions pause on `visibilitychange`**: a countdown running in a background tab punishes switching away.
- **Show and bind shortcuts per OS**: `Cmd` on macOS, `Ctrl` elsewhere. If a button has a shortcut, surface it in its tooltip.
- **Feedback (errors, confirmations, status) is visible on the page**, never behind hover or inside a modal.

## Tooltips and menus

- **Tooltips open after a 200ms delay** so incidental cursor travel doesn't flicker them on.
- **Warm state**: once one tooltip is open, neighbors open instantly with no delay and no entrance animation; clear ~300ms after the last closes.
- **Submenus get a triangular safe area** (`clip-path`) so diagonal travel toward the submenu doesn't close it.

## Output (mobile fixes)

Apply the fixes; the code is the deliverable. Opinionated and brief: say the line and the reason, move on. Then at most a few lines:
- **What was wrong**: the matched symptom and its one-line why.
- **What changed**: file and declaration, one line each.
- **What needs a phone**: which fixes were verified from code and which the user must confirm on hardware, with what to look for. If it can't be verified without a device, say so instead of claiming it's fixed.
