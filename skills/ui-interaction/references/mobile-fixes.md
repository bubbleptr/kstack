# Mobile Native Feel: Symptom Catalog & Fixes

Plain CSS for clarity. Write it in the project's existing styling system; the values matter, the syntax doesn't. Apply each fix where its why applies, not globally out of habit.

## Symptom table

Match what the user is seeing, then read the section.

| Problem | Solution | § |
| --- | --- | --- |
| Hover state stuck after tap | Wrap in `@media (hover: hover) and (pointer: fine)` | 1 |
| Gray/blue flash on tap | Kill `-webkit-tap-highlight-color` | 2 |
| Layout has the wrong height | `100dvh` (app) or `100svh` (hero) | 3 |
| Page zooms into input | Input font size 16px at the minimum | 4 |
| Tap feels laggy | Feedback on pointer-down + `touch-action: manipulation` | 5 |
| Taps miss small buttons | 44px hit area through a pseudo-element | 6 |
| Pull-to-refresh hijacks scroll | `overscroll-behavior: none` on `html, body` | 7 |
| Content stops at the notch | `viewport-fit=cover` + `env(safe-area-inset-*)` | 8 |
| Keyboard covers the input or the CTA | `interactive-widget=resizes-content` + `dvh` | 9 |
| Keyboard opens by itself | No `autoFocus` on coarse pointers | 9 |
| Long-press selects button text | `user-select: none` on controls only | 10 |
| Carousel scrolls vertically | `touch-action: pan-y` on the gesture surface | 11 |
| Video opens fullscreen or won't autoplay | `muted` + `playsinline` | 12 |
| Status bar color doesn't match | `theme-color` per color scheme | 13 |
| Anchor links land under the sticky header | `scroll-margin-top` | 14 |
| Right in Chrome, wrong on phone | Test on real hardware | 15 |

## 1. Hover state stuck after tap

Touch has no hover; the first tap applies `:hover` and leaves it until the user taps elsewhere.

```css
@media (hover: hover) and (pointer: fine) {
  .button:hover {
    background: var(--gray-3);
    transform: scale(1.02);
  }
}
```

Both conditions matter: `(hover: hover)` = primary input can hover; `(pointer: fine)` = precise, ruling out styluses and Android devices that claim hover. Tailwind v4 `hover:` already compiles to `@media (hover: hover)`; in v3 set `future.hoverOnlyWhenSupported`.

Touch users get press feedback through `:active` (§5). Nothing may be reachable through hover alone: render row actions always at lower emphasis, or reveal them on `:focus-within` too.

## 2. Gray/blue flash on tap

The loudest "this is a website" signal; it fights your press feedback.

```css
html {
  -webkit-tap-highlight-color: transparent;
}
```

Set once, globally. Then every tappable element needs its own `:active` state.

## 3. Wrong height

`100vh` on mobile is the largest viewport (chrome collapsed), so it overflows by the URL bar on load and bottom-pinned buttons sit under it.

```css
/* App shell, drawers, anything that tracks the visible area as chrome shows/hides */
.app { height: 100dvh; }

/* Heroes and first screens: smallest the viewport gets, never cut off */
.hero { min-height: 100svh; }
```

`dvh` resizes as the URL bar collapses: right for an app shell, layout shift on marketing content. `svh` is stable: right for a hero. `lvh` is the old `vh`; almost never wanted. Tailwind: `h-dvh`, `min-h-svh`; `h-screen`/`min-h-screen` are `100vh` and carry the bug. Keep a `100vh` fallback line above only if the support matrix demands it.

## 4. Page zooms into the input

iOS Safari zooms on focus when input font is under 16px and does not zoom back out on blur. `maximum-scale=1` is the wrong fix.

```css
input, textarea, select {
  font-size: 16px; /* the minimum; 1rem at the default root size */
}

@media (pointer: coarse) {
  input, textarea, select { font-size: 16px; }
}
```

Tailwind: `text-base md:text-sm`. `text-sm` alone on an input is the bug. Keyboard attributes (`inputmode`, `enterkeyhint`, `one-time-code`) are in `forms.md`.

## 5. Tap feels laggy

**300ms click delay.** Browsers wait for a possible double-tap. `width=device-width` skips it in most but not all cases (iOS Safari still delays on some elements):

```css
button, a, input, [role="button"], .tappable {
  touch-action: manipulation;
}
```

**Feedback on release instead of press.** A button that changes only on `click` responds when the finger leaves, which reads as lag even at 0ms. Style `:active`; if JS drives the visual, listen to `pointerdown`, not `click`:

