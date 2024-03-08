---
title: "scrollend: the event we've been debouncing around for years"
date: "2024-03-08"
category: Browser APIs
description: The scroll event fires dozens of times per second. Detecting when it stops has required a debounce hack since the beginning. scrollend is the correction.
tags:
  - javascript
  - browser-api
  - scroll
  - events
draft: false
---

The `scroll` event is one of those things that seems fine until you need to do something when scrolling stops. Then you discover that "stopped" has no native definition, and you spend the next hour writing this:

```javascript
let scrollTimer;

window.addEventListener('scroll', () => {
  clearTimeout(scrollTimer);
  scrollTimer = setTimeout(() => {
    // scroll "stopped"
    doTheThing();
  }, 150);
}, { passive: true });
```

One hundred and fifty milliseconds. Completely arbitrary. Too short and you fire during a slow scroll. Too long and the UI feels sluggish. Some developers use 100ms. Some use 200ms. Some use 250ms. Nobody agrees because there's no right answer — you're guessing at inertia scroll completion.

The `scrollend` event ships in Chrome 114, Firefox 109. It fires when the scroll position has genuinely settled, including after momentum/inertia scroll on mobile and trackpad.

```javascript
window.addEventListener('scrollend', () => {
  // This fires exactly once, after all inertia has resolved.
  doTheThing();
});
```

No timeout. No clearTimeout. No guessing. The browser knows when the scroll actually ended because it controls the inertia physics. Now it tells you.

## A practical example: snap-scroll carousel positioning

The debounce pattern breaks worst with CSS scroll snapping. Snap animation duration varies by device — longer on mobile during a fast fling. A 150ms timeout will often fire before the snap has completed, which means your "after snap" logic runs on the wrong slide.

```javascript
const track = document.querySelector('.carousel-track');
const indicators = document.querySelectorAll('.carousel-indicator');

track.addEventListener('scrollend', () => {
  // Snap has resolved. This is the settled position.
  const activeIndex = Math.round(track.scrollLeft / track.offsetWidth);

  indicators.forEach((dot, i) => {
    dot.classList.toggle('active', i === activeIndex);
  });
});
```

The `scrollend` version works correctly on a slow phone doing a long fling snap. The debounce version didn't.

## On passive events and the browser knowing things we don't

There's a recurring pattern in browser API evolution: the browser always had better access to information about what was happening than JavaScript did, but for years it didn't expose that information in usable form.

`scrollend` is in this category. The browser has known when inertia scroll completed since day one of touch interfaces. It just didn't tell you. You had to approximate.

Same story with `ResizeObserver` (before it, you polled `getBoundingClientRect` in a `requestAnimationFrame` loop). Same story with `IntersectionObserver`. Same story with `scrollend`.

Each one of these APIs is the browser admitting it was withholding information that would have been useful.

## The one thing `scrollend` doesn't do

It doesn't fire if the scroll position didn't change. If a user swipes up and then reverses completely back to the original position, `scrollend` fires because the scroll settled. `scroll` fired the whole way. Neither tells you that the net displacement was zero — that requires you to check the before/after position yourself if you care.

Minor. Worth knowing.

The 150ms debounce had a long run. Retire it.
