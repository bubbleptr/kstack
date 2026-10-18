# Easing and Springs

## Easing curve library

Built-in keywords (`ease-out`, `ease-in-out`) are usually too weak to feel intentional. Use custom curves, sorted weak → strong. Stronger curves feel snappier and more designed; start around cubic/quart and adjust by eye.

```css
/* ease-out: enters, exits, user-initiated UI */
--ease-out-quad:  cubic-bezier(0.25, 0.46, 0.45, 0.94);
--ease-out-cubic: cubic-bezier(0.215, 0.61, 0.355, 1);
--ease-out-quart: cubic-bezier(0.165, 0.84, 0.44, 1);
--ease-out-quint: cubic-bezier(0.23, 1, 0.32, 1);
--ease-out-expo:  cubic-bezier(0.19, 1, 0.22, 1);
--ease-out-circ:  cubic-bezier(0.075, 0.82, 0.165, 1);

/* ease-in-out: elements moving/morphing on screen */
--ease-in-out-quad:  cubic-bezier(0.455, 0.03, 0.515, 0.955);
--ease-in-out-cubic: cubic-bezier(0.645, 0.045, 0.355, 1);
--ease-in-out-quart: cubic-bezier(0.77, 0, 0.175, 1);
--ease-in-out-quint: cubic-bezier(0.86, 0, 0.07, 1);
--ease-in-out-expo:  cubic-bezier(1, 0, 0, 1);
--ease-in-out-circ:  cubic-bezier(0.785, 0.135, 0.15, 0.86);
```

Why ease-out for enters: the fast start delivers immediate feedback (the element jumps toward its destination, then settles). Ease-in back-loads the movement, so the same duration feels slower and sluggish.

`linear` only for constant-rate motion: marquees, tickers, time visualization.

## Springs

Springs have no fixed duration; they simulate physics, feel alive, and preserve velocity when interrupted.

Use a spring instead of a curve for:

- Drag interactions with momentum (sheets, swipe-to-dismiss, sliders)
- Gestures the user can interrupt mid-motion
- Elements meant to feel physical/alive (Dynamic Island-style morphs)
- Playful, organic interfaces

**Prefer the duration + bounce API** (Apple's approach; far easier to reason about than mass/stiffness/damping):

```js
{ type: "spring", duration: 0.5, bounce: 0.2 }
// vs the physics API you rarely need:
{ type: "spring", mass: 1, stiffness: 100, damping: 10 }
```

**Bounce:** default `bounce: 0` in product UI. Add bounce only for playful, physical interactions (drag-to-dismiss release, a toy-like toggle), subtle: 0.1–0.3, never more.

**Interruptibility is the killer feature:** when a gesture reverses mid-flight, a spring carries current velocity into the new animation; a keyframe restarts from zero and looks broken. Gesture-driven alone justifies a spring.

## Transitions vs. keyframes

A transition interpolates from wherever the element currently is toward its newest target; interrupted, it retargets from the live position and takes only as long as the remaining distance needs. A keyframe animation plays a fixed timeline from frame one; interrupted, it snaps back and replays.

```css
/* Interruptible: hitting close mid-open reverses from the current position */
.side-panel { transform: translateX(100%); transition: transform 240ms ease-out; }
.side-panel[data-open] { transform: translateX(0); }

/* Not interruptible: closing mid-open replays or jumps */
.side-panel[data-open] { animation: panel-in 240ms ease-out forwards; }
```

Rule: anything the user can toggle gets a transition. Keyframes only for sequences that always run to completion.