```css
.button {
  transition: transform 100ms var(--ease-out), background 100ms;
}
.button:active {
  transform: scale(0.97);
  background: var(--gray-4);
}
```

Feedback fires on press; the action still fires on `click`, so the user can cancel by sliding off. Press feedback 100–160ms, `ease-out`, never below `scale(0.95)`. Use the codebase's easing tokens if they exist (see ui-motion); don't fork a new curve.

## 6. Taps miss small buttons

A finger pad covers about 44px. The visual can stay small; the hit area can't.

```css
.icon-button {
  position: relative;
  width: 24px;
  height: 24px;
}
.icon-button::before {
  content: "";
  position: absolute;
  inset: -10px; /* 24 + 10 + 10 = 44 */
}
```

Or size directly:

```css
.small-button {
  min-width: 44px;
  min-height: 44px;
  display: flex;
  align-items: center;
  justify-content: center;
}
```

Tailwind: `relative before:absolute before:-inset-2.5`. Two hit areas must never overlap; if they collide, shrink both to the largest non-colliding size, or add spacing. Checkbox/radio rows: one `<label>` wrapping control and text.

## 7. Pull-to-refresh hijacks scroll

Wrong in an app with its own scroll containers, a draggable drawer, or a canvas.

```css
html, body {
  overscroll-behavior: none;
}

.sheet-content {
  overflow-y: auto;
  overscroll-behavior: contain;
}
```

`none` on the root, `contain` on inner scrollables (keeps their own bounce, stops the page behind). Never `touchmove` + `preventDefault()`: it blocks scrolling and makes the listener non-passive, costing frames.

Open modal/sheet: lock background scroll with a dialog primitive (Base UI, Radix, vaul), not `overflow: hidden` on `body`, which iOS Safari has a long history of ignoring for touch.

## 8. Content stops at the notch

Paint edge to edge, pad content away from the notch, Dynamic Island, and home indicator.

```html
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover" />
```

```css
.app-header {
  padding-top: env(safe-area-inset-top);
}
.bottom-bar {
  padding-bottom: env(safe-area-inset-bottom);
}
.sheet {
  padding-bottom: calc(1rem + env(safe-area-inset-bottom, 0px));
}
.sidebar {
  padding-left: env(safe-area-inset-left); /* landscape puts the notch on the side */
}
```

Without the meta tag every `env()` is `0px`. Fixed headers, bottom tab bars, toasts, and sheets need this; normal content gets it through the header's padding. Give `env()` a fallback inside `calc`. Next.js: `viewport` export `viewportFit: "cover"`.

## 9. Keyboard covers the input, or opens by itself

iOS shrinks the visual viewport and scrolls the input into view. Android Chrome overlays the keyboard by default, hiding bottom-pinned composers and CTAs.

```html
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover, interactive-widget=resizes-content" />
```

With it, a chat composer at the bottom of a `100dvh` flex column stays above the keyboard with no JavaScript.

A fixed bottom CTA covers the last field: pad the scroll container.

```css
.form-scroll {
  padding-bottom: calc(var(--cta-height) + env(safe-area-inset-bottom, 0px));
}
```

Never open the keyboard for the user; autofocus on desktop only:

```jsx
const isCoarse =
  typeof window !== "undefined" && window.matchMedia("(pointer: coarse)").matches;

<input autoFocus={!isCoarse} />
```

## 10. Long-press selects button text

Control text isn't selectable; content text must stay selectable.

```css
button, [role="button"], .tab, .chip, .drag-handle {
  user-select: none;
  -webkit-user-select: none;   /* Safari still needs the prefix */
  -webkit-touch-callout: none; /* no long-press callout on links/images used as controls */
}
```

Never on `body`: users copy addresses, error messages, order numbers.

## 11. Carousel scrolls vertically

Tell the browser which axes the element owns. Values name what the *browser* may still do.

```css
.carousel {
  touch-action: pan-y; /* carousel handles horizontal; browser keeps vertical */
}
.drag-surface {
  touch-action: none;  /* custom gesture (drag-to-dismiss sheet, slider, canvas) owns every axis */
}
.vertical-sheet-handle {
  touch-action: pan-x; /* sheet handles vertical; horizontal stays with the browser */
}
```

`none` only on elements that really handle everything, or the user can't scroll past them. Native-scroll carousel: prefer `scroll-snap-type: x mandatory` on the track and `scroll-snap-align: start` on slides; browser physics beat a hand-rolled spring and `touch-action` becomes unnecessary. Hand-rolled gesture motion (drag-to-dismiss, sheets) uses springs to keep the finger's velocity; see ui-motion.

## 12. Video opens fullscreen or won't autoplay

