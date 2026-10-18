# Forms & Inputs: Code

Plain CSS/JSX for clarity. Implement in the project's existing styling system; the values matter, the syntax doesn't.

## Labels

Clicking the label must focus the input:

```html
<label for="email">Email</label>
<input id="email" type="email" />

<!-- Or wrap the input -->
<label>
  Email
  <input type="email" />
</label>
```

## Input types and keyboard attributes

```html
<input type="email" />    <input type="password" />
<input type="tel" />      <input type="url" />
<input type="number" />   <input type="search" />
```

Right mobile keyboard, browser validation, and autofill for free.

| Field | Attributes |
| --- | --- |
| Codes | `inputmode="numeric"` |
| Amounts | `inputmode="decimal"` |
| Email / phone | `type="email"` / `type="tel"` |
| Usernames, codes | `autocapitalize="none"` `autocorrect="off"` |
| Verification inputs | `autocomplete="one-time-code"` (iOS offers the SMS code) |
| Return key label | `enterkeyhint="send"` / `"search"` / `"done"` |

## 16px inputs (stops iOS zoom)

```css
input, textarea, select {
  font-size: 16px; /* the minimum; 1rem at the default root size */
}

/* If the design wants smaller input text on desktop, scale up only on touch */
@media (pointer: coarse) {
  input, textarea, select { font-size: 16px; }
}
```

Tailwind: `text-base pointer-fine:text-sm` (v4.1+; otherwise a `@media (pointer: fine)` rule). Never a width breakpoint like `md:`, which shrinks inputs on an iPad. `text-sm` alone on an input is the bug.

## Focus

```css
input:focus-visible { outline: 2px solid var(--focus-color); outline-offset: 2px; }
```

## Spellcheck, autocomplete, password managers

Off for usernames, search, codes, slugs. On for email, name, address, payment.

```html
<input type="text" spellcheck="false" autocomplete="off" />
<input data-lpignore="true" data-1p-ignore />
```

## Decorations overlay the input

```css
.input-wrapper {
  position: relative;
}

.input-icon {
  position: absolute;
  left: 12px;
  top: 50%;
  transform: translateY(-50%);
  pointer-events: none;
}

.input-field { padding-left: 40px; }
```

Clickable decorations (a clear button) refocus the input:

```jsx
<button
  className="input-icon-button"
  onClick={() => inputRef.current?.focus()}
>
  <SearchIcon />
</button>
```

## Autofocus: modal on desktop only

```jsx
const isCoarse =
  typeof window !== "undefined" && window.matchMedia("(pointer: coarse)").matches;

<input autoFocus={!isCoarse} />
```

## Enter submits; Cmd/Ctrl+Enter submits textareas

```jsx
<form onSubmit={handleSubmit}>
  <input type="text" />
  <button type="submit">Submit</button>
</form>
```

```jsx
function handleKeyDown(e) {
  if ((e.metaKey || e.ctrlKey) && e.key === 'Enter') {
    handleSubmit();
  }
}
```

## Buttons

```html
<!-- Good -->
<button onClick={handleClick}>Click me</button>
<!-- Bad -->
<div onClick={handleClick}>Click me</div>
<span onClick={handleClick}>Click me</span>
```

Disable while submitting and say what's happening:

```jsx
const [isSubmitting, setIsSubmitting] = useState(false);

<button
  disabled={isSubmitting}
  onClick={async () => {
    setIsSubmitting(true);
    try {
      await submitForm();
    } finally {
      // A failed request must not leave the button permanently disabled.
      setIsSubmitting(false);
    }
  }}
>
  {isSubmitting ? 'Submitting...' : 'Submit'}
</button>
```

Shortcut surfaced as a tooltip (show `Cmd` or `Ctrl` per OS, see `a11y-patterns.md`):

```jsx
<Tooltip content="Save (Cmd+S)">
  <button onClick={save}>Save</button>
</Tooltip>
```

## Validation: reward early, punish late

Validate on blur or submit. After a field has shown an error, validate on change so it clears the moment it's fixed. Error next to its field:

```jsx
<div className="field">
  <input type="email" aria-invalid={!!error} />
  {error && <span className="error">{error}</span>}
</div>
```

## Checkbox rows: no dead zones

```css
.checkbox-wrapper {
  display: flex;
  align-items: center;
  gap: 8px;
  cursor: pointer;
}

/* Make the entire row clickable, not just the checkbox */
.checkbox-wrapper label { cursor: pointer; flex: 1; }
```

Or one label wrapping everything:

```html
<label class="checkbox-row">
  <input type="checkbox" />
  <span>Remember me</span>
</label>
```

## Destructive confirmation

```jsx
function handleDelete() {
  if (confirm('Are you sure you want to delete this?')) deleteItem();
}
```

`confirm()` is the floor; real products use a proper confirmation modal.

## Fixed bottom CTA over the last field

```css
.form-scroll {
  padding-bottom: calc(var(--cta-height) + env(safe-area-inset-bottom, 0px));
}
```

## Prefill

Prefill from the logged-in user's data. A link to a form prefills from its context: a "Change username" link lands on a form prefilled with "I'd like to change my username to:".

## Checklist

- [ ] Every input has a label that focuses it on click
- [ ] Correct `type` on every input
- [ ] `font-size: 16px` minimum on inputs, textareas, selects
- [ ] Visible `:focus-visible` state on all controls
- [ ] `spellcheck="false"` / `autocomplete="off"` where autofill is noise
- [ ] Decorations absolutely positioned over the input; clicks refocus it
- [ ] Autofocus in modals, suppressed on coarse pointers
- [ ] Inputs wrapped in `<form>`; Enter submits; Cmd/Ctrl+Enter submits textareas
- [ ] Real `<button>` elements; disabled + loading text during submission
- [ ] `:active { transform: scale(0.97) }` press feedback
- [ ] Errors validate on blur/submit and render next to their field
- [ ] Checkbox rows fully clickable
- [ ] Destructive actions confirmed
- [ ] Forms prefilled from user data and link context
