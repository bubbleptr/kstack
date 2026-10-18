# Marketing pages: implementation and checklist

Examples use Next.js/React; the principles are framework-agnostic. Match the project's framework conventions.

## Intro animation, once per session

```jsx
useEffect(() => {
  const hasSeenIntro = sessionStorage.getItem('hasSeenIntro');
  if (hasSeenIntro) {
    setSkipIntro(true);
  } else {
    sessionStorage.setItem('hasSeenIntro', 'true');
  }
}, []);
```

`sessionStorage`, not `localStorage`: the intro plays again on a genuinely new visit, not on every internal navigation.

## Performance

Preload fonts so text doesn't reflow when the webfont lands:

```jsx
import { preload } from 'react-dom';

// In app initialization
preload('/fonts/inter-var.woff2', { as: 'font', type: 'font/woff2' });
```

Preload above-the-fold images:

```html
<link rel="preload" as="image" href="/hero-image.webp" />
```

Statically generate blog, changelog, docs, and any frequently updated data: build time + revalidation, never request-time fetching.

```jsx
// Next.js example
export async function generateStaticParams() {
  const posts = await getPosts();
  return posts.map((post) => ({ slug: post.slug }));
}

export const revalidate = 3600; // Revalidate every hour
```

## Header navigation

Hover submenus exist in the DOM at all times, visually hidden, not mounted on hover, so the HTML stays real for SEO and accessibility:

```html
<nav>
  <button aria-expanded="false">Products</button>
  <div class="submenu" aria-hidden="true">
    <!-- Full content here, not dynamically loaded -->
  </div>
</nav>
```

## CTAs follow auth state

| State | CTA |
| --- | --- |
| Logged out | "Get Started" or "Sign Up" |
| Logged in | "Go to Dashboard" or "Open App" |

```jsx
<Button href={isLoggedIn ? '/dashboard' : '/signup'}>
  {isLoggedIn ? 'Go to Dashboard' : 'Get Started'}
</Button>
```

## Documentation sites

- Copy button on every code snippet. No exceptions.
- Every page copyable as markdown: a "Copy as Markdown" button plus `.md` URLs (`/docs/getting-started.md` returns raw markdown). Serves humans and the LLMs reading your docs.
- Visual examples everywhere: show what the code produces; let people see and ideally touch the result.

## Blog and changelog

- RSS at predictable paths: `/blog/rss.xml`, `/changelog/rss.xml`.
- Balance headings:

  ```css
  article h1, article h2 {
    text-wrap: balance;
  }
  ```

## Pre-ship checklist

- [ ] No scroll-triggered animations, scroll hijacking, non-1:1 parallax, or auto-advancing carousels
- [ ] Intro animation gated behind `sessionStorage`
- [ ] Fonts preloaded; no layout shift on load
- [ ] Above-the-fold images preloaded
- [ ] Blog/docs/changelog statically generated with revalidation
- [ ] Nav submenu content present in the DOM when closed
- [ ] CTAs switch copy and destination based on auth state
- [ ] Docs snippets have copy buttons; pages export as markdown (`.md` URLs)
- [ ] Docs show visual examples, not just code
- [ ] RSS feeds live for blog and changelog
- [ ] `text-wrap: balance` on article headings
- [ ] Code-built illustrations have `aria-label`, disabled selection, disabled pointer events (see [icons-and-illustrations.md](icons-and-illustrations.md))
