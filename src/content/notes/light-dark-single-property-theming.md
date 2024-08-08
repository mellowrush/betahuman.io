---
title: "One property. Two modes. No JavaScript."
date: "2024-08-08"
category: CSS
description: light-dark() became baseline in July 2024. One CSS function, one property, two values — the browser picks based on prefers-color-scheme. The JavaScript theme toggle becomes optional.
tags:
  - css
  - dark-mode
  - color
  - design-tokens
draft: false
---

The most successful campaigns don't try to be two things. They find the one idea that contains both possibilities — the single message that sells the product to completely different people for completely different reasons.

`light-dark()` is that kind of solution.

It became baseline in July 2024. The premise: one function that accepts two color values and returns the appropriate one based on whether the system preference is light or dark.

```css
:root {
  color-scheme: light dark;
}

body {
  background: light-dark(#f5f2eb, #1a1814);
  color: light-dark(#1a1814, #f5f2eb);
}
```

That's the entire theme. No `[data-theme="dark"]` attribute selector. No `@media (prefers-color-scheme: dark)` block. No class swap. No `localStorage`. No JavaScript.

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

I want to be honest about the tradeoff, because eliminating JavaScript isn't always an improvement — it's a choice with consequences.

A JavaScript theme toggle (the button on this site) does something `light-dark()` alone cannot: **it lets the user override the system preference**. `light-dark()` responds to `prefers-color-scheme`. If the user's OS is set to light mode but they prefer a dark site, `light-dark()` shows them light.

`color-scheme` can be controlled via CSS and HTML, but not in a way that persists across page loads without `localStorage`. So:

- **Pure `light-dark()` system**: respects OS preference. No override. No JS. Zero friction for users who configure their OS correctly.
- **JS toggle + `[data-theme]` selectors**: respects OS preference *and* allows override. Requires JavaScript. Requires persistence.

The answer depends on your users. A developer tool or technical site: the audience manages their OS themes correctly, pure `light-dark()` is appropriate. A consumer product: many users set their OS to light mode and expect the app to have its own dark mode setting.

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

Without any `data-theme` attribute, the system preference applies via `light-dark()`. When the user toggles, the JS sets `data-theme` and the attribute selectors override. Best of both.

## The `color-scheme` property on individual elements

```css
/* This element always renders in dark mode, regardless of system preference */
.code-block {
  color-scheme: dark;
  background: light-dark(#1e1e1e, #1e1e1e);  /* same either way */
  color: light-dark(#e0e0e0, #e0e0e0);
}
```

When you set `color-scheme` on an element, you're also telling browser-rendered UI inside it (scrollbars, form controls, system elements) which mode to use. The property propagates to UA-rendered content in a way that `background-color` doesn't.

## The experimental: one function to govern them all

The question I ran on this site: if you convert the entire token system to `light-dark()` and remove `[data-theme]` selectors, what breaks?

Answer: almost nothing, if you audit the specificity. The `[data-theme="dark"]` selectors have specificity `[0, 1, 0]` — one attribute selector. That beats `:root` (`[0, 0, 1]`). If your JS toggle overwrites at the `:root` level via custom property re-declaration, the specificity model needs adjustment.

The simpler path: keep the JS toggle, add `light-dark()` as the default values in your token declarations, and let the JS override only where needed. You get graceful degradation (the site works without JS with correct OS-matched colors) and user override capability (the toggle works as before).

One property. Two modes. The rest is preference.
