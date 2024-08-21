---
title: The browser already has a router
date: "2024-08-21"
category: Browser APIs
description: The Navigation API intercepts navigations at the browser level — clicks, form submissions, history.back() — and lets you handle them with custom transitions. No client-side router library required.
tags:
  - javascript
  - browser-api
  - navigation
  - spa
  - view-transitions
draft: false
---

We made a mistake in the early 2010s, and we've been maintaining it ever since.

The mistake was deciding that the browser's navigation model was fundamentally inadequate, and that the correct response was to replace it entirely with JavaScript. Full routing control: intercept every link click, push your own history entries, decide what renders and when. The framework model.

The framework model was rational. The browser offered no interception point. `hashchange` existed; it was limited. `popstate` caught history events; it didn't catch link clicks. If you wanted smooth transitions, you had to own the navigation.

The Navigation API, shipping in Chrome 102 and building support through 2023–2024, gives you the interception point. You can intercept navigations — any navigation: link clicks, form submissions, `history.back()`, JavaScript-triggered — and decide what to do with them.

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

Every internal navigation — every link click, every `history.pushState` call, every back/forward button press — routes through this handler. The browser still manages the URL bar, the history stack, scroll restoration, and the back button. You manage the render.

## Paired with View Transitions

The feature combination that changes what "web navigation" can feel like:

```javascript
window.navigation.addEventListener('navigate', (event) => {
  if (!event.canIntercept) return;

  event.intercept({
    async handler() {
      // Fetch the new page
      const html = await fetch(event.destination.url).then(r => r.text());
      const doc = new DOMParser().parseFromString(html, 'text/html');

      // Transition with View Transitions API
      if (!document.startViewTransition) {
        // Fallback: no transition
        document.getElementById('main').replaceWith(doc.getElementById('main'));
        return;
      }

      await document.startViewTransition(() => {
        document.getElementById('main').replaceWith(doc.getElementById('main'));
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

Both APIs together: smooth animated transitions between pages, no router library, no framework. The browser handles history. The CSS handles animation. The JavaScript fetches content.

## What the router library still provides

Being honest about what you're giving up:

- **Nested layouts**: Router libraries handle nested layout preservation efficiently. The Navigation API approach fetches full pages and swaps content. If you have a persistent sidebar with its own state, you need to be clever about what you replace.
- **Client-side state management**: Router libraries integrate with state systems. Navigation API doesn't — you manage that yourself.
- **Prefetching patterns**: Libraries like TanStack Router have sophisticated data-fetching strategies tied to navigation. You'd build that on top of the Navigation API.
- **Safari support**: As of mid-2024, Safari doesn't support the Navigation API. `window.navigation` is undefined. Requires a polyfill or graceful fallback.

## The polyfill situation

The WICG maintains an official Navigation API polyfill. For most use cases, it works adequately. For production use in Safari-heavy traffic, test it.

```html
<script type="module">
  import { navigation } from 'https://cdn.jsdelivr.net/npm/@virtualstate/navigation/+esm';
  // Navigation API available cross-browser
</script>
```

## The thing it changes about how you think

React Router, Vue Router, Next.js, SvelteKit — all of them are built on the premise that the browser's navigation model is a problem to be solved. The Navigation API is the browser acknowledging that the premise was sometimes right, and shipping a native version of the solution.

The router library isn't obsolete. It's just no longer the only way to have navigation control. For sites that need full routing sophistication — nested layouts, data prefetching, code splitting — the library remains appropriate.

For sites that want smooth transitions between server-rendered pages without adopting a full framework: the platform now has an answer. Whether you use it depends on how much of the router you actually needed.
