# Text: scale, rhythm, lines, polish, floors

## Units for text

- `rem`: relative to the root font size; the default for font sizes because it honors the reader's browser settings.
- `em`: relative to the current element's font size; for things that ride along with the text (icon sizing, underline offsets, tracking).
- `%` on `font-size`: behaves like `em`, relative to the parent.
- `px`: frozen; ignores user font-size preferences. Acceptable for hairlines and borders, suspicious on text.

## Build a scale, then obey it

A closed list of sizes the whole product draws from; departures are exceptions needing justification. Ad-hoc per-component values reliably rot as the codebase grows.

Ratio-derived example (1.2 on a 16px root):

```css
:root {
  --step--1: 0.833rem;
  --step-0: 1rem;
  --step-1: 1.2rem;
  --step-2: 1.44rem;
  --step-3: 1.728rem;
  --step-4: 2.074rem;
}
```

Ready-made scales are equally legitimate: Tailwind's `text-xs` through `text-9xl` pairs every size with a tuned default line-height.

**Name steps by role on teams.** `text-lg` describes a size; `text-card-title` or `text-caption` describes a job and polices its own usage. Solo, size names plus written conventions are fine; once several people commit, role names pay for themselves.

## Leading (line-height)

- **Body copy:** `1.5`–`1.6`.
- **Display and headings:** around `1.1`. Default leading leaves large sizes gappy.

Always unitless. `line-height: 1.5` recomputes when the size changes; `line-height: 24px` breaks on the first size tweak or user zoom. Tailwind's `leading-snug` / `leading-normal` / `leading-relaxed` rarely need overrides.

Paragraph spacing: set it to 1× the line-height, not an arbitrary value like 12px, so vertical rhythm stays consistent.

## Tracking and kerning

- **Kerning** is the font's built-in pair adjustment ("To", "Wa"), applied automatically. `font-kerning: none` is a deliberate, unusual act. For large display headlines, metric kerning leaves awkward gaps between certain pairs: enable optical kerning for large type.
- **Tracking** (`letter-spacing`) depends on size and case:
  - Display sizes: slightly negative.
  - Small uppercase labels: slightly positive; default tracking on all-caps looks cramped and amateur.
  - Body text: zero.

```css
.hero-title {
  font-size: var(--step-4);
  letter-spacing: -0.015em;
}

.eyebrow {
  font-size: var(--step--1);
  text-transform: uppercase;
  letter-spacing: 0.06em;
}
```

Track in `em` so the adjustment scales with the text.

## Weight and style

- Light weights are size-dependent: illegible at 11px, elegant at 32px. Don't use light weights small.
- For hover emphasis without layout shift, increase weight via a variable font axis (through `font-weight`) rather than swapping to a fixed heavier weight.
- Italic is for citations and linguistic stress; **bold** is for UI emphasis. Italic as UI hierarchy is a publishing convention in the wrong medium.

## Cropping built-in space with text-box

