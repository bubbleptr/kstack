# OKLCH: reading, converting, building scales

## Reading a value

```css
color: oklch(0.586 0.222 17.585);          /* a saturated crimson */
color: oklch(0.398 0.066 227.392 / 0.9);   /* deep slate-cyan at 90% opacity */
```

- **L** 0 (black) to 1 (white), perceptually even: 0.5 looks mid-toned regardless of hue.
- **C** from 0 (gray) to around 0.4 in extreme cases; how high it can go depends on L and H (the gamut is lumpy).
- **H** angle 0–360°. Landmarks: ~20 red, ~90 yellow, ~145 green, ~195 cyan, ~260 blue, ~330 pink.
- Opacity goes after a slash, never as a fourth comma-separated argument. `oklch(0.5, 0.1, 200, 0.5)` is invalid.

Three decimals per channel; trim trailing zeros; write `-0` as `0`. Every evergreen browser has shipped `oklch()` since 2023 (Baseline, upward of 96% of users).

## Converting to OKLCH

Value-for-value substitution: the declaration, selector, and surrounding function stay exactly as written; only the color literal changes notation.

**Never convert:**

- Keywords: `currentColor`, `transparent`, `inherit`, `initial`, `unset`.
- Gradient machinery: rewrite the stops, leave the function shape and interpolation hints (`in oklab`, `in srgb`, angle, positions) untouched.
- Third-party configuration that parses hex (charting library, manifest field) gets hex.

**Input:** hex in 3, 6, or 8 digits (`#c00`, `#e11d48`, `#e11d4880`), `rgb()`/`rgba()` in comma or space syntax, `hsl()`/`hsla()` likewise.

```css
/* before */
.alert-title { color: #e11d48; }
.hero        { background: #164e63; }
.callout     { border-color: #fef3c7; }
.badge       { background: rgb(22, 163, 74); }
.divider     { border-top: 1px solid rgba(255, 255, 255, 0.12); }
.link        { color: hsl(268, 60%, 45%); }

/* after */
.alert-title { color: oklch(0.586 0.222 17.585); }
.hero        { background: oklch(0.398 0.066 227.392); }
.callout     { border-color: oklch(0.962 0.058 95.617); }
.badge       { background: oklch(0.627 0.17 149.214); }
.divider     { border-top: 1px solid oklch(1 0 0 / 0.12); }
.link        { color: oklch(0.468 0.202 299.662); }
```

Fully opaque: three-channel form, no `/ 1`.

**Whole-file pass:**

1. Sweep hex literals first, then `rgb()`/`rgba()` and `hsl()`/`hsla()`.
2. Inside gradients, touch only the stop colors.
3. Skip everything on the never-convert list.
4. Keep comments, indentation, declaration order: the diff shows nothing but color values changing.
5. If a converted value is in a contrast-critical pair, sanity-check the pair afterward.

## Step naming

Lightest to darkest: 50, 100, 200 … 900, 950. Eleven steps is Tailwind's full ramp; a nine-step version omits 400 and 600; a five-step version keeps 100/300/500/700/900. 50 is always the near-white tint, 950 the near-black shade.

## Constructing a ramp

1. **Anchor the L endpoints, then space evenly.** Step 50 around L 0.97, step 950 around L 0.22, interpolate linearly. Don't run to L 0 or L 1: chroma collapses at the poles and the endpoints stop reading as the hue.
2. **Shape chroma as a curve, not a constant.** Express each step's C as a fraction of that step's sRGB ceiling: ~30% at the extremes rising to ~85% around 400–500. Tints stay airy, shades stay rich, nothing leaves the gamut.
3. **Clamp against the gamut per step.** `maxC(L, H)` differs at every lightness; compute or look it up per step before applying the fraction. A highly saturated base necessarily loses absolute chroma toward both ends.

Hue stays fixed for all steps. Magenta-pink brand hue (H = 330):

```css
:root {
  --berry-50:  oklch(0.97 0.007 330);
  --berry-100: oklch(0.895 0.042 330);
  --berry-200: oklch(0.82 0.103 330);
  --berry-300: oklch(0.745 0.189 330);
  --berry-400: oklch(0.67 0.251 330);
  --berry-500: oklch(0.595 0.231 330);
  --berry-600: oklch(0.52 0.195 330);
  --berry-700: oklch(0.445 0.151 330);
  --berry-800: oklch(0.37 0.105 330);
  --berry-900: oklch(0.295 0.063 330);
  --berry-950: oklch(0.22 0.03 330);
}
```

Absolute chroma peaks near 400 rather than 500: at this hue the sRGB gamut is widest around L 0.67.

Light tints: build them by reducing chroma in OKLCH, not by dropping opacity; opacity tints go grey and lifeless.

## Families of hues

To make several hues feel like one system (brand + success + warning), equalize:

- **The same L per step number**: every `-500` at the same lightness.
- **The same fraction of each hue's own ceiling**, not the same absolute C. A shared absolute value ("all our 500s use 0.19") over-saturates roomy hues and flattens cramped ones.

```css
:root {
  /* all at L 0.65, each at 70% of its own sRGB ceiling */
  --teal-500:   oklch(0.65 0.08 185);   /* ceiling here ≈ 0.115 */
  --amber-500:  oklch(0.65 0.096 75);   /* ceiling here ≈ 0.137 */
  --violet-500: oklch(0.65 0.155 300);  /* ceiling here ≈ 0.221 */
}
```

## Dark mode by inversion

Never hand-pick a second palette. Keep one scale and swap which steps the semantic roles point at:

```css
:root {
  --surface: var(--berry-50);
  --ink: var(--berry-950);
}

[data-theme="dark"] {
  --surface: var(--berry-950);
  --ink: var(--berry-50);
}
```

Because L is perceptually linear, a pair separated by ΔL reads just as distinct after the flip. Mid-scale steps swap with their mirrors: 200 ↔ 800, 300 ↔ 700.

## Derived states

Hover: `color-mix(in oklch, var(--color) 85%, black)`, not a hand-picked darker hex; hardcoded hexes drift hue unexpectedly.

## The case against HSL ramps

**The hue lies as lightness moves.** HSL blue at fixed hue 240: `hsl(240 65% 25%)` lands at oklch H ≈ 272.6, `hsl(240 65% 55%)` at ≈ 274.3, the pale `hsl(240 65% 88%)` at ≈ 285.2: a ~13° slide toward violet that users see as light steps "going purple".

**The lightness number is decorative.** `hsl(55 100% 50%)` (yellow) and `hsl(255 100% 50%)` (blue-violet) claim identical lightness; one is nearly white-bright, the other dark.

## Auditing an inherited scale for drift

1. Convert every step to oklch.
2. Find the spread between min and max H.
3. Over ~10° is visible drift: keep the mid-tone hue and regenerate the ramp with the procedure above.
