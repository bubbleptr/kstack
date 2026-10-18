# Motion Debugging and Review

## Symptom → fix

| Symptom | Fix |
| --- | --- |
| Animation feels off, can't say why | Screen-record it and scrub frame by frame; details invisible at full speed show up |
| Shaky/jittery ~1px at start or end | `will-change: transform` on the animated element |
| Hover state flickers | The hover moved the element out from under the cursor: animate a child, keep the parent's hover area static |
| Transition between two states looks disconnected | Add a subtle `blur(2px)` during the transition to bridge the states (well under 20px) |
| Hover animations firing on phones | Wrap in `@media (hover: hover) and (pointer: fine)` |
| Everything transitions when the theme toggles | Disable transitions during the swap ([motion-performance.md](motion-performance.md)) |
| Small icon buttons hard to hit | Expand the hit area, don't scale the visual (recipe in `ui-engineering`) |
| Sequential tooltips feel sluggish | Delay + animate only the first; siblings open instantly (`data-instant`) |
| Spinner feels slow though load time is fixed | Spin it faster; perceived speed is real UX |

### Hover flicker fix

```html
<div class="card"><div class="card-inner">…</div></div>
```

```css
.card-inner { transition: transform 200ms ease; }
.card:hover .card-inner { transform: translateY(-20%); } /* parent hover area never moves */
```

## Common mistakes

| Smell | Fix |
| --- | --- |
| `transition: all 400ms ease-in` | Name the properties, 200ms, `ease-out` |
| Bare Tailwind `transition` class everywhere (broad default property list) | Explicit property list |
| Enter from `scale(0)` or `translateY(100vh)` | `scale(0.95)` + `opacity: 0`, small travel |
| Animating `height`/`width`/`margin` | `transform` + `opacity`; grid-rows or clip tricks for expansion |
| Popover scales from center | `transform-origin` from the trigger |
| Modal and overlay use different durations | Identical duration + easing for paired elements |
| Keyframe animation on an open/close toggle | CSS transition (interruptible) |
| Animating arrow-key navigation / keyboard actions | No animation on keyboard-initiated changes |
| Exit longer or louder than enter | Exit ~20–30% faster, subtler |
| `scale(0.9)` on press | `scale(0.96)`; below `0.95` feels exaggerated |
| `will-change` as a lucky charm on every card | Only on animating elements that showed stutter |
| `useState`-driven drag/scroll handlers | Refs + direct style writes |
| No reduced-motion handling | Add the media query / `useReducedMotion()` the moment you add motion |
| New dependency for a simple fade | Use the project's existing stack |

## Review order

When reviewing animation code, check in order:

1. Should this animate at all (frequency, keyboard-initiated)?
2. Transform/opacity (filter, clip-path) only? No `transition: all`, and no bare Tailwind `transition` where only one or two properties should move? No blur above 20px on animating elements?
3. Easing matches the enter/move/hover rule?
4. Duration under 300ms, exit faster?
5. Interruptible (transition or spring, not keyframes, for toggles)?
6. Origin-aware?
7. Reduced-motion path exists?
8. No per-frame React state; `will-change` only where stutter was observed; theme switch disables transitions; off-screen loops paused?

Report findings as a Before/After markdown table.
