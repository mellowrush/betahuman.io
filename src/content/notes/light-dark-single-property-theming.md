---
title: "One property. Two modes. No JavaScript."
date: "2024-08-08"
category: CSS
description: "light-dark() became baseline in July 2024. One CSS function, one property, two values — the browser picks based on prefers-color-scheme. The JavaScript theme toggle becomes optional."
tags:
  - css
  - dark-mode
  - color
  - design-tokens
draft: false
---

`light-dark()` became Baseline in July 2024. One function, two color values — the browser picks based on whether the system preference is light or dark.

```css
:root {
  color-scheme: light dark;
}

body {
  background: light-dark(#f5f2eb, #1a1814);
  color: light-dark(#1a1814, #f5f2eb);
}
```

No `[data-theme="dark"]` attribute selector. No `@media (prefers-color-scheme: dark)` block. No class swap. No `localStorage`. No JavaScript.

> **Browser support:** Baseline July 2024. Chrome 123+, Firefox 120+, Safari 17.5+. [MDN reference](https://developer.mozilla.org/en-US/docs/Web/CSS/color_value/light-dark). The `color-scheme: light dark` declaration on `:root` is required — without it, `light-dark()` resolves to its light value in all contexts.

<div class="ld-demo" role="region" aria-label="Live demo: light-dark() color tokens">
  <style>
    .ld-demo {
      border: 1px solid var(--bh-border-strong, #d4c8c0);
      border-radius: 8px;
      padding: 1rem 1.25rem;
      margin: 1.75rem 0;
      background: var(--bh-bg, #faf7f4);
    }
    .ld-demo-label {
      font-family: monospace;
      font-size: 0.7rem;
      letter-spacing: 0.08em;
      text-transform: uppercase;
      color: var(--bh-text-faint, #9a8f87);
      margin-bottom: 0.875rem;
      display: block;
    }
    .ld-swatches {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(120px, 1fr));
      gap: 0.5rem;
    }
    .ld-swatch {
      border-radius: 4px;
      padding: 0.625rem 0.75rem;
      font-family: monospace;
      font-size: 0.7rem;
    }
    /* These change with prefers-color-scheme */
    .ld-swatch-bg       { background: light-dark(#f5f2eb, #1a1814); color: light-dark(#1a1814, #f5f2eb); }
    .ld-swatch-elevated { background: light-dark(#ffffff, #252118); color: light-dark(#1a1814, #e8e3d8); }
    .ld-swatch-muted    { background: light-dark(#e8e3d8, #2a261e); color: light-dark(#5a554c, #9a9388); }
    .ld-swatch-accent   { background: light-dark(#c4522a, #e06a3a); color: #fff; }
    .ld-swatch-border   { background: light-dark(#e8e3d8, #2e2a20); color: light-dark(#5a554c, #9a9388); }
  </style>
  <span class="ld-demo-label">↓ Switch your OS to dark mode — swatches update automatically</span>
  <div class="ld-swatches">
    <div class="ld-swatch ld-swatch-bg">--bg</div>
    <div class="ld-swatch ld-swatch-elevated">--bg-elevated</div>
    <div class="ld-swatch ld-swatch-muted">--bg-muted</div>
    <div class="ld-swatch ld-swatch-accent">--accent</div>
    <div class="ld-swatch ld-swatch-border">--border</div>
  </div>
</div>

## A full design token system with light-dark()

```css
:root {
  color-scheme: light dark;

  --bg:           light-dark(#f5f2eb, #1a1814);
  --bg-elevated:  light-dark(#ffffff, #252118);
  --bg-muted:     light-dark(#e8e3d8, #2a261e);
  --text:         light-dark(#1a1814, #e8e3d8);
  --text-muted:   light-dark(#5a554c, #9a9388);
  --text-faint:   light-dark(#6a6560, #706a60);
  --border:       light-dark(#e8e3d8, #2e2a20);
  --accent:       light-dark(#c4522a, #e06a3a);
}
```

One `color-scheme` declaration on `:root`. The browser reads the system preference and resolves every `light-dark()` call accordingly.

## What the JavaScript toggle buys you

Eliminating JavaScript isn't always an improvement. It's a choice with consequences.

A JavaScript theme toggle does something `light-dark()` alone cannot: it lets the user override the system preference. `light-dark()` responds to `prefers-color-scheme`. If the user's OS is set to light mode but they prefer a dark site, `light-dark()` shows them light.

- **Pure `light-dark()`**: respects OS preference. No override. No JS. Zero friction for users who configure their OS correctly.
- **JS toggle + `[data-theme]` selectors**: respects OS preference *and* allows override. Requires JavaScript and persistence.

The answer depends on your users. A developer tool: the audience manages their OS themes — pure `light-dark()` is appropriate. A consumer product: many users set their OS to light and expect an in-app dark mode setting.

## Hybrid: `light-dark()` as fallback, JS toggle as override

```css
:root {
  color-scheme: light dark;

  /* light-dark() provides the default */
  --bg: light-dark(#f5f2eb, #1a1814);
}

/* JS toggle overrides via data-theme attribute */
[data-theme="dark"] {
  --bg: #1a1814;
  color-scheme: dark;
}

[data-theme="light"] {
  --bg: #f5f2eb;
  color-scheme: light;
}
```

Without any `data-theme` attribute, the system preference applies via `light-dark()`. When the user toggles, JS sets `data-theme` and the attribute selectors override. Best of both.

## The `color-scheme` property on individual elements

```css
/* This element always renders in dark mode, regardless of system preference */
.code-block {
  color-scheme: dark;
  background: #1e1e1e;
  color: #e0e0e0;
}
```

Setting `color-scheme` on an element also tells browser-rendered UI inside it (scrollbars, form controls, system elements) which mode to use. The property propagates to UA-rendered content in a way that `background-color` doesn't.

## What breaks when you go pure light-dark()

If you convert the entire token system to `light-dark()` and remove `[data-theme]` selectors, specificity needs auditing. `[data-theme="dark"]` has specificity `[0, 1, 0]` — one attribute selector — which beats `:root`'s `[0, 0, 1]`. If your JS toggle overwrites at the `:root` level via custom property re-declaration, the specificity model needs adjustment.

The simpler path: keep the JS toggle, add `light-dark()` as the default values in your token declarations and let the JS override only where needed. Graceful degradation (the site works without JS, with correct OS-matched colors) plus user override capability (the toggle works as before).

One property. Two modes. The rest is preference.
