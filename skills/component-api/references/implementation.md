# Component API: Examples & Implementation

Express variant styles in the project's existing styling system (Tailwind, CSS Modules, vanilla-extract, styled-components). Don't impose a new one.

## Composition over configuration

```jsx
// Good - composable
<Card>
  <CardHeader>
    <CardTitle>Title</CardTitle>
    <CardDescription>Description</CardDescription>
  </CardHeader>
  <CardContent>Content here</CardContent>
  <CardFooter>
    <Button>Action</Button>
  </CardFooter>
</Card>

// Bad - configuration object
<Card
  header={{ title: "Title", description: "Description" }}
  content="Content here"
  footer={{ actions: [{ label: "Action", onClick: () => {} }] }}
/>
```

## Compound components

```jsx
// Good - compound components
<Dialog>
  <Dialog.Trigger>Open</Dialog.Trigger>
  <Dialog.Content>
    <Dialog.Title>Are you sure?</Dialog.Title>
    <Dialog.Description>This action cannot be undone.</Dialog.Description>
    <Dialog.Close>Cancel</Dialog.Close>
  </Dialog.Content>
</Dialog>

// Bad - prop drilling everything
<Dialog
  trigger="Open"
  title="Are you sure?"
  description="This action cannot be undone."
  closeText="Cancel"
/>
```

Share implicit state through context: the root provides, the parts consume.

```jsx
const DialogContext = createContext(null);

function Dialog({ children }) {
  const [open, setOpen] = useState(false);
  return (
    <DialogContext.Provider value={{ open, setOpen }}>
      {children}
    </DialogContext.Provider>
  );
}

function DialogTrigger(props) {
  const { setOpen } = useContext(DialogContext);
  return <button onClick={() => setOpen(true)} {...props} />;
}
```

## Customization level

```jsx
// Too rigid - no customization
<Button>Click me</Button>

// Too flexible - overwhelming API
<Button
  backgroundColor="#000"
  hoverBackgroundColor="#333"
  activeBackgroundColor="#111"
  borderRadius={4}
  paddingX={16}
  paddingY={8}
  fontSize={14}
  fontWeight={500}
  // ... 30 more props
>
  Click me
</Button>

// Just right - variants + escape hatch
<Button variant="primary" size="md" className="custom-override">
  Click me
</Button>
```

## Naming

```jsx
// Good - consistent, mirrors HTML
<Input disabled />
<Button disabled />
<Select disabled />

// Bad - inconsistent
<Input disabled />
<Button isDisabled />
<Select readonly />

// Good - positive booleans
<Input disabled />
<Modal open />

// Bad - negatives
<Input notEnabled />
<Modal isNotClosed />

// Good - on* handlers
<Input onChange={} onBlur={} />
<Dialog onOpenChange={} onClose={} />

// Bad
<Input handleChange={} blurHandler={} />
```

## Controlled + uncontrolled in one component

```jsx
function Input({
  value: controlledValue,
  defaultValue,
  onChange,
  ...props
}) {
  const [internalValue, setInternalValue] = useState(defaultValue ?? "");

  const isControlled = controlledValue !== undefined;
  const value = isControlled ? controlledValue : internalValue;

  function handleChange(e) {
    if (!isControlled) {
      setInternalValue(e.target.value);
    }
    onChange?.(e);
  }

  return <input value={value} onChange={handleChange} {...props} />;
}

// Uncontrolled
<Input defaultValue="hello" />

// Controlled
<Input value={value} onChange={(e) => setValue(e.target.value)} />
```

## `asChild` with Radix Slot

```jsx
import { Slot } from "@radix-ui/react-slot";

function Button({ asChild, ...props }) {
  const Comp = asChild ? Slot : "button";
  return <Comp {...props} />;
}

// Render as button (default)
<Button>Click me</Button>

// Render as link
<Button asChild>
  <a href="/page">Click me</a>
</Button>

// Render as Next.js Link
<Button asChild>
  <Link href="/page">Click me</Link>
</Button>
```

## Forwarding refs

```jsx
const Button = forwardRef<HTMLButtonElement, ButtonProps>(
  ({ children, ...props }, ref) => {
    return (
      <button ref={ref} {...props}>
        {children}
      </button>
    );
  }
);
```

## Spreading remaining props

```jsx
function Button({ variant, size, className, ...props }) {
  return (
    <button
      className={cn(buttonVariants({ variant, size }), className)}
      {...props}
    />
  );
}

// Now these work:
<Button data-testid="submit" aria-label="Submit form">
  Submit
</Button>
```

## Sensible defaults

```jsx
function Button({
  variant = "primary",
  size = "md",
  type = "button", // Not "submit" - safer default
  ...props
}) {
  // ...
}
```

## Children vs render props vs slot props

```jsx
// Simple - use children
<Card>
  <CardContent />
</Card>

// Complex with data - render prop
<List
  items={users}
  renderItem={(user) => <UserCard user={user} />}
/>
```

Slot props for optional sections taking arbitrary JSX:

```jsx
function Card({ children, header, footer }) {
  return (
    <div className="card">
      {header && <div className="card-header">{header}</div>}
      <div className="card-content">{children}</div>
      {footer && <div className="card-footer">{footer}</div>}
    </div>
  );
}

<Card
  header={<h2>Title</h2>}
  footer={<Button>Save</Button>}
>
  Main content
</Card>
```

## Error boundaries

Wrap complex components so one broken widget doesn't take down the page:

```jsx
function ComponentWithErrorBoundary({ children }) {
  return (
    <ErrorBoundary fallback={<ErrorFallback />}>
      {children}
    </ErrorBoundary>
  );
}
```

## File structure and exports

```
components/
├── button/
│   ├── button.tsx        # Main component
│   ├── button.test.tsx   # Tests
│   └── index.ts          # Public exports
├── card/
│   ├── card.tsx
│   ├── card-header.tsx
│   ├── card-content.tsx
│   └── index.ts
```

```tsx
// components/card/index.ts
export { Card } from "./card";
export { CardHeader } from "./card-header";
export { CardContent } from "./card-content";

// Or as compound component
export { Card, CardHeader, CardContent } from "./card";
```
