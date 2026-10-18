# Typefaces, formats, and font capabilities

## File formats

Serve `woff2` on the web, full stop. It wraps the font in Brotli compression and every current browser supports it. `woff` compresses less efficiently and matters only if a project genuinely must reach very old browsers. `ttf` and `otf` are raw desktop containers, with no web-oriented compression and noticeably heavier over the wire: finding one in `public/fonts` is a bug to fix, not a preference to respect. Loading strategy (preloading, `font-display`, subsetting) is the project's own decision; on marketing pages, preload (see [marketing-pages.md](marketing-pages.md)).

## The five broad families

Category communicates before any individual font does:

- **Sans-serif**: uniform, unadorned strokes that hold up at small sizes. The default for product interfaces (Helvetica, Roboto, Public Sans).
- **Serif**: small finishing strokes give each line a rail for the eye. At home in essays, editorial pages, anything read at length.
- **Monospace**: one fixed advance width, so vertical columns align. Code, diffs, tabular IDs.
- **Display**: drawn to be seen big: posters, heroes, mastheads. Often too spindly or quirky for paragraph duty.
- **Script**: imitates handwriting. A garnish for rare decorative moments, never a workhorse.

CSS's generic keywords cover these loosely; `cursive` maps to script faces and `fantasy` to decorative ones.

## Optical cuts: Text vs Display

A family whose name contains "Display" is not automatically a display face; many families ship multiple optical cuts. Freight comes as Freight Text, Freight Display and Freight Big: the Text cut has sturdier strokes and a looser fit that survive paragraph sizes, while Big carries hairline contrast that only resolves at poster scale. Match the cut to the size you're actually setting, not to the vibe of its name. A variable font with an `opsz` axis handles this automatically when `font-optical-sizing: auto` is on.

## How many, and how to pair

- One face is a fine answer. Two covers nearly every product. Past three, hierarchy dissolves into noise. Marketing pages get more latitude than application UI.
- Weights and sizes obey the same economy: each exists to mark hierarchy, and stacking too many erases the distinctions they were meant to draw.
- Pair across categories, not within them. A serif for display over a sans for text reads as a considered split between showing and reading. Two nearly interchangeable grotesques (or two humanist sans-serifs) read as an oversight.

## Scope: a fix is not a redesign

No typography fix requires a new typeface. Smoothing, wrap modes, tabular figures, underline tuning all operate inside whatever family the project already uses. Only swap or add a face when the task explicitly asks for one, and never pull in a commercial font to tick a review box.

When a face change *is* the task, two honest options:

```css
/* Native feel: hand rendering to the operating system */
body {
  font-family: system-ui, sans-serif;
}

/* Licensed brand face, backed by fallbacks with similar proportions */
body {
  font-family: "Graphik", "Helvetica Neue", Arial, sans-serif;
}
```

A branded face is a business decision as much as a visual one; whatever is chosen, the stack behind it must degrade gracefully. Declare a fallback stack designed to match the webfont's x-height and weight: an unstyled fallback causes layout shift on load.

## Why equal sizes look unequal

`font-size` sets the height of the em box, not of any visible letter. Within that box every design makes its own choices: **x-height**, **cap height**, where the **baseline** sits, how far **ascenders** and **descenders** reach. A face with a generous x-height fills more of its box and reads bigger than a small-x-height face at the same declared size, which is why swapping families often means re-tuning sizes, and why fallback fonts can jolt the layout when they render first.

## Static vs variable

A **static** font binds one weight and style to one file (Light, Regular, Semibold, Bold = four binaries). A **variable** font packs a continuous design space into one file, so `font-weight: 473` renders a real 473.

Neither wins by default. A project using exactly one or two weights often ships fewer bytes with static files. A project spanning many weights, an optical-size axis or bespoke axes almost always comes out ahead with a variable file. Load only the weights you actually use: shipping 8 weight/style combinations when 3 are used adds page weight and delays render for nothing.

## Axes

| Tag | Axis | What it varies |
| --- | --- | --- |
| `wght` | Weight | Stroke heaviness, the territory of `font-weight` |
| `wdth` | Width | How condensed or extended the glyphs are |
| `opsz` | Optical size | Contrast, spacing and detail retuned per display size |
| `slnt` | Slant | Oblique angle |
| `ital` | Italic | Toggles true italic forms |
| custom | — | Anything the designer invents, in uppercase tags |

Fraunces exposes `wght`, `opsz`, `SOFT` (rounds the corners) and `WONK` (deliberately irregular alternates). Manrope's variable build exposes `wght` alone. A font exposes only the axes it was built with.

## The property beats the raw tag

Whenever CSS offers a dedicated property, it wins over `font-variation-settings` / `font-feature-settings`. The property degrades correctly (`font-weight: 620` still bolds a fallback font), whereas a raw `"wght"` tag addressed at a font that never loaded does nothing.

```css
/* Right: registered axes go through their properties */
.masthead {
  font-weight: 620;
  font-optical-sizing: auto;
}

/* Right: a custom axis has no property, so the tag is the only door */
.pull-quote {
  font-variation-settings: "SOFT" 80;
}

/* Wrong: fallback fonts ignore this entirely */
.masthead {
  font-variation-settings: "wght" 620;
}
```

## Forbid synthesized styles

Request an unloaded weight or style and the browser manufactures one: thickened strokes for fake bold, sheared glyphs for fake italic. Turn it off so a missing file announces itself as an obvious regression:

```css
html {
  font-synthesis: none;
}
```

## OpenType features

Features behave identically in static and variable fonts; a font offers only what its designer put in.

- `tnum`: tabular figures; every digit occupies the same width
- `lnum` / `onum`: lining versus old-style figures
- `zero`: slashed zero, so `0` can't impersonate `O`
- `frac`: diagonal fractions from sequences like 1/2
- `liga`: standard ligatures (fi)
- `ss01`–`ss20`: stylistic sets, numbered grab-bags of alternates
- `cv01`–`cv99`: character variants, per-glyph alternates

```css
/* Right: dedicated properties exist for these */
.countdown { font-variant-numeric: tabular-nums; }
.serial    { font-variant-numeric: slashed-zero; }

/* Right: numbered sets have no property */
.wordmark  { font-feature-settings: "ss02" 1; }
```

Tabular figures: any value that updates in place (timers, live prices, counters) shoves its neighbors on every change unless digits share one width. Number columns (prices, data tables) also get `tabular-nums`; proportional figures don't stack.

## Small caps, superiors, inferiors

- **Small caps**: request real ones with `font-variant-caps: small-caps` rather than shrinking uppercase text.
- **Superior / inferior figures** (m², CO₂): `font-variant-position: super` / `sub` selects genuine glyphs instead of scaled-and-shifted imposters. Default `<sup>` is too large and throws off line spacing; size and baseline-shift footnote markers manually when real superiors aren't available.

Both silently require the glyphs to exist in the font; verify before relying on them.

## Reading the numbered slots

`ss01` in one family restyles the digits, in another it flattens terminals. The number is a slot, not a meaning. Consult the foundry's specimen or feature docs before enabling one, and leave a comment saying what the slot does in this particular font.
