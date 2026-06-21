---
title: "content-visibility: the rendering budget you didn't know you had"
date: "2024-07-03"
category: Performance
description: "content-visibility: auto tells the browser to skip rendering offscreen content until it's needed. On long pages, this changes the numbers significantly. Here's what actually breaks when you use it."
tags:
  - performance
  - css
  - rendering
  - paint
draft: false
---

Apply `content-visibility: auto` to a list of 500 items and the initial paint benchmark looks like someone cleared the debt. That's a real number. The caveat is that the browser has opinions about what it skips and those opinions conflict with JavaScript that assumes all DOM elements have real dimensions.

`content-visibility: auto` tells the browser: if this element is far outside the viewport, skip layout, paint and compositing for it. Keep it in the DOM. Keep it accessible. Keep it in the find-in-page index. Just don't pay the rendering cost until it might actually be visible.

```css
.article-section {
  content-visibility: auto;
  contain-intrinsic-size: auto 300px;
}
```

The `contain-intrinsic-size` is load-bearing. Without it, the browser renders every skipped element as zero height — the scrollbar thumb is wrong and the page jumps when you scroll near a skipped section. `auto 300px` tells the browser to estimate 300px before first render. After the element renders once, `auto` updates the estimate from the actual measured value.

> **Browser support:** Chrome 85+, Edge 85+, Firefox 125+ (April 2024). Safari: not yet. Check [MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/content-visibility) before shipping to production. The feature degrades gracefully — browsers without support just render everything normally.

## Where it actually helps

The benefit scales with how many offscreen elements exist and how expensive each one is to paint:

- Long article-style pages with many sections
- Infinite scroll lists with complex items (images, multiple text nodes, interactive elements)
- Dashboard pages with many cards below the fold on load

For short pages where most content is visible on load: measurable, but not worth the gotchas.

## What breaks

**`getBoundingClientRect()` on skipped elements returns zeros.**

Any code that reads the position or size of a `content-visibility: auto` element that's currently skipped gets garbage values. Carousels that measure item widths, sticky positioning that reads offsets, chart libraries that size from container width — these fail silently.

```javascript
// Returns {width: 0, height: 0, top: 0, ...} if element is skipped
const rect = section.getBoundingClientRect();
```

The fix: force layout before reading with `element.offsetHeight` (which triggers paint), or wrap in an IntersectionObserver that waits for visibility before measuring.

**`IntersectionObserver` still works, but differently.**

Skipped elements aren't invisible to IntersectionObserver — they're in the DOM. Their "visibility" for IO purposes respects containing-block clipping. Test your scroll-triggered animations on pages that use `content-visibility`.

**Find-in-page (Ctrl+F) works, but can be jarring.**

The browser keeps text in the accessibility tree even when skipped, so Ctrl+F finds the text. When it jumps to a match in a skipped section, the browser has to render that section on demand — visually noticeable on pages with many large sections.

**Anchor link navigation can land wrong.**

`<a href="#section-5">` navigates by computing the target's position. If that section is currently skipped and its `contain-intrinsic-size` estimate is off, the scroll position lands in the wrong place. The browser corrects, which creates a visible jump.

## The honest recommendation

Apply `content-visibility: auto` to sections that won't be measured by JavaScript — sections that are independently self-contained and don't need their dimensions read by sibling or ancestor code.

For content pages (articles, documentation, blog posts): safe, effective, worthwhile. For application dashboards with JavaScript-driven layout: test carefully before shipping.

```css
/* Safe for article-style content */
.prose-section {
  content-visibility: auto;
  contain-intrinsic-size: auto 400px;
}

/* Less safe for interactive widget containers — measure impact first */
.dashboard-widget {
  /* test before adding content-visibility here */
}
```

The rendering budget exists whether you use it or not. Using it well is knowing which elements you can afford to defer and which ones have JavaScript waiting on the other side.
