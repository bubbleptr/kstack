# Ship-check lookup tables

Load during a pre-launch sweep: the greps in Phase 2, the severity examples in Phase 3, the findings-table shape and device/human examples in Phase 5, and the Never Ship table before writing the verdict.

## Greps that pay off across lanes

```
transition: all          transition-all           ease-in
scale(0)                 width:|height:           will-change
filter: blur(            prefers-reduced-motion   (hover: hover)
aria-label               outline: none            outline-none
onClick                  role="button"            <label
autoFocus                placeholder=             disabled=
100vh                    h-screen                 safe-area-inset
user-scalable            maximum-scale            theme-color
font-size: 1[0-5]px      text-sm                  z-index: 9
<img                     <video                   preload
localStorage             useEffect                dark:
console.log              debugger                 TODO|FIXME|XXX
lorem                    example.com              localhost
metadata                 og:image                 force-dynamic
```

`grep` wrappers often skip gitignored and binary-looking files. For an exhaustive sweep use `command grep` or `rg --no-ignore` scoped to `src`.

## Severity examples

The lane files carry a default severity per check. The severity tree in SKILL.md overrides the default when context demands: a missing `alt` on the only product image is a blocker, a `100vh` on a desktop-only tool is nothing.

| Severity | Examples |
| --- | --- |
| **BLOCKER** | Focus escapes a modal; icon buttons with no name; `outline: none` sitewide; inputs zoom iOS; layout shifts on load; `user-scalable=no`; submit fires twice; blank empty state on the first screen a new user sees; error state that renders nothing; lorem ipsum or fabricated testimonials on a public page; Lapse or a Leva panel mounted in production; `console.log` of user data; animating layout properties on a frequent interaction; no reduced-motion on large movement; dark mode with unreadable text; no `<title>` |
| **SHOULD FIX** | Sticky hover on touch; tap highlight flash; `100vh` app shell; missing safe areas; no `:active` feedback; fonts not preloaded; long list not virtualized; `transition: all`; placeholder as the only label; spinner where a skeleton belongs; theme flash on refresh; `z-index: 9999`; `ease-in` on an entrance; missing OG image; content routes fetched per request |
| **POLISH** | Tabular numbers on counters; `text-wrap: balance` on headings; `theme-color` per scheme; `enterkeyhint`; `scroll-margin-top` under a sticky header; `…` instead of `...`; RSS feed; intro animation gated per session |

## Findings table (Part 2)

One markdown table per lane that has findings, in lane order A → I. One row per finding, sorted by severity. Never a "Before:/After:" list. The **Status** column says whether the fix was applied. A lane with no findings gets one line: "B. Performance: clean."

| Severity | Before | After | Why | Status |
| --- | --- | --- | --- | --- |
| BLOCKER | `<button><CloseIcon /></button>` at `Dialog.tsx:41` | `<button aria-label="Close dialog">` | Screen readers announce "button" with no name | Fixed |
| BLOCKER | `<Lapse />` mounted unconditionally at `layout.tsx:18` | Mounted only when `process.env.NODE_ENV === "development"` | The animation inspector ships to every visitor | Fixed |
| BLOCKER | `projects.length === 0` renders `null` at `Projects.tsx:33` | — | Every new account opens to a blank page | Open (needs copy) |
| SHOULD FIX | `height: 100vh` on `.app` at `layout.css:8` | `height: 100dvh` | `100vh` overflows behind the mobile URL bar | Fixed |
| SHOULD FIX | `transition: all 300ms` at `Button.tsx:20` | `transition: background-color 150ms, transform 150ms var(--ease-out)` | `all` animates unintended properties off the GPU | Fixed |
| POLISH | `.counter` at `Stats.tsx:14` has no `tabular-nums` | `font-variant-numeric: tabular-nums` | Changing digits shift the layout | Fixed |

Cite `file:line` for every row. When one root cause produces many instances (forty ungated `:hover` rules), report it as one row with the count and fix it at the root.

## Device and human lists (Part 3)

