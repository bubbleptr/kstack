# Performance Recipes

Plain CSS/JS for clarity; implement in the project's existing styling system and framework.

## Virtualize long lists

```jsx
import { useVirtualizer } from '@tanstack/react-virtual';

function VirtualList({ items }) {
  const parentRef = useRef(null);

  const virtualizer = useVirtualizer({
    count: items.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 50,
  });

  return (
    <div ref={parentRef} style={{ height: '400px', overflow: 'auto' }}>
      <div style={{ height: virtualizer.getTotalSize(), position: 'relative' }}>
        {virtualizer.getVirtualItems().map((virtualItem) => (
          <div
            key={virtualItem.key}
            style={{
              position: 'absolute',
              top: virtualItem.start,
              left: 0,
              width: '100%',
              height: virtualItem.size,
            }}
          >
            {items[virtualItem.index]}
          </div>
        ))}
      </div>
    </div>
  );
}
```

## Preload fonts

```jsx
import { preload } from 'react-dom';

preload('/fonts/inter-var.woff2', {
  as: 'font',
  type: 'font/woff2',
  crossOrigin: 'anonymous',
});
// Plain HTML equivalent:
// <link rel="preload" href="/fonts/inter.woff2" as="font" type="font/woff2" crossorigin />
```

## Preload hero images, lazy-load the rest

```html
<link rel="preload" as="image" href="/hero.webp" />
```

Below-the-fold images get `loading="lazy"` so they don't compete with critical resources.

## Pause off-screen work

Looping animations, video, and resource-intensive operations:

```js
const observer = new IntersectionObserver((entries) => {
  entries.forEach((entry) => {
    if (entry.isIntersecting) {
      startAnimation();
    } else {
      pauseAnimation();
    }
  });
});

observer.observe(element);
```

## Static generation

```jsx
// Next.js example
export async function getStaticProps() {
  const posts = await fetchPosts();
  return {
    props: { posts },
    revalidate: 3600, // Revalidate hourly
  };
}
```
