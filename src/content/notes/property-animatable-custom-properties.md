---
title: "@property: the one that makes custom properties actually animate"
date: "2024-06-18"
category: CSS
description: "CSS custom properties can't animate because the browser doesn't know their type. @property declares the type. After that, they animate, transition, and interpolate like native values."
tags:
  - css
  - animation
  - custom-properties
  - property
draft: false
---

Let me walk you through a failure so you understand why this matters.

At some point in the last few years, you probably tried to animate a CSS custom property with a transition and nothing happened. You had something like:

```css
.element {
  --glow-opacity: 0;
  box-shadow: 0 0 20px rgba(196, 82, 42, var(--glow-opacity));
  transition: --glow-opacity 300ms ease;
}

.element:hover {
  --glow-opacity: 0.8;
}
```

Didn't work. The browser jumped from 0 to 0.8 with no interpolation. The transition appeared to do nothing.

The reason: CSS custom properties are opaque to the browser. The browser doesn't know if `--glow-opacity` is a number, a color, a length, or a string. Without knowing the type, it can't interpolate. Interpolating between "apple" and "orange" is undefined behavior. So the browser skips it.

`@property` solves this by registering the custom property with a type:

```css
@property --glow-opacity {
  syntax: '<number>';
  inherits: false;
  initial-value: 0;
}

.element {
  box-shadow: 0 0 20px rgba(196, 82, 42, var(--glow-opacity));
  transition: --glow-opacity 300ms ease;
}

.element:hover {
  --glow-opacity: 0.8;
}
```

Now the browser knows `--glow-opacity` is a number between (presumably) 0 and 1. It can interpolate. The transition works.

## A rotating gradient without JavaScript

This is the demo that makes people stop and check the source code:

```css
@property --gradient-angle {
  syntax: '<angle>';
  inherits: false;
  initial-value: 0deg;
}

@keyframes spin-gradient {
  to { --gradient-angle: 360deg; }
}

.spinning-border {
  background: conic-gradient(
    from var(--gradient-angle),
    var(--rust) 0deg,
    var(--paper) 90deg,
    var(--rust) 180deg,
    var(--paper) 270deg,
    var(--rust) 360deg
  );
  animation: spin-gradient 4s linear infinite;
}
```

Without `@property`, `--gradient-angle` can't be animated because the browser doesn't know it's an angle. With it, the `conic-gradient` spins smoothly. No JavaScript. No `requestAnimationFrame`. No manual angle tracking.

## A progress ring

```css
@property --ring-progress {
  syntax: '<number>';
  inherits: false;
  initial-value: 0;
}

.progress-ring {
  --ring-progress: 0;
  background: conic-gradient(
    var(--accent) calc(var(--ring-progress) * 1%),
    var(--surface-muted) 0
  );
  border-radius: 50%;
  width: 80px;
  height: 80px;
  transition: --ring-progress 600ms cubic-bezier(0.22, 0.61, 0.36, 1);
}
```

```javascript
// Update progress
ring.style.setProperty('--ring-progress', '75');
```

The `conic-gradient` updates with an eased transition. The visual is a smooth-filling arc. One property update, one line of JavaScript.

## The `syntax` values you'll actually use

The `syntax` field accepts a subset of CSS value types:

```css
@property --my-color   { syntax: '<color>';  inherits: true;  initial-value: black; }
@property --my-length  { syntax: '<length>'; inherits: false; initial-value: 0px; }
@property --my-number  { syntax: '<number>'; inherits: false; initial-value: 0; }
@property --my-angle   { syntax: '<angle>';  inherits: false; initial-value: 0deg; }
@property --my-percent { syntax: '<percentage>'; inherits: false; initial-value: 0%; }
```

The `inherits` flag matters: if `true`, the property follows the CSS cascade like `color` or `font-size`. If `false`, it doesn't inherit from parent to child — useful for properties that are component-local.

## Firefox baseline

`@property` has been in Chrome since version 85 (October 2020). Firefox finally shipped it in version 128 (June 2024). That cross-browser gap is what kept it out of production for most teams. As of this month, it's baseline.

The four years we spent waiting is, in the way of these things, already starting to feel like it happened to someone else. Ship it.
