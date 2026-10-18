# Keyboard, Screen Reader & Pointer Patterns: Code

## Labels for icon-only buttons and CSS illustrations

```html
<button aria-label="Close dialog">
  <CloseIcon />
</button>

<button aria-label="Search">
  <SearchIcon />
</button>
```

```jsx
<div
  role="img"
  aria-label="Abstract geometric pattern"
  className="decorative-illustration"
/>
```

## Hidden content leaves the tab order

`opacity: 0` and `transform` keep elements focusable. Use:

```css
.hidden-panel {
  visibility: hidden;
}
```

```jsx
<div inert={!isVisible}>...</div>
```

## Focus never lands off-screen

```jsx
function handleFocus(e) {
  e.target.scrollIntoView({
    behavior: 'smooth',
    block: 'nearest',
  });
}
```

## Modal focus

On open: move focus to the first interactive element, or the dialog itself. On close: return focus to the trigger. Never leave focus on a removed node (it falls back to `<body>` and keyboard users re-tab from the top).

## Reduced motion for video

```jsx
const prefersReducedMotion = window.matchMedia(
  '(prefers-reduced-motion: reduce)'
).matches;

<video
  autoPlay={!prefersReducedMotion}
  controls={prefersReducedMotion}
  muted
  playsinline
/>
```

## Pause timers when the tab is hidden

```js
let timeoutId;
let remainingTime;
let startTime;

document.addEventListener('visibilitychange', () => {
  if (document.hidden) {
    clearTimeout(timeoutId);
    remainingTime -= Date.now() - startTime;
  } else {
    startTime = Date.now();
    timeoutId = setTimeout(callback, remainingTime);
  }
});
```

## Shortcut labels per OS

```js
const isMac = navigator.platform.toUpperCase().indexOf('MAC') >= 0;
const modKey = isMac ? 'Cmd' : 'Ctrl';

// Display: "Save (Cmd+S)" on Mac, "Save (Ctrl+S)" on Windows
```

Bind the matching key, not just the label.

## Tooltip delay

```css
.tooltip {
  transition-delay: 200ms;
}
```

## Tooltip warm state

Once one tooltip has opened, a neighboring trigger opens instantly: no delay, no entrance animation. Clear the shared flag ~300ms after the last tooltip closes.

```jsx
const [isWarm, setIsWarm] = useState(false);

// When any tooltip opens, set warm state
// Clear warm state after 300ms of no tooltip being open
```

## Submenu safe area

The cursor travels diagonally from the parent item to the submenu; without a safe zone it exits the trigger and the submenu closes.

```css
.submenu-trigger::after {
  content: '';
  position: absolute;
  /* Creates a "safe zone" for cursor movement */
  clip-path: polygon(0 0, 100% 0, 100% 100%);
  /* Adjust based on submenu position */
}
```

## Review checklist

- [ ] Hover styles wrapped in `@media (hover: hover) and (pointer: fine)`
- [ ] No functionality reachable only via hover
- [ ] Every tap target ≥ 44px hit area
- [ ] `touch-action: manipulation` on buttons, links, inputs; `none` on custom gesture surfaces
- [ ] Autoplaying videos have `muted` and `playsinline`
- [ ] Icon-only buttons have `aria-label`; code-built illustrations have `role="img"` + `aria-label`
- [ ] Tab order skips hidden content (`visibility: hidden` or `inert`)
- [ ] Keyboard focus scrolls into view; modals move and restore focus
- [ ] Every animation and autoplaying video respects `prefers-reduced-motion`
- [ ] Timers pause on `visibilitychange`
- [ ] Shortcut hints show Cmd/Ctrl per OS
- [ ] Tooltips delayed, with warm state; submenus have a diagonal safe area
- [ ] Feedback visible on the page, not behind hover or in a modal
