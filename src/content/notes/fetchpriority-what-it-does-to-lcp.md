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

There's a problem with every piece of advice that sounds obvious: the less obvious the mechanism, the more confidently people follow the rule without understanding when it stops working.

`fetchpriority="high"` on the LCP image. You've heard it. You've applied it. It might have helped. Or you applied it wrong and it made things marginally worse and you didn't notice either way.

Here's what's actually happening.

## What `fetchpriority` does

The browser's preload scanner discovers resources before the main parser reaches them. It assigns them network priorities. Images, by default, are `Low` priority until the main thread determines they're in the viewport. `fetchpriority="high"` promotes the image to `High` priority immediately — before the viewport check.

```html
<!-- Without fetchpriority: image discovered as Low, bumped to High when viewport check runs -->
<img src="/hero.jpg" alt="Hero image" width="1200" height="600">

<!-- With fetchpriority: image starts as High immediately -->
<img src="/hero.jpg" alt="Hero image" width="1200" height="600" fetchpriority="high">
```

The gain is real but conditional. The preload scanner is fast. If your HTML is small and simple, the viewport check happens quickly enough that the priority promotion barely matters. If your HTML is large — lots of `<head>` content, large `<script>` blocks before the image — the delay between "image discovered" and "image promoted to High" is significant. That's where `fetchpriority` moves the needle.

## The interaction with `loading="lazy"`

Never combine `fetchpriority="high"` with `loading="lazy"` on the same image.

```html
<!-- This is contradictory. The browser will prioritize fetching something it's also told to defer. -->
<!-- Behavior is undefined. Don't. -->
<img src="/hero.jpg" fetchpriority="high" loading="lazy">
```

The LCP image is not a lazy-load candidate. It's in the viewport on load. Give it neither `loading="lazy"` nor explicit `loading="eager"` (eager is the default). Just `fetchpriority="high"`.

## Connection ramp-up: where it gets interesting

On slow or cold connections, HTTP/2 multiplexing means the browser ramps up congestion window over the first few round trips. `fetchpriority="high"` gets your image into the first burst of requests — which means it benefits from the lowest-latency window of the connection.

A concrete example: if you have five resources competing in the first burst (fonts, scripts, stylesheets, the LCP image), without `fetchpriority` the image gets queued. With it, the browser favors the image in the first round trip.

The practical impact varies by connection quality. On fast connections, you often can't measure it. On 3G simulation or a cold server, you can see 200–400ms differences.

## When it doesn't help

- **If the image is already the highest-priority resource on the page.** Some pages have one large image and minimal other resources. `fetchpriority` adds nothing because there's no competition.
- **If you're using a CDN with predictive push.** A properly configured CDN with server push or early hints may already be sending the image before the HTML request has even completed. Fetch priority is a request-phase hint; if the resource is arriving before the request, the hint is moot.
- **If the image is `srcset`-based and the browser hasn't resolved which source to use.** Fetch priority applies to the fetch, not the source selection. If the source selection is slow, that delay is upstream of where `fetchpriority` helps.

## The actual measurement

Use WebPageTest with a real device profile, not Chrome's throttling simulation. The throttling simulation doesn't accurately model connection ramp-up or bufferbloat. Run three tests with and without `fetchpriority` on a Moto G4 at 3G Slow and look at the waterfall.

If the image request starts in the same waterfall position with or without the attribute, your bottleneck is elsewhere.

`fetchpriority` is not a magic LCP fix. It is a sequencing hint that helps the browser do what you already wanted it to do — get the important thing first. The important thing being first is your job. The browser's job is to not delay it unnecessarily once it knows.
