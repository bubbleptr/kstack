---
name: ui-motion
description: "Motion for product interfaces: whether to animate, easing, duration, springs, enter/exit, press/hover feedback, reduced motion, and animation performance (transition properties, will-change, GPU compositing, React render loop, theme switches). Use when adding or reviewing motion on modals, popovers, drawers, sheets, toasts, dropdowns, tooltips, buttons, or icon swaps; choosing a cubic-bezier, duration, or spring; or fixing animation that feels slow, janky, choppy, jittery, or 'off'. Triggers: animation, transition, transition: all, easing, ease-out, spring, bounce, keyframes, stagger, fade, slide, scale on press, hover, microinteraction, Framer Motion, motion/react, AnimatePresence, WAAPI, will-change, GPU, fps, frame drops, blur filter, theme switch, prefers-reduced-motion, make it smooth."
---

# UI Motion

Animation in a product interface is feedback, not decoration. Default to less: a fast, well-eased 200ms transition beats an elaborate 500ms sequence every time. When in doubt, don't animate. When you do, easing and duration are decisions, not defaults inherited from a snippet.

**Match the project's stack.** Check what it already uses (CSS transitions/keyframes, Tailwind, Motion via `motion`/`framer-motion`, React Spring, WAAPI) and write in that. Never add an animation library for something CSS transitions can do; never hand-roll spring physics when Motion is installed.

## References

| Read when | File |
| --- | --- |
| Building a modal, drawer, popover, dropdown, tooltip, toast, staggered entrance, exit, icon swap, or press-feedback Button | [references/component-patterns.md](references/component-patterns.md) |
| Picking a custom curve, configuring a spring, or deciding transition vs. keyframes | [references/easing-and-springs.md](references/easing-and-springs.md) |
| Animation is janky/choppy, touching `will-change`, `transition` property lists, React per-frame updates, Motion transforms, theme switching, or writing the reduced-motion code | [references/motion-performance.md](references/motion-performance.md) |
| Motion looks off and you need a symptom → fix, or reviewing animation code | [references/debugging-and-review.md](references/debugging-and-review.md) |

## 1. Should this animate at all?

Ask how often the user triggers it:

- **100+ times a day** (command menu, keyboard navigation, tab switching in a power tool) → **no animation.** Speed beats smoothness; Raycast doesn't animate its menu for this reason.
- **Keyboard-initiated** (arrow-key list navigation, shortcuts, focus movement) → **never animate.** Animation makes the UI feel disconnected from fast input.
- **Occasional** (modal, dropdown, drawer) → standard animation, under 300ms.
- **Rare or first-time** (onboarding, empty state, success celebration) → can be more elaborate.
- **Marketing page vs. product:** marketing may use longer, showier motion; product UI stays fast and purposeful.

Animate only when it adds information: spatial continuity (where did this come from?), state feedback (did my click register?), or cause → effect. If it answers no question, cut it.

## 2. Which easing?

1. **Entering or exiting the screen** (modal, dropdown, toast, tooltip) → `ease-out`. The fast start reads as instant response.
2. **Already on screen, moving or resizing** (reorder, layout shift, slider) → `ease-in-out`. Natural accelerate-then-brake.
3. **Hover / color / shadow** → `ease` (or a soft ease-out), 150ms.
4. **Constant-speed only** (marquee, spinner, progress-as-time) → `linear`. A hold-to-delete ring is linear because it *is* time.
5. **`ease-in` alone → almost never.** The slow start delays feedback and feels sluggish. Only legitimate as the exit half of a pair (exit `ease-in`, enter `ease-out`).

Built-in CSS keywords are weak. Prefer custom curves; default enter curve `cubic-bezier(0.32, 0.72, 0, 1)`. Full weak → strong library in [easing-and-springs.md](references/easing-and-springs.md).

Use a **spring** instead of a curve when motion is gesture-driven or interruptible (drag-to-dismiss, sheets, anything following a finger): springs preserve velocity when interrupted; keyframes restart from zero. Default `{ type: "spring", duration: 0.5, bounce: 0 }`. Add bounce (0.1–0.3, never more) only for playful, physical interactions.

## 3. What duration?

| What | Duration |
| --- | --- |
| Hover, press, color/shadow changes | 100–150ms |
| Tooltips, dropdowns, popovers | 150–250ms |
| Modals, drawers, toasts | 200–300ms |
| Page-level transitions | 300–400ms |

- **UI animations stay under 300ms.** If it feels slow, it is slow: cut duration before anything else.
- **Exits run ~20–30% faster than enters** (enter 250ms, exit 180ms). The user already decided to leave.
- **Bigger elements and longer travel get more time**; a full-screen sheet needs more than a tooltip.
- Perceived speed is real UX: a snappy 180ms dropdown makes the whole app feel faster; a spinner that feels slow should spin faster.

