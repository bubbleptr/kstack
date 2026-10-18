# Contrast

## First, find the real pair

A contrast number means something only for a specific pairing: the text/icon/control color versus the surface it actually paints over, usually the nearest ancestor with an opaque background. Checking against the wrong surface produces confidently wrong conclusions.

## The one-channel repair

Perceived contrast tracks the L gap; chroma and hue contribute almost nothing. The repair is always the same: push the foreground's L away from the background's L, leaving C and H alone so the color keeps its identity.

```css
/* fails: lavender text nearly as light as its lavender card */
.card       { background: oklch(0.93 0.03 300); }
.card-label { color: oklch(0.72 0.14 300); }

/* passes: only L moved */
.card-label { color: oklch(0.42 0.14 300); }
```

Raising C to "make it pop" is the classic dead end: vividness is not contrast.

## APCA (prefer for design decisions)

APCA models perceived lightness difference, the same foundation as oklch; WCAG 2's ratio misjudges many pairs in both directions. It reports Lc, signed (dark-on-light negative, light-on-dark positive); judge the magnitude.

- Body-size text: |Lc| ≥ 60, 75+ comfortable
- Large or heavy text: |Lc| ≥ 45, 60+ preferred
- Non-text essentials (borders, icons, focus rings): |Lc| ≥ 30

Floors, not goals.

## WCAG 2 (when a contract, audit, or policy names it)

- Body text: 4.5:1 AA, 7:1 AAA
- Large text (about 24px and up, or bold from about 18.5px): 3:1 AA, 4.5:1 AAA
- UI components and meaningful graphics: 3:1

The formula is blunt: it flags some readable pairs and passes some murky ones. For compliance work satisfy both systems; the union is rarely much stricter than APCA alone.

The specs disagree: a 3:1 ratio fails WCAG AA for body text but can pass APCA for large text, and APCA is more nuanced for large text and UI components. Neither spec is complete by itself; judge in context.

## Fast triage

- Surface L above 0.6 → light, pair with dark content; at or below 0.6 → pair with light content.
- Near-white surface (L ≥ 0.85): foreground L ≤ 0.45.
- Near-black surface (L ≤ 0.25): foreground L ≥ 0.75.

These get within range in one guess; they don't replace running APCA or WCAG before shipping. Mid-tone backgrounds (L 0.3–0.6, where no foreground has much room) and borderline cases need real numbers. Check both themes.
