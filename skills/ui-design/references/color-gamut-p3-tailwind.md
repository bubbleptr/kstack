# Gamut, Display P3, Tailwind v4

## The shape of the displayable

OKLCH describes more colors than any screen shows. sRGB is the universal floor; Display P3 (modern phones, Macs, better monitors) is a strict superset, roughly a quarter larger, with its extra room in intense greens, reds, and pinks.

The chroma ceiling swings with hue and lightness. Computed sRGB reference points:

- At L 0.5, violet-purples around H 285 reach C ≈ 0.29, reds around H 25 ≈ 0.20, cyans near H 195 stay under ≈ 0.09: a 3× spread.
- The roomy region migrates: at L 0.7 the peak is in the pinks (H ≈ 330, C ≈ 0.31); at L 0.9 in spring greens (H ≈ 130, C ≈ 0.24).
- Cyan is cramped at every lightness: plan for muted teals in sRGB.

## When a value doesn't fit

Out-of-gamut C gets browser gamut-mapped with results you didn't choose. Keep L and H (the color's identity) and pull C down to the boundary:

```css
/* asks for more red than sRGB has at this lightness */
color: oklch(0.6 0.31 20);

/* largest C that renders faithfully: */
color: oklch(0.6 0.241 20);
```

## Progressive enhancement for P3

Ship the sRGB-safe value unconditionally; layer wider gamut behind a capability query:

```css
.cta {
  background: oklch(0.66 0.2 330);
}

@media (color-gamut: p3) {
  .cta {
    background: oklch(0.66 0.32 330);
  }
}
```

sRGB ceiling here is about 0.30, P3's about 0.33, so 0.32 renders only on wide-gamut screens.

For pre-2023 browsers with no `oklch()` parser, put a hex floor underneath and gate with `@supports`:

```css
.cta {
  background: #d25bcb;
}

@supports (color: oklch(1 0 0)) {
  .cta {
    background: oklch(0.66 0.2 330);
  }

  @media (color-gamut: p3) {
    .cta {
      background: oklch(0.66 0.32 330);
    }
  }
}
```

## Tailwind v4

The bundled palette is authored in oklch; custom theme colors should be too. No hex tokens in `@theme`.

Tokens named `--color-<name>-<step>` inside `@theme` generate the full utility set (`bg-berry-400`, `text-berry-100`, `border-berry-700`):

```css
@theme {
  --color-berry-50:  oklch(0.97 0.007 330);
  --color-berry-100: oklch(0.895 0.042 330);
  --color-berry-200: oklch(0.82 0.103 330);
  --color-berry-300: oklch(0.745 0.189 330);
  --color-berry-400: oklch(0.67 0.251 330);
  --color-berry-500: oklch(0.595 0.231 330);
  --color-berry-600: oklch(0.52 0.195 330);
  --color-berry-700: oklch(0.445 0.151 330);
  --color-berry-800: oklch(0.37 0.105 330);
  --color-berry-900: oklch(0.295 0.063 330);
  --color-berry-950: oklch(0.22 0.03 330);
}
```

Scale construction is in [color-oklch.md](color-oklch.md).

**Opacity suffixes** compose with oklch: `text-berry-600/75` emits `oklch(0.52 0.195 330 / 0.75)`. No separate alpha variables.

**Porting a hex-based theme:**

1. Re-express every `@theme` color token as oklch.
2. Chase down `theme()` calls and component-level hardcoded hex that duplicated those tokens; point them at the converted values.
3. Re-run contrast checks and eyeball dark mode: colors approximated by eye in hex often turn out slightly off once measured perceptually.
4. Where the brand wants more punch than sRGB allows, add the `@media (color-gamut: p3)` layer rather than raising the base token's chroma.