## 4. Rules applied every time

1. **Animate only `transform` and `opacity`** (plus `filter`/`clip-path`, which also composite). They skip layout and paint. Never animate `width`, `height`, `margin`, `padding`, `top/left`, or box-shadow spread: use `scale`/`translate` and cross-fades. Keep `filter: blur()` under 20px (expensive, worst on Safari) and off frequently-animating elements.
2. **Name the transitioned properties; never `transition: all`.** Tailwind's bare `transition` class is not `all`, but it covers a broad default list (colors, opacity, shadow, transform, filters), so theme switches animate colors and shadows by accident. Use the narrow utilities (`transition-transform`, `transition-opacity`) or an explicit list.
3. **Interactive state gets transitions; one-shots get keyframes.** Transitions retarget from the live position when the user reverses mid-animation; keyframes restart and look broken. `@keyframes` only for enter sequences and loaders that always run to completion.
4. **Never enter from `scale(0)`.** Start at `scale(0.95)` (`0.97` for small elements) with `opacity: 0`; elements always keep a visible shape.
5. **Scale from the trigger.** Popovers, dropdowns, tooltips set `transform-origin` to the trigger side (`var(--radix-popper-transform-origin, var(--transform-origin, top))` in Radix/Base UI), not center.
6. **Paired elements share timing.** Modal + overlay, drawer + backdrop, tooltip + arrow get identical duration and easing; mismatched timing reads as broken.
7. **Press feedback: `scale(0.96)` on `:active`**, 150ms ease-out *transition* so a mid-press release glides back. Never below `0.95` (looks exaggerated/cartoonish). Skip it on dense or high-frequency controls; give the Button an opt-out prop.
   ```css
   .pressable { transition: scale 150ms ease-out; }
   .pressable:active { scale: 0.96; }
   ```
8. **Hover:** transition only the properties that change (e.g. `color, background-color, box-shadow`), ~150ms ease-out. Hover must never change layout. Gate hover motion to `@media (hover: hover) and (pointer: fine)`: touch fires hover on tap (Tailwind v4 `hover:` does this already).
9. **Exits are quieter than enters.** Attention has moved on. Fade with a small directional nudge (`translateY(-12px)`, opacity 0, ~150ms ease-in), not a full-height slide, and not an instant `display: none` that severs spatial context.
10. **Don't animate initial page-load state.** Something sitting at rest when the page paints has no reason to play an entrance. In Motion, `initial={false}` on `AnimatePresence` for toggles/tabs/icon swaps, but not on genuine first-run entrances (staggered hero, loading state), where it would skip the entrance. Hard-refresh to confirm nothing that should animate went static.
11. **Respect `prefers-reduced-motion`, always.** Every animation ships with its reduced-motion path, not as a follow-up. Disable all of it, including opacity and color fades; no exceptions.
    ```css
    @media (prefers-reduced-motion: reduce) {
      .modal { animation: none; transition: none; } /* no !important */
    }
    ```
    In Motion, gate `initial`/`animate` with `useReducedMotion()` and pass `initial={false}` / static values when it returns true. Replace autoplaying video with a play button.
12. **Disable transitions during theme switches.** Otherwise every color on the page tweens independently.
13. **Keep per-frame animation out of React state.** A re-render per frame drops frames; use refs + direct style writes, WAAPI, or the library's loop. Don't animate CSS variables consumed deep in a tree: every change recalculates styles for all descendants.
14. **`will-change` is a targeted fix, not seasoning.** Only `transform`, `opacity`, `filter` (compositor properties), only on elements that animate, only after observing first-frame stutter or ~1px start/end jitter (Safari is the usual offender). Never `will-change: all`, never globally: each layer costs GPU memory.
15. **Pause looping animations off-screen** (spinners, shimmer, marquees) with `IntersectionObserver` or `animation-play-state: paused` to save CPU/battery. General off-screen work pausing (video, heavy operations) lives in `ui-engineering`.
16. **CSS vs. JS engine:** CSS transitions/animations run off the main thread and suit simple predetermined changes; JS engines (Motion, React Spring) are required for dynamic values, springs, and gestures. Pick the simplest tool that supports the interaction.

## Calibration

Don't ship an animation in the same sitting you wrote it; come back with fresh eyes, since motion is refined over days, not hours. Use the product daily: animations that become annoying through repetition are the ones to remove. Users should finish their task, not admire the transition.

## Related

`ui-engineering` (hit areas, layout shift, rendering/list/loading performance), `ui-interaction` (touch, a11y), `ui-design` (surfaces, color), `ui-review` (full UI review).
