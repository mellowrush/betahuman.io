---
title: The CSS-only version of what we've been doing wrong
date: "2023-08-15"
category: CSS
description: Scroll-driven animations landed in Chrome 115. Most people still aren't using them. Here's what that says about us.
tags:
  - css
  - animation
  - scroll
  - no-js
draft: false
---

Somewhere around 2019, we collectively decided that scroll animations required JavaScript. Not because that was true. Because that's what we were selling.

Chrome 115 shipped `animation-timeline: scroll()` and `animation-timeline: view()`. Firefox followed in August 2023. At which point the entire Intersection Observer economy became, if not obsolete, at least significantly less necessary.

The premise is simple: instead of tying an animation to time, you tie it to scroll position. The browser does the math. You do the design.

Here's a reading progress bar. The complete implementation:

```css
@keyframes grow-progress {
  from { transform: scaleX(0); }
  to   { transform: scaleX(1); }
}

#progress-bar {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  height: 3px;
  background: var(--accent, #c4522a);
  transform-origin: left;
  animation: grow-progress linear;
  animation-timeline: scroll();
}
```

That's it. No `window.addEventListener('scroll')`. No `requestAnimationFrame`. No percentage calculation. The browser drives it, and it's perfectly smooth because it runs on the compositor thread — the same reason CSS transforms feel faster than their JS equivalents.

The `view()` timeline is more surgical. It tracks whether an element is inside the viewport:

```css
@keyframes fade-in-up {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.section {
  animation: fade-in-up 0.6s ease both;
  animation-timeline: view();
  animation-range: entry 0% entry 30%;
}
```

`animation-range: entry 0% entry 30%` means: start the animation when 0% of the element enters the viewport, finish it when 30% has entered. No IntersectionObserver. No class toggling. No JavaScript thread wakeup cost.

## What it doesn't do yet

Let's be honest about the limitations, because this is a genuinely incomplete story:

- **No Firefox support for `view()` until late 2023.** `scroll()` arrived first; `view()` took longer.
- **Debugging is hard.** Chrome DevTools has rudimentary scroll timeline support, but you're mostly flying without instruments.
- **`animation-timeline: scroll()` only works on scrollable ancestors.** If the scroll happens on `document`, you need `scroll(root)` explicitly.
- **No easy way to trigger "run once."** CSS animations replay every time the element re-enters range. That's sometimes useful, sometimes not what you want.

## The honest verdict

Intersection Observer isn't dead. For "run once when element enters viewport" — the classic fade-in pattern — IO is still cleaner. Scroll-driven animations shine brightest for progress indicators, parallax effects, and sticky-element transitions where you actually want scroll position to drive continuous animation state.

We spent five years installing libraries to do math the browser was always capable of. That's fine. That's how technology works. The point isn't to be embarrassed about the past.

The point is to notice when the past has expired.

```css
@media (prefers-reduced-motion: reduce) {
  * {
    animation-timeline: auto !important;
    animation: none !important;
  }
}
```

Include this. Non-negotiable.
