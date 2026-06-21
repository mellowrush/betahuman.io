---
title: The page that was already there
date: "2024-05-22"
category: Browser APIs
description: "The Speculation Rules API lets you declare which pages to prefetch or prerender. When the user clicks, the page is already rendered. It feels like a faster internet."
tags:
  - performance
  - browser-api
  - navigation
  - prerender
draft: false
---

The Speculation Rules API lets you declare which URLs to prerender. The browser renders them — fully painted, JavaScript executed — in a background context. When the user clicks the link, navigation is a display swap. The latency is gone because the work happened when there was time to spare.

```html
<script type="speculationrules">
{
  "prerender": [
    {
      "where": {
        "href_matches": "/*"
      },
      "eagerness": "moderate"
    }
  ]
}
</script>
```

Declarative format. No JavaScript event handlers. No scroll-listener-based prefetch timing. You describe the rules; the browser decides when the network and CPU budget permits.

> **Browser support:** Chrome 109+ (Speculation Rules), prerender in Chrome 121+. Firefox and Safari: no support as of mid-2024. Check [MDN](https://developer.mozilla.org/en-US/docs/Web/API/Speculation_Rules_API) for current status. Treat as progressive enhancement — pages load normally without it.

## Eagerness levels

The `eagerness` field controls when speculation triggers:

- **`"immediate"`**: Speculate as soon as the rules are parsed. Aggressive. For pages you're highly confident the user will visit.
- **`"eager"`**: Same as immediate in Chrome's current implementation.
- **`"moderate"`**: Speculate when the user hovers over a link for ~200ms, or on tap on touch devices. The sweet spot for most sites.
- **`"conservative"`**: Only on pointerdown or touchstart. Safest. Least effective.

For a documentation site where the next page is almost always "the next article in a series," `moderate` is correct. For a homepage where the user might go anywhere, `conservative` avoids burning network on pages they never visit.

## Prerender vs. prefetch

These are meaningfully different:

```html
<script type="speculationrules">
{
  "prefetch": [
    { "where": { "href_matches": "/about" }, "eagerness": "immediate" }
  ],
  "prerender": [
    { "where": { "href_matches": "/notes/*" }, "eagerness": "moderate" }
  ]
}
</script>
```

**Prefetch**: downloads the document and subresources. Parsing, rendering and JavaScript execution still happen on navigation. Faster, but not instant.

**Prerender**: renders the page completely in a background browsing context. On navigation, Chrome swaps the prerendered context into the tab. Sub-100ms perceived navigation time on a cold server.

The tradeoff: prerender is expensive. A prerendered page executes all its JavaScript, fires analytics events (Chrome has safeguards against double-counting these) and uses real memory. Don't prerender every page on a large site.

## Checking if your page was prerendered

If you have analytics that track page views, you need to know whether a visit came from prerender or real navigation:

```javascript
if (document.prerendering) {
  // Currently being prerendered — user hasn't actually "visited" yet
  document.addEventListener('prerenderingchange', () => {
    // Now they have
    analytics.track('pageview');
  });
} else {
  analytics.track('pageview');
}
```

## What it feels like

The first time you implement this on a site with even modest server response times — 200ms to 400ms TTFB — and you click a link that navigates in 50ms, something feels different about the web. Not faster in the usual sense. Different. The model is different.

The browser isn't waiting for the server because the server already responded. The server isn't slow — it was just responding to a question that hadn't been asked yet.

Ship the script tag. Most of the time, you'll never think about it again. Your users will.
