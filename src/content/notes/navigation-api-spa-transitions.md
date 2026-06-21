---
title: The browser already has a router
date: "2024-08-21"
category: Browser APIs
description: "The Navigation API intercepts navigations at the browser level — clicks, form submissions, history.back() — and lets you handle them with custom transitions. No client-side router library required."
tags:
  - javascript
  - browser-api
  - navigation
  - spa
  - view-transitions
draft: false
---

In the early 2010s, the browser offered no interception point for navigation. `hashchange` existed; it was limited. `popstate` caught history events; it didn't catch link clicks. If you wanted smooth transitions, you had to own the navigation entirely. The router library was the only answer.

The Navigation API, shipping in Chrome 102, gives you the interception point the browser was missing. You can intercept any navigation — link clicks, form submissions, `history.back()`, JavaScript-triggered — and decide what to do with it.

```javascript
window.navigation.addEventListener('navigate', (event) => {
  // Ignore external navigations
  if (!event.canIntercept) return;

  // Ignore hash-only navigations
  if (new URL(event.destination.url).pathname === location.pathname) return;

  event.intercept({
    async handler() {
      const response = await fetch(event.destination.url);
      const html = await response.text();
      const parser = new DOMParser();
      const doc = parser.parseFromString(html, 'text/html');

      // Update the page with the new content
      document.title = doc.title;
      document.getElementById('main-content').replaceWith(
        doc.getElementById('main-content')
      );
    }
  });
});
```

Every internal navigation routes through this handler. The browser still manages the URL bar, history stack, scroll restoration and back button. You manage the render.

> **Browser support:** Chrome 102+. Safari: check [MDN](https://developer.mozilla.org/en-US/docs/Web/API/Navigation_API) for current status — not supported as of mid-2024, implementation in progress. For production use with Safari traffic, use the polyfill (see below) or fall back to the traditional `history.pushState` pattern.

## Paired with View Transitions

The combination that changes what "web navigation" can feel like:

```javascript
window.navigation.addEventListener('navigate', (event) => {
  if (!event.canIntercept) return;

  event.intercept({
    async handler() {
      const html = await fetch(event.destination.url).then(r => r.text());
      const doc = new DOMParser().parseFromString(html, 'text/html');
      const newMain = doc.getElementById('main');

      if (!document.startViewTransition) {
        // Fallback: swap without transition
        document.getElementById('main').replaceWith(newMain);
        return;
      }

      await document.startViewTransition(() => {
        document.getElementById('main').replaceWith(newMain);
      });
    }
  });
});
```

```css
::view-transition-old(root) {
  animation: 200ms ease fade-out;
}

::view-transition-new(root) {
  animation: 200ms ease fade-in;
}

@keyframes fade-out { to   { opacity: 0; } }
@keyframes fade-in  { from { opacity: 0; } }
```

Both APIs together: animated transitions between pages, no router library, no framework. The browser handles history. CSS handles animation. JavaScript fetches content.

## What the router library still provides

Being honest about what you're giving up:

- **Nested layouts.** Router libraries preserve nested layout state efficiently. The Navigation API approach fetches full pages. A persistent sidebar with its own state needs manual handling.
- **Client-side state management.** Router libraries integrate with state systems. Navigation API doesn't.
- **Prefetching patterns.** Libraries like TanStack Router have sophisticated data-fetching strategies tied to navigation. You'd build those on top.

## The polyfill situation

The WICG maintains an official Navigation API polyfill. For most use cases, it works adequately. For production use in Safari-heavy traffic, test it.

```html
<script type="module">
  // Pin to a specific version in production — don't use @latest
  import { navigation } from 'https://cdn.jsdelivr.net/npm/@virtualstate/navigation@1.0.0-alpha.211/+esm';
</script>
```

## The thing it changes about how you think

React Router, Vue Router, Next.js, SvelteKit — all built on the premise that the browser's navigation model is inadequate. The Navigation API is the browser acknowledging that this was sometimes right and shipping a native version of the solution.

The router library isn't obsolete. For sites that need full routing sophistication — nested layouts, data prefetching, code splitting — the library remains appropriate.

For sites that want smooth transitions between server-rendered pages without adopting a full framework: the platform now has an answer. Whether you use it depends on how much of the router you actually needed.
