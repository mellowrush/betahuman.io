---
title: "content-visibility: the rendering budget you didn't know you had"
date: "2024-07-03"
category: Performance
description: content-visibility: auto tells the browser to skip rendering offscreen content until it's needed. On long pages, this changes the numbers significantly. Here's what actually breaks when you use it.
tags:
  - performance
  - css
  - rendering
  - paint
draft: false
---

There's a version of this article I could write that opens with a benchmark showing 4× faster initial paint on a list of 500 items. That version would be true, slightly misleading, and would end with you applying `content-visibility: auto` to everything and discovering that several things broke in ways nobody warned you about.

Let's skip to the honest version.

`content-visibility: auto` tells the browser: "if this element is far outside the viewport, skip layout, paint, and compositing for it. Don't skip it entirely — keep it in the DOM, keep it accessible, keep it in the find-in-page index — but don't pay the rendering cost until it might actually be visible."

```css
.article-section {
  content-visibility: auto;
  contain-intrinsic-size: auto 300px;
}
```

The `contain-intrinsic-size` is load-bearing. Without it, the browser renders every skipped element as if it has zero height, which means the scrollbar thumb is the wrong size and the page jumps when you scroll near a skipped section (the browser suddenly has to calculate real height). `auto 300px` tells the browser: "before you've rendered this, estimate its height at 300px." After first render, `auto` updates the estimate from the actual measured value.

## Where it actually helps

The benefit is proportional to how many offscreen elements exist and how expensive each one is to paint. The sweet spot:

- Long article-style pages with many sections
- Infinite scroll lists with complex list items (images, multiple text nodes, interactive elements)
- Dashboard pages with many cards that are below the fold on load

For short pages or pages where most content is visible on load: measurable but unimportant.

## What breaks

This is the section that wasn't in the Chrome announcement:

**1. `getBoundingClientRect()` on skipped elements returns zeros.**

If any code reads the position or size of a `content-visibility: auto` element that's currently skipped, it gets garbage values. Any carousel that measures item widths, any sticky positioning that reads offsets, any chart library that sizes itself from container width — these will fail silently.

```javascript
// Returns {width: 0, height: 0, top: 0, ...} if element is skipped
const rect = section.getBoundingClientRect();
```

The fix: force layout before reading, using `element.offsetHeight` (which forces the browser to paint it) or by wrapping in an IntersectionObserver that waits for visibility.

**2. `IntersectionObserver` still works, but differently.**

Skipped elements aren't invisible to IntersectionObserver — they're in the DOM. But their "visibility" for IO purposes considers the containing-block clipping. Test your scroll-triggered animations on pages that use `content-visibility`.

**3. Find-in-page (Ctrl+F) works, but slowly.**

The browser keeps text in the accessibility tree even when skipped, so Ctrl+F finds the text. But when it jumps to a match in a skipped section, the browser has to suddenly render that section, which can be visually jarring on pages with many large sections.

**4. Anchor link navigation can jump wrong.**

`<a href="#section-5">` navigates by computing the target's position. If that section is currently skipped and its `contain-intrinsic-size` estimate is wrong, the scroll position lands in the wrong place. The browser then corrects, which creates a visible jump.

## The honest recommendation

Apply `content-visibility: auto` to sections you're confident won't be measured by JavaScript — typically sections that are independently self-contained and don't need their dimensions read by sibling or ancestor code.

For most content pages (articles, documentation, blog posts): safe, effective, worthwhile. For application dashboards with JavaScript-driven layout: test carefully.

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

The performance gain is real. The four-times-faster headline can be true for the right page. The question is whether your page has JavaScript that assumes all DOM elements have real dimensions at all times — and most application-style pages do.

The rendering budget exists whether you use it or not. Using it well is just a matter of knowing which elements you can afford to defer.
