---
title: What Chrome 115 made optional
date: "2023-08-15"
category: CSS
description: "Scroll-driven animations landed in Chrome 115. Most people still aren't using them. Here's what that says about us."
tags:
  - css
  - animation
  - scroll
  - no-js
draft: false
---

The browser couldn't do this, so we wrote JavaScript instead. Scroll listeners. rAF loops. IntersectionObserver callbacks. A small industry grew up around the gap.

Chrome 115 arrived in July 2023. `animation-timeline: scroll()`. `animation-timeline: view()`. Firefox 115 the same month. Safari is still giving the matter its full consideration.

The entire Intersection Observer consulting economy became, depending on your position, obsolete or newly optional.

The idea is this: instead of tying an animation to time, you tie it to scroll position. The browser does the math. You write the design. The JavaScript file stays closed.

> **Browser support (August 2023):** Chrome 115+, Firefox 115+. Safari: not supported. Check before shipping to production — there's no graceful degradation unless you write it.

## Reading progress. No event listeners.

```css
@keyframes grow-progress {
  from { transform: scaleX(0); }
  to   { transform: scaleX(1); }
}

#progress-bar {
  position: fixed;
  top: 0; left: 0; right: 0;
  height: 3px;
  background: var(--accent, #c4522a);
  transform-origin: left;
  animation-name: grow-progress;
  animation-duration: 1ms; /* required — Firefox ignores scroll timelines without a duration */
  animation-timing-function: linear;
  animation-timeline: scroll(root);
}
```

No `window.addEventListener('scroll')`. No `requestAnimationFrame`. No percentage arithmetic. The compositor thread runs it — same reason CSS transforms feel better than their JavaScript equivalents, because they are.

`scroll(root)` targets the document scroller. `scroll()` without arguments targets the nearest scrollable ancestor — useful inside overflow containers, which is what the demo below uses.

