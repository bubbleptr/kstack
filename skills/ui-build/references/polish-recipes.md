# Polish Recipes

## Hit area extension

Grow a small control's touchable region with a pseudo-element that bleeds past its box. Negative `inset` is simplest:

```css
/* 16px dismiss icon, 40px effective target */
.toast-dismiss { position: relative; }
.toast-dismiss::before {
  content: "";
  position: absolute;
  inset: -12px; /* 16 + 12 + 12 = 40 */
}
```

```tsx
// Tailwind
<button className="relative before:absolute before:-inset-3">
  <XIcon className="size-4" />
</button>
```

44px minimum with `max()`:

```css
.icon-button { position: relative; }
.icon-button::after {
  content: "";
  position: absolute;
  width: max(100%, 44px);
  height: max(100%, 44px);
  top: 50%;
  left: 50%;
  translate: -50% -50%;
}
```

If an extended area collides with a neighbor's, shrink it to the largest non-colliding size.

## Font weight tokens and state

```css
:root {
  --font-weight-normal: 400;
  --font-weight-medium: 500;
  --font-weight-semibold: 600;
  --font-weight-bold: 700;
}

/* Bad: layout shift */
.tab:hover { font-weight: 600; }

/* Good: constant weight, color signals state */
.tab { font-weight: 500; }
.tab.selected { color: var(--color-primary); }
```

## Z-index scale

```css
:root {
  --z-dropdown: 100;
  --z-modal: 200;
  --z-tooltip: 300;
  --z-toast: 400;
}
```

Prefer `isolation: isolate` (or `position: relative`) to create a local stacking context instead of raising z-index.

## Scoped scrollbar styling

```css
.code-block::-webkit-scrollbar {
  width: 8px;
  height: 8px;
}
.code-block::-webkit-scrollbar-thumb {
  background: rgba(0, 0, 0, 0.2);
  border-radius: 4px;
}
```

## Anchors, safe areas, truncation

```css
/* Room for the sticky header when scrolling to anchors */
[id] {
  scroll-margin-top: 80px; /* header height */
}

/* Notches and home indicators */
.footer { padding-bottom: env(safe-area-inset-bottom); }
.sidebar { padding-left: env(safe-area-inset-left); }

/* Truncate text in grid cells */
.grid-cell-text {
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
```
