# Surface recipes

Shadow and color tokens belong wherever the project keeps design tokens (Tailwind `@theme` / config, `:root` variables, or the theme object).

## Nested radii

```css
/* Derived: corner gap stays even all the way around */
.dialog {
  border-radius: 16px;
  padding: 6px;
}
.dialog-body {
  border-radius: 10px; /* 16 − 6 */
}

/* Pinched: child repeats the parent's radius */
.dialog,
.dialog-body {
  border-radius: 16px;
}
```

```tsx
// Tailwind: subtract the padding when picking the child's radius
<section className="rounded-3xl p-3">   {/* 24px outer, 12px gap */}
  <article className="rounded-xl">      {/* 24 − 12 = 12px ✓ */}
    ...
  </article>
</section>
```

Past a `24px` gap between layers, the surfaces read as independent; pick each radius on its own merits.

## Depth without borders

Three ingredients: a 1px spread that stands in for the border, a tight crisp layer just beneath, a wider soft layer for ambient depth.

```css
:root {
  --elevation-raised:
    0 0 0 1px rgb(0 0 0 / 0.05),
    0 1px 3px rgb(0 0 0 / 0.05),
    0 4px 10px -4px rgb(0 0 0 / 0.05);
  --elevation-raised-hover:
    0 0 0 1px rgb(0 0 0 / 0.07),
    0 1px 3px rgb(0 0 0 / 0.07),
    0 4px 10px -4px rgb(0 0 0 / 0.08);
}

/* Hook into however the project switches themes:
   a class, a data attribute, or prefers-color-scheme */
.dark {
  --elevation-raised: 0 0 0 1px rgb(255 255 255 / 0.09);
  --elevation-raised-hover: 0 0 0 1px rgb(255 255 255 / 0.14);
}
```

Hover transitions only the shadow:

```css
.panel {
  box-shadow: var(--elevation-raised);
  transition: box-shadow 150ms ease-out;
}
.panel:hover {
  box-shadow: var(--elevation-raised-hover);
}
```

The lone ring works anywhere a real hairline border sits awkwardly against its background:

```css
box-shadow: 0 0 0 1px rgb(0 0 0 / 0.07);
```

**Shadow** when the element reads as raised: cards, bordered button styles, menus, popovers, dialogs, anything travelling across mixed or image backgrounds, hover/focus lift.
**Border** when the line separates: row dividers, table gridlines, form-field edges (a visible outline is an accessibility feature; converting it costs accessibility), hairline rules in dense layouts.

## Separation borders

- Light mode 1px borders: `rgba(0,0,0,0.08)` recedes; solid `#e0e0e0` looks pasted on.
- Dark mode 1px borders: the opposite. Solid (`#2a2a2a`-style) sits quietly; `rgba(255,255,255,0.1)` glows.

Hairlines on high-density screens:

```css
:root {
  --border-hairline: 1px;

  @media only screen and (min-device-pixel-ratio: 2),
    only screen and (min-resolution: 192dpi) {
    --border-hairline: 0.5px;
  }
}

.divider {
  border-bottom: var(--border-hairline) solid var(--gray-6);
}
```

## Gradients and fades

- **Eased over linear.** Linear gradients between solid colors band. Use multiple stops following an easing curve (generator: https://larsenwork.com/easing-gradients/).
- **Mask over gradient overlay.** A mask composites with any content or background; an overlay matches one background color.

  ```css
  .fade-bottom {
    mask-image: linear-gradient(to bottom, black 80%, transparent);
  }
  ```

- **No fades on scrollable content.** It restricts the viewable area and cuts off what the user is scrolling to read.

## Image outlines

```css
img {
  outline: 1px solid rgb(0 0 0 / 0.1);
  outline-offset: -1px; /* pulled inside, costs no layout space */
}

.dark img {
  outline-color: rgb(255 255 255 / 0.1);
}
```

```tsx
<img
  alt={alt}
  src={src}
  className="-outline-offset-1 outline-1 outline-black/10 dark:outline-white/10"
/>
```

Fixed, not themeable: black 10% light, white 10% dark. Palette neutrals (`outline-slate-*`, `outline-zinc-*`, `outline-neutral-*`, `#111827`) and accent colors are off the table; a tinted line picks up the surface behind the image and looks like grime. `outline`, not `border` (no effect on box size), inset with `outline-offset: -1px`.

## Dark mode surfaces

A numbered scale as CSS variables, flipped per theme:

```css
:root {
  --gray-1: #fafafa;
  --gray-2: #f5f5f5;
  --gray-12: #171717;
}

[data-theme="dark"] {
  --gray-1: #171717;
  --gray-2: #1f1f1f;
  --gray-12: #fafafa;
}
```

In Tailwind, don't hand-write `dark:` overrides per component; point at the variables:

```css
/* Good: variables flip automatically */
.button {
  background: var(--gray-12);
  color: var(--gray-1);
}

/* Avoid: manual overrides everywhere */
.button {
  @apply bg-gray-900 dark:bg-gray-100;
}
```

(New scale values are authored in oklch; see [color-oklch.md](color-oklch.md).)

## Symptoms and fixes

- **Corners pinch where a rounded child meets its parent**: child copied the radius; derive it (outer − padding).
- **Card looks stamped on rather than raised**: solid depth border; swap in the layered shadow token.
- **Cards disappear in dark mode**: light shadow stack still active; collapse to the white ring.
- **Shadow hover feels laggy or heavy**: transitions `all`; scope to `box-shadow`.
- **Stripes across a large gradient**: banding; add eased intermediate stops.
- **Fade only looks right on one background**: it's an overlay; rebuild with `mask-image`.
- **Fade over a scrolling list**: remove it.
- **Image edges look dirty**: tinted outline; pure black/white at 10%.
- **`dark:` overrides in every component**: flip a numbered variable scale at the theme root.

## Checklist

- [ ] Inner radii derived (outer − padding) wherever rounded surfaces nest
- [ ] Cards, buttons, containers signal depth with shadow tokens, not solid borders
- [ ] Separation borders intact, alpha in light / solid in dark, hairline on high-DPI
- [ ] Dark theme collapses every shadow stack to a single white ring
- [ ] Large gradients eased; fades via `mask-image`; nothing fading over a scroll area
- [ ] Every image has the 1px inset outline, pure black/white at 10%
- [ ] Themes switch by flipping variables, not per-component `dark:` classes
