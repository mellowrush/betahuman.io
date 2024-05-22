---
title: The page that was already there
date: "2024-05-22"
category: Browser APIs
description: The Speculation Rules API lets you declare which pages to prefetch or prerender. When the user clicks, the page is already rendered. It feels like a faster internet.
tags:
  - performance
  - browser-api
  - navigation
  - prerender
draft: false
---

People don't leave things they've already arrived at.

That's the insight behind loyalty programs, subscription boxes, and every product that makes you feel you're already *there* before you've decided to stay. The psychology of the fait accompli. If the work is done before the decision, the decision is easier.

The Speculation Rules API applies this to page navigation. You declare which URLs to prerender. The browser renders them — fully rendered, painted, JavaScript executed — in a background context. When the user clicks the link, the "navigation" is a display swap. The latency is gone because the work happened when there was time to spare.

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

This is a declarative format. No JavaScript event handlers. No scroll-listener-based prefetch timing. You describe the rules; the browser decides when the network and CPU budget permits execution.

## Eagerness levels

The `eagerness` field controls when speculation triggers:

- **`"immediate"`**: Speculate as soon as the rules are parsed. Aggressive. Useful for pages you're highly confident the user will visit.
- **`"eager"`**: Same as immediate in practice (Chrome currently maps them identically).
- **`"moderate"`**: Speculate when the user hovers over a link for ~200ms or taps on touch devices. The sweet spot for most sites.
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

**Prefetch**: downloads the document and subresources. Parsing, rendering, and JavaScript execution still happen on navigation. Faster, but not instant.

**Prerender**: renders the page completely in a background browsing context. On navigation, Chrome swaps the prerendered context into the tab. Sub-100ms perceived navigation time on a cold server.

The tradeoff: prerender is more expensive. A prerended page executes all its JavaScript, fires analytics events (Chrome has safeguards against double-counting these), and uses real memory. Don't prerender every page on a large site.

## Checking if your page was prerendered

If you have analytics that track page views, you need to know whether a visit came from a prerender or a real navigation. Chrome exposes this:

```javascript
if (document.prerendering) {
  // We are currently being prerendered.
  // Don't fire analytics yet — the user hasn't actually "visited."
  document.addEventListener('prerenderingchange', () => {
    // Now they have. Fire your event.
    analytics.track('pageview');
  });
} else {
  analytics.track('pageview');
}
```

## What it feels like

The first time you implement this on a site with even modest server response times — 200ms to 400ms TTFB — and you click a link that navigates in 50ms, you experience a visceral sense that something about the web has changed. Not improved. Changed. The model is different.

The browser isn't waiting for the server because the server already responded. The server isn't slow — it was just responding to a question that hadn't been asked yet.

That feeling of arrival before departure. That's what this is.

The Speculation Rules API is Chrome-only as of this writing (May 2024), with Firefox and Safari positions still unclear. Treat it as progressive enhancement: pages load normally without it; with it, they load faster than the user expected possible. The improvement is invisible when it's not available. The improvement is memorable when it is.

Ship the script tag. Most of the time, you'll never think about it again. Your users will.
