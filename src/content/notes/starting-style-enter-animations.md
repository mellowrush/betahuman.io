---
title: "@starting-style: the DOM hack retirement party"
date: "2023-10-05"
category: CSS
description: We've been setting opacity to 0, waiting a frame, then setting it to 1 for years. The browser finally handles this natively.
tags:
  - css
  - animation
  - display
draft: false
---

Here's something we've been doing since approximately forever:

```javascript
// The hack
element.style.opacity = '0';
document.body.appendChild(element);

requestAnimationFrame(() => {
  requestAnimationFrame(() => {
    // Two rAFs because one isn't enough for some browsers.
    // Nobody agrees on why. We just all started doing this.
    element.style.opacity = '1';
  });
});
```

Two nested `requestAnimationFrame` calls. The outer one hands off to the next paint. The inner one hands off to the one after that. We've been doing this dance because CSS transitions can't animate from "doesn't exist yet" to "exists." There's no start state.

`@starting-style`, which shipped in Chrome 117 and Firefox 129, fixes this at the source.

```css
.toast {
  opacity: 1;
  transform: translateY(0);
  transition: opacity 250ms ease, transform 250ms ease;
}

@starting-style {
  .toast {
    opacity: 0;
    transform: translateY(-8px);
  }
}
```

The `@starting-style` block defines where the transition starts when an element first becomes visible — whether that's from `display: none`, from `visibility: hidden`, or from not existing in the DOM at all. The browser reads it as: "before you paint this for the first time, treat these as the initial values."

## The `display: none` transition problem

The reason this matters more than you might think: for years, you couldn't animate to or from `display: none`. You could transition opacity to 0, but the element remained interactive (it's invisible, not gone). You could use `visibility: hidden`, which doesn't interact, but `visibility` transitions are instant — no easing.

The actual solution most of us used was height: 0 with overflow: hidden and a JS measurement step. An embarrassing amount of production code is running this right now.

Chrome 116 shipped `transition-behavior: allow-discrete`, which allows `display` and `visibility` to participate in transitions. Combined with `@starting-style`, you get:

```css
.dialog {
  display: block;
  opacity: 1;
  transition: opacity 200ms ease, display 200ms allow-discrete;
}

.dialog[hidden] {
  display: none;
  opacity: 0;
}

@starting-style {
  .dialog {
    opacity: 0;
  }
}
```

When `hidden` is removed, the dialog transitions in. When `hidden` is added, it fades out and then `display: none` applies at the end. The browser handles the sequencing.

## What this means for headless component libraries

The bundle size argument for headless animation libraries (Framer Motion, @headlessui's Dialog, etc.) just got weaker. Not dead — they still handle complex choreography, spring physics, and layout animations — but "enter/exit animations for a modal" is now approximately five lines of CSS.

I ran the comparison on a small project last month. The animation behavior for a toast notification stack went from 180 lines of JavaScript with a dependency to:

```css
@keyframes toast-enter {
  from { transform: translateX(110%); }
  to   { transform: translateX(0); }
}

.toast {
  animation: toast-enter 240ms cubic-bezier(0.22, 0.61, 0.36, 1);
  transition: opacity 180ms ease, display 180ms allow-discrete;
}

.toast.is-dismissing {
  opacity: 0;
}

@starting-style {
  .toast {
    transform: translateX(110%);
    opacity: 0;
  }
}
```

The JS becomes: add class `is-dismissing` → wait 180ms → remove from DOM. Three lines.

I'm not claiming CSS is always the better tool for animation. I'm claiming we handed JavaScript a problem it didn't need to own, and we've been maintaining that mistake ever since.

`@starting-style` is the receipt.