Fonts reserve space above the cap height and below the baseline, which is why a label never sits dead-center in its button. `text-box` crops it: edge selector `trim-both` / `trim-start` / `trim-end`, plus where each edge lands: `cap` (tight top), `alphabetic` (baseline, tight bottom), `text` (font's text edges, keeps descender room).

```css
.tag           { text-box: trim-both cap alphabetic; } /* optical centering */
.section-title { text-box: trim-start cap; }           /* flush to block above */
.stat-label    { text-box: trim-end alphabetic; }      /* snug bottom */
```

Ship as progressive enhancement: layouts must remain acceptable where the trim never happens.

## Measure

Roughly 45–75 characters per line for sustained reading, ~65 as the target. `max-width: 65ch` (`ch` = width of the font's `0`); Tailwind's `max-w-prose` is exactly `65ch`. A rem-based cap works too; recheck it whenever the body size or family changes, since the same width holds a different character count in a different font.

## Wrap modes

- `text-wrap: balance`: equalizes line lengths. For headings; an 80/20 two-line split looks accidental.
- `text-wrap: pretty`: avoids the orphaned final word. For standfirsts, card blurbs, empty-state copy.
- Long-form paragraphs get neither: browsers cap `balance` at a small line count, and an evened-out paragraph trades away width reading uses.
- `balance` isn't always right: on a 3-line headline it can leave line 1 awkwardly short. Sometimes a manual break wins.

## Overflow protection

- A long unbroken token (URL, hash, compound German noun) escapes its container unless `overflow-wrap: break-word`.
- A short label that wraps mid-phrase ("In<br>progress" in a badge) looks broken: `white-space: nowrap` on badges, buttons, key-value labels; combine with truncation if space is tight.
- For justified or very narrow columns, `hyphens: auto` breaks at dictionary-correct points; it needs a correct `lang`.

## Alignment

`text-align: start` is the default for a reason: the ragged edge gives the eye a landmark. Justification without hyphenation opens rivers; it belongs, if anywhere, in print-like editorial layouts, not interfaces. Centering works for short display text and fails for anything that wraps more than twice.

## Truncation

- One line: `white-space: nowrap`, `overflow: hidden`, `text-overflow: ellipsis` (Tailwind `truncate`).
- Several lines: `line-clamp`.

Truncation is acceptable only when the reader has another route to the whole string: tooltip, `title` attribute, expandable row, detail page. A clipped ID with no way to read the rest is a data-loss bug wearing CSS.

## Casing

Store copy the way a human would type it (sentence case) and let `text-transform` decide whether it renders as "Settings" or "SETTINGS". Casing then changes in one declaration at the next redesign, and screen readers and translation tooling get natural text.

## The right characters

- Curly quotes in prose (“ ” ‘ ’); straight quotes stay in code.
- En dash, no spaces, for spans: `Mon–Fri`, `1998–2004`. The hyphen is for hyphenation.
- Em dash for an interrupting thought, not a pair of hyphens.
- `…` instead of three periods; never type `...` in markup. Use `text-overflow: ellipsis` when truncation should respond to container width.
- `&nbsp;` welds a value to its unit (`42&nbsp;GB`) or a name to its initial.
- `&shy;` marks permitted break points inside a long word.

## Language and direction

- Declare `lang` on the document and on inline language switches (quote style, hyphenation, speech synthesis).
- Right-to-left content gets `dir="rtl"`.
- Use logical properties everywhere a physical one exists: anything with left/right in its name has an `inline-start`/`inline-end` twin.

```css
/* Flow-relative: mirrors itself under dir="rtl" */
.timeline-item { padding-inline-start: 12px; text-align: start; }

/* Physical: wrong the moment direction flips */
.timeline-item { padding-left: 12px; text-align: left; }
```

## Underlines

Never underline non-links: underline is a hyperlink affordance. For links, defer to the font's metrics:

```css
a {
  text-decoration-thickness: from-font;
  text-underline-position: from-font;
}
```

Or tune by hand, in `em`:

```css
.prose a {
  text-decoration-color: color-mix(in oklab, currentColor 40%, transparent);
  text-decoration-thickness: 0.08em;
  text-underline-offset: 0.16em;
  text-decoration-skip-ink: auto;
  transition: text-decoration-color 150ms ease;
}
.prose a:hover { text-decoration-color: currentColor; }
```

`text-decoration-skip-ink: auto` breaks the line around descenders. Dotted (`text-decoration-style: dotted`) is the settled cue for "there's more here": abbreviations, glossary terms, anything with a tooltip.

```css
abbr[title] { text-decoration: underline dotted; }
```

Only the color of a native underline transitions reliably. An underline that grows, slides or thickens on hover must be its own element (border, pseudo-element, background gradient).

## Selection and highlights

- `::selection` is a quiet brand touch; the pair must stay comfortably readable.
- `user-select: none` on button labels and control chrome, where selection reads as malfunction. Check that select-all collects content, not interface.
- Native-feeling apps (Electron, PWAs) usually turn selection off globally and re-enable it only on copyable content.
- `::target-text` styles the passage a `#:~:text=` link jumps to.
- The CSS Custom Highlight API paints arbitrary ranges (search hits, collaborative cursors) without wrapping elements.

## Placeholders and carets

- `::placeholder`: visibly lighter than entered text, but not so faint it fails contrast. Placeholder is never the label.
- `caret-color` tints the insertion bar; replacing the caret outright means rebuilding text editing, almost never a good trade.

## Font smoothing

Once, at the root layout, both vendor properties (Tailwind `antialiased`):

```css
body {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
```

## Ornamental text

- `::first-letter`: drop-cap hook, universal support. `initial-letter` sizes it in line units; missing from Firefox, treat as enhancement.
- `::first-line`: styles exactly the first rendered line.
- `background-clip: text`: masks a gradient or image to glyphs (with transparent text color). Not on headlines as decoration; see the unslop pass.
- `-webkit-text-stroke`: outlines glyphs; works in all modern engines. Seams *inside* stroked letters are overlapping contours in the font (common in variable fonts); no CSS removes them, a static instance does.
- `text-shadow`: follows letterforms rather than the box.

## The floors

Readers zoom, enlarge default font size, and force line spacing; nothing should shatter when they do.

**Mobile inputs ≥ 16px.** Mobile Safari zooms the page into any focused field under `16px`. Use `16px` on small viewports, shrink only at wider breakpoints:

```tsx
<input type="search" className="text-base md:text-sm" />
```

Never patch with `maximum-scale=1` in the viewport meta: Safari ignores it for pinch zoom while Chrome on Android and others enforce it and strip zoom, violating WCAG 1.4.4 (Resize Text).

**Size floors.** Body `16px`. Dense UI text (controls, menus) may drop to `14px` (inputs keep `16px` on mobile), `13px` for captions and metadata, almost never below `12px`.

**Contrast floors (WCAG AA).** Regular text `4.5:1`. Large text (about `24px` and up, or bold from about `18.5px`) may relax to `3:1`. Check both themes; gray-on-gray failures love dark mode. Full contrast method in [color-contrast.md](color-contrast.md).

## Fast diagnoses

| Symptom | Likely cause | Fix |
| --- | --- | --- |
| Layout twitches as a number ticks | Proportional figures | `font-variant-numeric: tabular-nums` |
| Bold looks smudged or too black | Weight never loaded, browser faked it | Load the weight; `font-synthesis: none` |
| Tapping a field on iPhone zooms the page | Input text under `16px` | `16px` on small viewports (`text-base md:text-sm`) |
| Headline breaks seven words / one word | Greedy wrapping | `text-wrap: balance` |
| Paragraph ends on a stranded word | No orphan control | `text-wrap: pretty` |
| Underline slices through g, y, p | Skip-ink or position off | `text-decoration-skip-ink: auto`, `from-font` metrics |
| Long URL blows out a card | No break opportunity | `overflow-wrap: break-word` |
| Weight change does nothing on some machines | Weight set via raw `"wght"` | `font-weight` |
| Cramped in one component, airy in another | One-off sizes and line-heights | Route everything through the scale |
| Reading a paragraph feels like work | Measure far beyond 75 characters | Cap near `65ch` |
| ALL-CAPS strings scattered through JSX | Casing baked into copy | Sentence case + `text-transform: uppercase` |
| Ellipsis with no way to see the rest | Truncation without escape hatch | Tooltip, `title`, or expand control |
| Layout mirrored wrong in Arabic/Hebrew | Physical properties | Logical properties, `text-align: start` |
| Faint gray text failing audits | Contrast under `4.5:1` | Darken until AA passes (`3:1` allowed ≥ ~`24px`) |

## Checklist

- [ ] Font binaries are `woff2`; only used weights shipped
- [ ] `font-synthesis: none`; no fabricated bold or italic
- [ ] Weights and features via `font-weight` / `font-variant-*`; raw tags only for custom axes and numbered sets
- [ ] Every `font-size` traces back to the scale
- [ ] Line-height unitless: ~`1.1` display, `1.5`–`1.6` body
- [ ] Tracking: slightly negative on display, slightly positive on small uppercase, none on body
- [ ] Reading columns capped near `65ch` (45–75 characters)
- [ ] `balance` on headings, `pretty` on short descriptive blocks
- [ ] Live or columnar numbers use `tabular-nums`
- [ ] Every truncation offers a route to the full string
- [ ] Source strings keep human casing; shouting done by CSS
- [ ] Underlines tuned: `from-font` or explicit thickness, offset, skip-ink
- [ ] Mobile inputs ≥ `16px`; no `maximum-scale=1`
- [ ] Sizes at or above the floors; contrast `4.5:1` / `3:1`
- [ ] Smoothing declared once at the root
- [ ] Logical properties plus `lang`/`dir` where direction can vary
- [ ] Any `::selection` override stays readable
