# Motion Performance

## Transition only what changes

With `transition: all` (or Tailwind's bare `transition` class, which compiles to `transition-property: all`) every style change becomes a candidate animation: the browser tracks the whole style object, properties you never meant to tween (padding, shadow, colors) tween whenever they change, and the engine loses the optimizations a narrow, known property list allows.

```css
/* Do: name each property that actually animates */
.menu-trigger {
  transition-property: translate, opacity;
  transition-duration: 120ms;
  transition-timing-function: ease-out;
}

/* Don't: a shorthand that watches everything */
.menu-trigger { transition: all 120ms ease-out; }
```

```tsx
// Do: explicit property list
<a className="transition-[translate,opacity] duration-100 ease-out">
// Don't: bare `transition` is `transition-property: all` in disguise
<a className="transition duration-100 ease-out">
```

Tailwind: `transition-transform` expands to the whole transform family (`transform, translate, scale, rotate`); use it when only transforms move. Mixing in non-transform properties: arbitrary-value form, e.g. `transition-[opacity,filter,scale]`.

## Compositor-only properties

Animate `transform`, `opacity` (and `filter`, `clip-path`). Never animate:

- `width`, `height`, `padding`, `margin`, `top/left`: trigger layout for the whole subtree. For expanding panels, animate `transform: scaleY()`, use a clip or grid-rows trick, or measure and animate once.
- `filter: blur()` above 20px: expensive, especially Safari. Keep blurs subtle and off frequently-animating elements.
- CSS custom properties consumed deep in a component tree: every change triggers style recalculation for all descendants.

## `will-change`

By default the browser waits until motion begins before lifting an element onto its own compositor layer; that just-in-time promotion occasionally eats the opening frame. `will-change` requests the layer up front. Separately, elements can shift ~1px at the start/end of transform animations due to the CPU↔GPU handoff; `will-change: transform` keeps the element on the GPU throughout.

```css
.slide-panel { will-change: transform; }           /* sensible: the panel slides */
.slide-panel { will-change: transform, opacity; }  /* sensible: slide + fade */
.slide-panel { will-change: all; }                 /* wrong: defeats the hint */
.slide-panel { will-change: height, color; }       /* useless: layout/paint anyway */
```

| Rendering path | Properties | Does `will-change` pay off? |
| --- | --- | --- |
| Compositor | `opacity`, `transform`, `filter` (blur, brightness), `clip-path` | Yes: pre-promotion smooths the first frame |
| Paint | `color`, `background`, `border` | No |
| Layout | `width`, `height`, `top`/`left`, `margin` | No |

Browsers manage layers well on their own: reach for it after observing a first-frame hiccup (Safari is the usual offender), never as routine. Each promoted layer holds GPU memory; `will-change` added to every card "for performance" costs layers and fixes nothing.

## React and Motion

Per-frame `useState` is a re-render storm (the whole tree re-renders at pointer speed in `useState`-driven drag/scroll handlers). Drive frame values through refs and direct DOM writes:

```jsx
// Bad: re-render on every frame
const [position, setPosition] = useState(0);

// Good: refs + direct DOM manipulation
const elementRef = useRef(null);
useEffect(() => {
  let frame;
  function animate() {
    elementRef.current.style.transform = `translateX(${position}px)`;
    frame = requestAnimationFrame(animate);
  }
  frame = requestAnimationFrame(animate);
  return () => cancelAnimationFrame(frame);
}, []);
```

Motion: the transform string is hardware accelerated; the shorthand runs on the JS thread. Prefer the string form for elements that animate while the main thread is busy.

```jsx
<motion.div animate={{ transform: "translateX(100px)" }} /> // hardware accelerated
<motion.div animate={{ x: 100 }} />                          // more readable, not accelerated
```

## CSS vs. JS engines

- CSS transitions/animations run off the main thread: smoother under load. Use for simple, predetermined state changes.
- JS engines (Motion, React Spring) run on `requestAnimationFrame`: required for dynamic values, springs, gesture-driven motion.
- Don't pay for a JS animation loop to fade a tooltip.

## Theme switches must not animate

Flipping themes with transitions enabled makes every color tween independently. Disable for the swap, re-enable after paint:

```js
function setTheme(theme) {
  document.documentElement.classList.add('no-transitions');
  document.documentElement.setAttribute('data-theme', theme);
  // Re-enable after paint (double rAF)
  requestAnimationFrame(() => {
    requestAnimationFrame(() => {
      document.documentElement.classList.remove('no-transitions');
    });
  });
}
```

```css
.no-transitions,
.no-transitions * { transition: none !important; }
```

## Reduced motion recipe

```css
.modal { animation: fadeIn 200ms ease-out; }
@media (prefers-reduced-motion: reduce) {
  .modal { animation: none; }   /* transition: none for transitions; no !important */
}
```

```tsx
const reduce = useReducedMotion();
<motion.div initial={reduce ? false : { opacity: 0, y: 20 }} animate={{ opacity: 1, y: 0 }} />
```