<div class="sda-demo sda-demo--progress" role="region" aria-label="Live demo: scroll-driven progress bar">
  <style>
    .sda-demo {
      border: 1px solid var(--bh-border-strong, #d4c8c0);
      border-radius: 8px;
      margin: 1.75rem 0;
      overflow: hidden;
      position: relative;
      background: var(--bh-bg, #faf7f4);
    }
    .sda-demo--progress {
      height: 210px;
      overflow-y: scroll;
    }
    .sda-progress-bar {
      position: sticky;
      top: 0; left: 0;
      width: 100%;
      height: 3px;
      background: #c4522a;
      transform-origin: left;
      animation-name: sda-grow;
      animation-duration: 1ms;
      animation-timing-function: linear;
      animation-timeline: scroll();
    }
    @keyframes sda-grow {
      from { transform: scaleX(0); }
      to   { transform: scaleX(1); }
    }
    .sda-scroll-content {
      padding: 0.75rem 1.25rem 1.5rem;
    }
    .sda-demo-label {
      font-family: monospace;
      font-size: 0.7rem;
      letter-spacing: 0.08em;
      text-transform: uppercase;
      color: var(--bh-text-faint, #9a8f87);
      margin-bottom: 0.875rem;
      display: block;
    }
    .sda-prose {
      font-size: 0.875rem;
      line-height: 1.65;
      color: var(--bh-text-muted, #6b5e57);
      margin-bottom: 0.875rem;
    }
    .sda-prose:last-child { margin-bottom: 0; }
    @media (prefers-reduced-motion: reduce) {
      .sda-progress-bar {
        animation: none !important;
        transform: scaleX(1) !important;
        opacity: 0.35;
      }
    }
  </style>
  <div class="sda-progress-bar" role="progressbar" aria-label="Reading progress"></div>
  <div class="sda-scroll-content">
    <span class="sda-demo-label">↓ Scroll inside this box — Chrome &amp; Firefox 115+</span>
    <p class="sda-prose"><code>animation-timeline: scroll()</code> tracks the nearest scrollable ancestor. As you scroll down, the bar at the top grows. Nothing on the main thread. The compositor advances the animation directly from scroll position — no wakeup cost, no rAF loop.</p>
    <p class="sda-prose">For a real page-level bar you'd use <code>scroll(root)</code> instead, which targets the document scroller rather than the element's nearest scrollable ancestor (which in this demo is the box itself).</p>
    <p class="sda-prose"><code>transform-origin: left</code> is load-bearing here. Without it, <code>scaleX()</code> expands from center. That's the kind of thing you only forget once.</p>
    <p class="sda-prose">Note the explicit <code>animation-duration: 1ms</code>. Firefox ignores scroll-driven animations if no duration is set. Chrome is more forgiving. Write the 1ms anyway.</p>
  </div>
</div>

## The surgical version: `view()`

`scroll()` tracks the page. `view()` tracks *an element* — whether it's inside the viewport and by exactly how much.

```css
@keyframes fade-in-up {
  from { opacity: 0; transform: translateY(20px); }
  to   { opacity: 1; transform: translateY(0); }
}

.section {
  animation-name: fade-in-up;
  animation-duration: 1ms; /* required for Firefox */
  animation-timing-function: ease;
  animation-fill-mode: both;
  animation-timeline: view();
  animation-range: entry 0% entry 30%;
}
```

`animation-range: entry 0% entry 30%` means: start the animation when the element's leading edge enters the scrollport, finish it when 30% of the element has entered. No IntersectionObserver. No class toggling. No JavaScript thread wakeup cost.

<div class="sda-demo sda-demo--view" role="region" aria-label="Live demo: view-driven animations">
  <style>
    .sda-demo--view {
      height: 260px;
      overflow-y: scroll;
    }
    .sda-view-label {
      padding: 0.75rem 1.25rem 0.375rem;
      font-family: monospace;
      font-size: 0.7rem;
      letter-spacing: 0.08em;
      text-transform: uppercase;
      color: var(--bh-text-faint, #9a8f87);
      display: block;
    }
    .sda-view-item {
      display: flex;
      align-items: center;
      gap: 0.625rem;
      margin: 0.375rem 1.25rem;
      padding: 0.625rem 0.875rem;
      background: var(--bh-bg-muted, #f0ebe6);
      border-radius: 6px;
      font-size: 0.8125rem;
      font-family: monospace;
      color: var(--bh-text, #2a1f1b);
      animation-name: sda-fade-up;
      animation-duration: 1ms;
      animation-timing-function: ease;
      animation-fill-mode: both;
      animation-timeline: view();
      /* cover spans item-height + container-height (~304px total scroll travel).
         cover 0%–40% gives ~120px to complete the fade — slow enough to actually see. */
      animation-range: cover 0% cover 40%;
    }
    .sda-view-item::before {
      content: '';
      display: block;
      width: 6px; height: 6px;
      border-radius: 50%;
      background: #c4522a;
      flex-shrink: 0;
      opacity: 0.7;
    }
    /* spacer must exceed container height minus label so all items start below the fold */
    .sda-view-spacer {
      height: 220px;
      display: flex;
      align-items: center;
      justify-content: center;
      font-family: monospace;
      font-size: 0.7rem;
      letter-spacing: 0.06em;
      text-transform: uppercase;
      color: var(--bh-text-faint, #9a8f87);
    }
    @keyframes sda-fade-up {
      from { opacity: 0; transform: translateY(16px); }
      to   { opacity: 1; transform: translateY(0); }
    }
    @media (prefers-reduced-motion: reduce) {
      .sda-view-item {
        animation: none !important;
        opacity: 1 !important;
        transform: none !important;
      }
    }
  </style>
  <span class="sda-view-label">↓ Scroll down — Chrome &amp; Firefox 115+</span>
  <div class="sda-view-spacer">↓ keep scrolling ↓</div>
  <div class="sda-view-item">animation-timeline: view()</div>
  <div class="sda-view-item">animation-range: entry 0% entry 80%</div>
  <div class="sda-view-item">No IntersectionObserver</div>
  <div class="sda-view-item">No class toggling on scroll</div>
  <div class="sda-view-item">No JavaScript thread wakeup</div>
  <div class="sda-view-item">Compositor-driven. Pure CSS.</div>
</div>

## The honest ledger

This is a genuinely incomplete story and I'd rather say so than pretend it isn't:

- **Safari doesn't support it.** As of August 2023, zero support. Check [MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/animation-timeline) before shipping.
- **Firefox requires `animation-duration: 1ms`.** Scroll-driven animations are ignored without an explicit duration. Chrome is forgiving about this; Firefox isn't. Write the `1ms`.
- **Debugging is primitive.** Chrome DevTools has rudimentary scroll timeline support. You are mostly flying without instruments, which is fine until it isn't.
- **`scroll()` defaults to the nearest scrollable ancestor.** For the document scroller, you need `scroll(root)` explicitly. Easy to overlook.
- **No clean "run once" primitive.** CSS animations replay every time the element re-enters range. Sometimes that's what you want. Often it isn't.

## Who this replaces — and who it doesn't

Intersection Observer isn't dead. For "run once when element enters viewport" — the standard entrance pattern — IO is still the cleaner tool. You fire a class and walk away. That's a real advantage.

Scroll-driven animations shine where you want scroll position to *continuously drive state*: progress bars, parallax, sticky-header transitions, scrubbing effects. If the animation belongs to a moment, use IO. If it belongs to a position, use this.

We spent five years installing libraries to do arithmetic the browser was always capable of. That's fine. That's how the industry works.

The point is to notice when the invoice has expired.

---

And include this. Non-negotiable:

```css
@media (prefers-reduced-motion: reduce) {
  * {
    animation-timeline: auto !important;
    animation: none !important;
  }
}
```
