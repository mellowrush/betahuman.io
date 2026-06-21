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

At some point you tried to animate a CSS custom property with a transition and nothing happened:

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

The browser jumped from 0 to 0.8. No interpolation. The transition did nothing.

The reason: CSS custom properties are opaque to the browser. Without knowing the type, the browser can't interpolate. Interpolating between "apple" and "orange" is undefined. So it skips it.

`@property` registers the custom property with a type:

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

Now the browser knows `--glow-opacity` is a number. It interpolates. The transition works.

> **Browser support:** Baseline 2024. Chrome 85+ (October 2020), Safari 16.4+ (March 2023), Firefox 128+ (June 2024). [MDN reference](https://developer.mozilla.org/en-US/docs/Web/CSS/@property).

## A rotating gradient without JavaScript

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
    var(--rust, #c4522a) 0deg,
    var(--paper, #faf7f4) 90deg,
    var(--rust, #c4522a) 180deg,
    var(--paper, #faf7f4) 270deg,
    var(--rust, #c4522a) 360deg
  );
  animation: spin-gradient 4s linear infinite;
}
```

Without `@property`, `--gradient-angle` can't be animated — the browser doesn't know it's an angle. With it, the `conic-gradient` spins smoothly. No JavaScript. No `requestAnimationFrame`.

<div class="prop-demo" role="region" aria-label="Live demo: @property animated gradient">
  <style>
    .prop-demo {
      border: 1px solid var(--bh-border-strong, #d4c8c0);
      border-radius: 8px;
      padding: 1rem 1.25rem;
      margin: 1.75rem 0;
      background: var(--bh-bg, #faf7f4);
      display: flex;
      align-items: center;
      gap: 1.25rem;
    }
    .prop-demo-label {
      font-family: monospace;
      font-size: 0.7rem;
      letter-spacing: 0.08em;
      text-transform: uppercase;
      color: var(--bh-text-faint, #9a8f87);
      flex: 1;
    }
    @property --prop-angle {
      syntax: '<angle>';
      inherits: false;
      initial-value: 0deg;
    }
    @keyframes prop-spin {
      to { --prop-angle: 360deg; }
    }
    .prop-spinner {
      width: 72px;
      height: 72px;
      border-radius: 50%;
      background: conic-gradient(
        from var(--prop-angle),
        #c4522a 0deg,
        #faf7f4 90deg,
        #c4522a 180deg,
        #faf7f4 270deg,
        #c4522a 360deg
      );
      animation: prop-spin 3s linear infinite;
      flex-shrink: 0;
    }
    @media (prefers-reduced-motion: reduce) {
      .prop-spinner { animation: none; }
    }
  </style>
  <div class="prop-spinner" aria-hidden="true"></div>
  <span class="prop-demo-label">
    <code>@property --gradient-angle { syntax: '&lt;angle&gt;' }</code><br>
    No JS. No rAF. CSS animates the angle directly.
  </span>
</div>

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
// Update progress — one property, eased transition
ring.style.setProperty('--ring-progress', '75');
```

The `conic-gradient` updates with an eased transition. The visual is a smooth-filling arc. One property update, one line of JavaScript.

## The `syntax` values you'll actually use

```css
@property --my-color   { syntax: '<color>';      inherits: true;  initial-value: black; }
@property --my-length  { syntax: '<length>';     inherits: false; initial-value: 0px; }
@property --my-number  { syntax: '<number>';     inherits: false; initial-value: 0; }
@property --my-angle   { syntax: '<angle>';      inherits: false; initial-value: 0deg; }
@property --my-percent { syntax: '<percentage>'; inherits: false; initial-value: 0%; }
```

The `inherits` flag matters: if `true`, the property follows the CSS cascade like `color` or `font-size`. If `false`, it doesn't inherit from parent to child — useful for properties that are component-local.

The four years we spent waiting for Firefox is, in the way of these things, already starting to feel like it happened to someone else.

Ship it.
