# UI Engineering Review Checklist

## Frequent offenders

| Problem | Remedy |
| --- | --- |
| Bold-on-hover tabs shifting layout | Constant weight; signal with color |
| Timer/price digits jiggling | `font-variant-numeric: tabular-nums` |
| Content jumping as images/skeletons load | Hardcode dimensions or `aspect-ratio` |
| Fonts loaded without preload | Preload; otherwise visible FOUT on every cold load |
| Brand-colored focus outline | Grey, black, or white only |
| `z-index: 9999` | Fixed scale, or `isolation: isolate` |
| 20px icon button with 20px hit area | Pseudo-element to 44×44 (40×40 desktop) |
| Icon visually off-center in button | −2px padding on icon side; fix SVG for asymmetric icons |
| Custom page scrollbar | Native on the page; customize only small scrollers |
| Blank div for empty list | Headline + explanation + action, sized like the filled state |
| Decorative layer blocking clicks | `pointer-events: none` |
| State flashing on refresh | Persist + set initial state before render |
| 2,000-row table with choppy scroll | Virtualize |
| Videos playing in background tabs and below the fold | Pause them off-screen |

## Checklist

- [ ] `-webkit-font-smoothing: antialiased` on body; fonts subsetted
- [ ] No font-weight changes on hover/selected states
- [ ] Dynamic numbers use `tabular-nums`
- [ ] Skeletons, placeholders, images, and videos have fixed dimensions; async content has skeletons
- [ ] Proper typographic characters; `text-wrap: balance` on headings
- [ ] Focus outlines neutral and intact; `::selection` subtle if styled
- [ ] Z-index from the scale, or stacking contexts via `isolation`
- [ ] Hit areas ≥ 44×44 touch / 40×40 desktop, never overlapping
- [ ] Icons optically centered
- [ ] Page scrollbar untouched; anchors have `scroll-margin-top`; safe areas padded
- [ ] Empty states designed; decorative elements inert; no refresh flash
- [ ] Lists over ~50 items are virtualized
- [ ] Fonts and hero images preloaded; below-fold images lazy
- [ ] Off-screen loops and video paused via IntersectionObserver
- [ ] Static content (blog, docs, changelog) generated at build time
- [ ] Motion items (transitions, press scale, `will-change`, theme switch): run the `ui-motion` review order
