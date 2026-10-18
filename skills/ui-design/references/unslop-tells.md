# Unslop: the tells, in sweep order

Each choice is defensible alone; together they are a watermark. A model reproduces the statistically most common pattern for every decision on the page at once, and fifty median choices compound into a recognizable design, not a median one.

## Subtract first

The additive bias shows up as extra everything: a caption under a heading that restates the heading, an icon in front of every label, a divider between sections that spacing already separates, a badge on the card, a second CTA, a helper line under the input, a border on a surface that also has a shadow. Every element competes and none of it was chosen.

Walk every element (icon, line, badge, caption, container, word) and ask **what breaks if this is gone?** If nothing, delete it. "It adds visual interest" and "it fills the space" are the reasons the model put it there, not reasons to keep it. Run the walk again: deletions expose each other, because an element that survived round one was often only balancing something you just removed. Deletion is free, can't introduce new tells, and every survivor gets hierarchy for nothing.

## 1. Color (loudest tell, goes first)

- **Indigo is not the brand color.** `bg-indigo-500`, violet, the blue-purple family is the single strongest tell, absorbed from years of Tailwind examples. Purple is not banned; *unchosen* purple is. Derive the palette from the product's domain, brand, or content (build it in OKLCH, see [color-oklch.md](color-oklch.md)). A finance tool, a farming app, and an AI startup should not share a hue by accident.
- **One gradient per screen, chosen, or none.** Purple-to-cyan wash, gradient hero background, gradient border, gradient text all on one page is decoration standing in for a decision. Casual gradients also band (ease them, see [surfaces.md](surfaces.md)). At most one, with a job, built from the chosen palette.
- **Dark-and-neon is a costume.** Dark-by-default with glowing borders and neon accents is the model's idea of "premium tech". Dark mode is a real feature with real rules (shadows stop working, surfaces need their own logic). Choose the default theme from where and when the product is used.

## 2. Typography

- **No gradient text.** `bg-clip-text text-transparent bg-gradient-to-r` on the headline costs contrast and legibility. Headlines get a solid color with real contrast.
- **The default stack is a shrug.** Inter at `font-black tracking-tight`, centered, is what a model writes when nobody chose anything. Inter is fine as a decision, paired and sized deliberately. Make one distinctive typographic choice per project; it does more to de-slop a page than any other single change.

## 3. Layout

- **The hero every model builds:** pill badge ("✨ Announcing v2.0"), centered headline, one-line subhead, primary button plus "Learn more", gradient orb behind it. Break the template where the content justifies it: lead with the actual product (real screenshot, live demo), left-align if the copy runs long, cut the badge unless there is genuinely an announcement.
- **Three cards, an icon, a heading, two lines.** The tell isn't the grid; it's that the layout was chosen before the content. One feature that matters gets full width with a real screenshot; six small ones get a compact list; unequal features get unequal space. Symmetry the content didn't ask for is the giveaway.
- **Bento is the new three-card grid.** If the cells were sized before anyone knew what goes in them, it's decoration.

## 4. Decoration

- **Glassmorphism needs glass.** `backdrop-blur` with translucent fill makes sense over something meaningful (photo, video, layered content), not a frosted card over a flat dark background.
- **Nothing floats behind the hero.** Blurred gradient orbs and animated blobs fill space the design didn't know what to do with. Delete them. If the background feels empty afterward, the foreground content is too weak; fix that.
- **One depth cue per surface.** Border, shadow, ring, *and* gradient on one card is hedging. Pick shadows or borders per [surfaces.md](surfaces.md) and apply consistently. `rounded-2xl` is not the answer to every radius: radii come from a scale, nested radii are derived.

## 5. Iconography

- **Emoji are not icons.** 🚀⚡🔒✨ as feature markers is the fastest way to look generated. One real icon set, one stroke width, or none; a features list without icons is more honest than one with rocket ships.
- **Sparkles don't mean AI.** If AI-powered matters to the user, the copy says what it does; if not, the ✨ badge is noise.

## 6. Copy on the screen

- **Generic headlines.** "Build faster. Ship smarter." The test: could a competitor paste it unchanged? Then it's slop. Run interface copy through `unslop`.
- **Fabricated proof.** "Trusted by 10,000+ developers" on an unlaunched product, stock-avatar testimonial grids, logo walls of companies that never said yes: social proof generated as a *layout* filled with invented content. A lie in the interface. Real numbers, real quotes with real names, or cut the section. An empty spot is a stronger signal than fake proof.

## 7. Motion

`fade-in-up` on scroll for every block, staggered children everywhere, `hover:scale-105` on cards, pulsing blobs: motion by default instead of by decision. Scroll-triggered entrances on static marketing content are almost always noise, and a card that isn't clickable has no business reacting to hover. Motion decisions in depth: `ui-motion`.

## Looking designed by someone

- **Derive the palette from something real**: the domain, an existing brand artifact, the content's own colors. A palette with a reason survives the squint test; a tasteful-but-arbitrary one doesn't.
- **One distinctive type decision**: a display face with a point of view, an unusual size contrast, a deliberate pairing. One is enough; three is a costume party.
- **Show the actual product**: real screenshot, real data, a working embed. Abstract illustration is what models reach for because they've never seen your product.
- **Let content break symmetry**: an odd number of features, one oversized item, a section that's just a sentence.
- **Repeat a signature**: one distinctive element (corner treatment, rule weight, motif) used consistently beats ten borrowed ones used once.
- **Restraint is a feature**: the confident version has fewer effects than the generated one. Adding "personality effects" on top is re-slopping with extra steps.