```html
<video autoplay muted playsinline loop>
  <source src="video.mp4" type="video/mp4" />
</video>
```

Without `muted` iOS refuses to autoplay; without `playsinline` it opens the fullscreen player. Under `prefers-reduced-motion: reduce`, don't autoplay; show controls (`autoPlay={!prefersReducedMotion} controls={prefersReducedMotion}`). Pause when scrolled out of view (IntersectionObserver pattern: ui-engineering).

## 13. Status bar color doesn't match

```html
<meta name="theme-color" media="(prefers-color-scheme: light)" content="#ffffff" />
<meta name="theme-color" media="(prefers-color-scheme: dark)" content="#0a0a0a" />
<meta name="color-scheme" content="light dark" />
```

Match the color at the very top of the page (header background, not brand color). Next.js: `viewport` export `themeColor: [{ media, color }]`. Class-based theme toggle: update the tag from JS on toggle. Installed PWA: `apple-mobile-web-app-status-bar-style` and manifest `theme_color`/`background_color` are the same decision. Read colors from project tokens; never invent them.

## 14. Anchor links land under the sticky header

```css
[id] {
  scroll-margin-top: 64px; /* the sticky header's height */
}
```

Cap the sticky header itself: an 80px header on a 375×667 phone in landscape leaves almost no content.

## 15. Right in Chrome, wrong on phone

Sticky hover, tap highlight, URL bar vs `vh`, input zoom, click delay, overscroll, safe areas, the software keyboard: none reproduce in device emulation.

- Phone over USB, dev server on `0.0.0.0`, open by the machine's LAN IP.
- iOS: Safari → Develop → the device. Android: `chrome://inspect`.
- Test on a phone a few years old, not the newest. Test with the keyboard open. Test in landscape once.
- Test as an installed PWA if that's a target; standalone changes viewport, safe areas, status bar.

The Xcode Simulator beats emulation but still misses touch feel. Real hardware is the bar.

## Baseline (ship before the first component)

```html
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover, interactive-widget=resizes-content" />
<meta name="theme-color" media="(prefers-color-scheme: light)" content="#ffffff" />
<meta name="theme-color" media="(prefers-color-scheme: dark)" content="#0a0a0a" />
<meta name="color-scheme" content="light dark" />
```

```css
html {
  -webkit-tap-highlight-color: transparent;
  -webkit-text-size-adjust: 100%; /* no font inflation in landscape */
  overscroll-behavior: none;      /* drop on a scrolling document where pull-to-refresh is welcome */
}

input, textarea, select {
  font-size: 16px;
}

button, a, input, [role="button"] {
  touch-action: manipulation;
}

button, [role="button"] {
  user-select: none;
  -webkit-user-select: none;
}

@media (hover: hover) and (pointer: fine) {
  /* all :hover rules live here */
}
```

Next.js root layout:

```ts
export const viewport: Viewport = {
  width: "device-width",
  initialScale: 1,
  viewportFit: "cover",
  interactiveWidget: "resizes-content",
  themeColor: [
    { media: "(prefers-color-scheme: light)", color: "#ffffff" },
    { media: "(prefers-color-scheme: dark)", color: "#0a0a0a" },
  ],
};
```

## Never ship

| Never | Instead |
| --- | --- |
| `user-scalable=no` or `maximum-scale=1` | 16px inputs, which fix the cause |
| Ungated `:hover` | `@media (hover: hover) and (pointer: fine)` |
| An action reachable only through hover | Always rendered, or revealed on `:focus-within` too |
| `100vh` / `h-screen` for an app shell or bottom-pinned UI | `100dvh` |
| `100dvh` on a marketing hero | `100svh` (no layout shift on scroll) |
| Press feedback on `click` only | `:active` / `pointerdown` |
| A 24px hit area on a 24px icon | Pseudo-element out to 44px, never overlapping a neighbor |
| `touchmove` + `preventDefault()` to stop overscroll | `overscroll-behavior` |
| `overflow: hidden` on `body` as the scroll lock | A dialog primitive that locks scroll |
| `user-select: none` on `body` | Only on controls |
| `touch-action: none` on something the user needs to scroll past | `pan-x` / `pan-y` |
| `env(safe-area-inset-*)` without `viewport-fit=cover` | Add the meta tag or the value is `0` |
| `autoFocus` with no pointer gate | Desktop only, through `(pointer: coarse)` |
| Autoplay video without `muted playsinline` | Both attributes, and no autoplay under reduced motion |
| One `theme-color` for both schemes | One per `prefers-color-scheme` |
| User-agent sniffing to detect touch | `(hover)` / `(pointer)` media queries |
| Declaring it fixed from device emulation | Real hardware |