- **Needs a device**: what you could not verify from code. Be specific about *what to look for*: "open the drawer on an iPhone and confirm the bottom padding clears the home indicator", "tab through the settings page with VoiceOver on and confirm the sidebar is read before the content", "scroll the dashboard on a 2-year-old Android and watch the sidebar open for dropped frames", "sign up with a fresh account and screenshot every screen before adding data", "toggle dark mode on the pricing page and read every line of text".
- **Needs a human**: fixes you did not apply because they change a decision, need a value you'd have to invent, or are bigger than a ship check. Each with the one-line why, the exact change you'd make if told to, and the owning skill when there is one.

## Never ship

Cross-lane self-check before you write the verdict. Any of these open is a **Not yet**.

| Never | Instead |
| --- | --- |
| Icon button with no accessible name | `aria-label` that names the action ("Search", not "icon") |
| `outline: none` with no visible replacement | `:focus-visible` outline with `outline-offset: 2px`, in grey, black, or white |
| `<div onClick>` doing a button's job | `<button type="button">` |
| Modal that lets focus escape or doesn't return it | A dialog primitive: trap focus, restore to the trigger on close |
| Closed drawer or panel still in the tab order | `inert` or `visibility: hidden` |
| `<label>` not associated with its input, or a placeholder as the label | `for`/`id`, or wrap the input |
| Error shown by color alone | Color + icon + colocated message, `aria-invalid` |
| Text under 4.5:1 in either theme | Move the foreground's lightness away from the background's |
| Movement with no `prefers-reduced-motion` handling | Swap movement for opacity under `reduce` |
| `user-scalable=no` or `maximum-scale=1` | 16px inputs |
| Input font size under 16px on touch | `font-size: 16px` (`text-base pointer-fine:text-sm`) |
| Submit button that stays enabled while submitting | `disabled={isSubmitting}` and a label that says what's happening |
| Inputs outside a `<form>` | `<form onSubmit>` so Enter submits |
| Destructive action with no confirmation | A confirmation dialog, not `window.confirm()` |
| Ungated `:hover`, or an action reachable only on hover | `@media (hover: hover) and (pointer: fine)`; render the control always |
| `100vh` for an app shell or bottom-pinned UI | `100dvh` |
| `env(safe-area-inset-*)` without `viewport-fit=cover` | Add the meta tag or the value is `0` |
| Image or video with no reserved dimensions | `width`/`height` attributes or `aspect-ratio` |
| Fonts loaded without `preload`, or with unused weights | Preload the used `woff2` files, only the used weights |
| Animating `width`/`height`/`margin`/`padding`/`top`/`left` | `transform` / `opacity` |
| `transition: all`, or Tailwind's bare `transition` | Named properties |
| `setState` per frame in a scroll, drag, or rAF handler | A ref write or a motion value |
| 1,000+ row list rendered directly | Virtualize |
| A list that renders `null` when empty | Headline + one line + the action that creates the first item |
| A failed request that renders nothing, or "Something went wrong" | What happened + a specific recovery action |
| A wait over 400ms with no indicator | A skeleton that holds the loaded shape |
| Theme or sidebar state that flashes on refresh | Read the stored value before first paint |
| A dark mode nobody opened | Open every route in it, or remove the toggle |
| `ease-in` on an entrance, or an entrance from `scale(0)` | `ease-out` from `scale(0.95)` + opacity |
| Animation on a keyboard shortcut or a 100+/day action | No animation |
| Lapse, Leva, devtools, or a prototype picker mounted in production | Gate the mount on `NODE_ENV === "development"` |
| `/proto`, `/lab`, or `/test` routes publicly reachable | Gate them out of the production build |
| `console.log`, `debugger`, lorem ipsum, fabricated stats or testimonials | Delete; real content or none |
| A page with no `<title>`, description, or OG image | Per-route metadata |
| Blog, docs, or changelog fetched at request time | Static generation with revalidation |
| Declaring it ready from device emulation | Real hardware, and say what to check |
