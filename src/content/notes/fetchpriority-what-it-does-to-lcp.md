---
title: "fetchpriority and the art of getting the important thing first"
date: "2024-04-28"
category: Performance
description: "fetchpriority=\"high\" on your hero image sounds obvious. The interaction with lazy loading, preload scanner, and connection ramp-up is less so."
tags:
  - performance
  - lcp
  - images
  - browser
draft: false
---

`fetchpriority="high"` on the LCP image. You've applied it. It might have helped. Or you applied it to the wrong element and gained nothing and didn't notice either way. Here's what's actually happening.

> **Browser support:** Chrome 101+, Firefox 132+ (October 2024), Safari 17.2+. [MDN reference](https://developer.mozilla.org/en-US/docs/Web/API/HTMLImageElement/fetchPriority).

## What `fetchpriority` does

The browser's preload scanner discovers resources before the main parser reaches them and assigns network priorities. Images are `Low` priority by default until the main thread determines they're in the viewport. `fetchpriority="high"` promotes the image to `High` immediately — before the viewport check.

```html
<!-- Without fetchpriority: image discovered as Low, bumped to High when viewport check runs -->
<img src="/hero.jpg" alt="Hero image" width="1200" height="600">

<!-- With fetchpriority: image starts as High immediately -->
<img src="/hero.jpg" alt="Hero image" width="1200" height="600" fetchpriority="high">
```

The gain is real but conditional. If your HTML is small and the viewport check happens fast, the priority promotion barely matters. If your HTML is large — lots of `<head>` content, large `<script>` blocks before the image — the delay between "image discovered" and "image promoted to High" is significant. That's where `fetchpriority` moves the needle.

It also works on `<link rel="preload">` for images that aren't in the initial HTML:

```html
<link rel="preload" as="image" href="/hero.jpg" fetchpriority="high">
```

## The interaction with `loading="lazy"`

Never combine `fetchpriority="high"` with `loading="lazy"` on the same image.

```html
<!-- Contradictory. The browser prioritizes fetching something it's also told to defer. -->
<!-- Behavior is undefined. Don't. -->
<img src="/hero.jpg" fetchpriority="high" loading="lazy">
```

The LCP image is not a lazy-load candidate. It's in the viewport on load. Give it `fetchpriority="high"` and leave `loading` at its default (eager). Nothing else.

## Connection ramp-up: where it gets interesting

On cold connections, HTTP/2 multiplexing means the browser ramps up its congestion window over the first few round trips. `fetchpriority="high"` gets your image into the first burst of requests — the lowest-latency window of the connection.

If five resources are competing in the first burst (fonts, scripts, stylesheets, the LCP image), without `fetchpriority` the image gets queued. With it, the browser favors the image in the first round trip.

The practical impact varies by connection quality. On fast connections, often unmeasurable. On 3G simulation or a cold server, 200–400ms differences are common.

## When it doesn't help

- **If the image is already the highest-priority resource.** Some pages have one large image and minimal other resources. No competition, no benefit.
- **If you're using a CDN with early hints or server push.** Fetch priority is a request-phase hint. If the resource is arriving before the HTML request completes, the hint is moot.
- **If the image is `srcset`-based and the browser hasn't resolved which source to use.** Fetch priority applies to the fetch, not the source selection. Slow source selection is upstream of where `fetchpriority` helps.

## The actual measurement

Use WebPageTest with a real device profile, not Chrome's throttling simulation. The simulator doesn't accurately model connection ramp-up or bufferbloat. Run tests with and without `fetchpriority` on a Moto G4 at 3G Slow and compare the waterfalls.

If the image request starts in the same waterfall position either way, your bottleneck is elsewhere.

`fetchpriority` is a sequencing hint. It helps the browser do what you already wanted — get the important thing first. Getting the important thing first is your job. The browser's job is to not delay it unnecessarily once it knows what it is.
