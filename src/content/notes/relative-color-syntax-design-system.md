---
title: One seed. A whole system.
date: "2024-01-18"
category: CSS
description: "Relative color syntax in CSS means one design token can produce your entire palette. No preprocessor. No JS. No separate color library."
tags:
  - css
  - design-tokens
  - color
  - oklch
draft: false
---

CSS relative color syntax lets you derive new colors mathematically from existing ones — inside CSS, with no preprocessor, no JavaScript, no color library.

```css
:root {
  --brand: oklch(52% 0.18 30);
}

.button {
  background: var(--brand);
  color: oklch(from var(--brand) 98% 0.02 h);               /* near-white tint of brand */
  border-color: oklch(from var(--brand) calc(l - 10%) c h); /* 10% darker */
}

.button:hover {
  background: oklch(from var(--brand) calc(l + 5%) c h);    /* 5% lighter on hover */
}
```

The syntax: `oklch(from var(--brand) l c h)` takes a source color and exposes its lightness, chroma and hue as named variables you can modify arithmetically.

`oklch` is the important choice here. Unlike `hsl`, oklch is *perceptually uniform* — a 10% lightness step looks the same visual distance regardless of where you are in the color range. Yellow at 50% lightness in `hsl` looks much brighter than blue at 50% lightness. oklch corrects for that.

> **Browser support:** Chrome 119+ (October 2023), Safari 16.4+ (March 2023), Firefox 128+ (July 2024). [MDN reference](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_colors/Relative_colors). Use `@supports (color: oklch(from red l c h))` for fallbacks.

## A complete palette from one token

```css
:root {
  --rust: oklch(52% 0.18 30);

  /* Tints — lighter versions */
  --rust-100: oklch(from var(--rust) 96% 0.04 h);
  --rust-200: oklch(from var(--rust) 88% 0.08 h);
  --rust-300: oklch(from var(--rust) 76% 0.14 h);

  /* Shades — darker versions */
  --rust-700: oklch(from var(--rust) calc(l - 15%) c h);
  --rust-800: oklch(from var(--rust) calc(l - 25%) c h);
  --rust-900: oklch(from var(--rust) calc(l - 35%) c h);

  /* Complementary — rotate the hue 180° */
  --rust-complement: oklch(from var(--rust) l c calc(h + 180));

  /* Analogous — adjacent on the wheel */
  --rust-analogous: oklch(from var(--rust) l c calc(h + 30));
}
```

Change `--rust` to any other oklch color. Every derivative updates. No Sass function. No color.js import. No design tool export step.

<div class="rcs-demo" role="region" aria-label="Live demo: relative color palette">
  <style>
    .rcs-demo {
      border: 1px solid var(--bh-border-strong, #d4c8c0);
      border-radius: 8px;
      padding: 1rem 1.25rem;
      margin: 1.75rem 0;
      background: var(--bh-bg, #faf7f4);
    }
    .rcs-demo-label {
      font-family: monospace;
      font-size: 0.7rem;
      letter-spacing: 0.08em;
      text-transform: uppercase;
      color: var(--bh-text-faint, #9a8f87);
      margin-bottom: 0.875rem;
      display: block;
    }
    .rcs-swatches {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(80px, 1fr));
      gap: 0.375rem;
    }
    .rcs-swatch {
      border-radius: 4px;
      height: 48px;
      display: flex;
      align-items: flex-end;
      padding: 0.25rem 0.375rem;
    }
    .rcs-swatch span {
      font-family: monospace;
      font-size: 0.65rem;
      color: oklch(from currentColor 20% 0.02 h);
      mix-blend-mode: multiply;
    }
    /* All derived from one seed */
    :root { --demo-seed: oklch(52% 0.18 30); }
    .rcs-100 { background: oklch(from var(--demo-seed) 96% 0.04 h); }
    .rcs-200 { background: oklch(from var(--demo-seed) 88% 0.08 h); }
    .rcs-300 { background: oklch(from var(--demo-seed) 76% 0.14 h); }
    .rcs-500 { background: var(--demo-seed); }
    .rcs-700 { background: oklch(from var(--demo-seed) calc(l - 15%) c h); }
    .rcs-800 { background: oklch(from var(--demo-seed) calc(l - 25%) c h); }
    .rcs-900 { background: oklch(from var(--demo-seed) calc(l - 35%) c h); }
    .rcs-comp { background: oklch(from var(--demo-seed) l c calc(h + 180)); }
  </style>
  <span class="rcs-demo-label">One seed (oklch 52% 0.18 30) → entire palette via relative color</span>
  <div class="rcs-swatches">
    <div class="rcs-swatch rcs-100"><span>100</span></div>
    <div class="rcs-swatch rcs-200"><span>200</span></div>
    <div class="rcs-swatch rcs-300"><span>300</span></div>
    <div class="rcs-swatch rcs-500"><span>500</span></div>
    <div class="rcs-swatch rcs-700"><span>700</span></div>
    <div class="rcs-swatch rcs-800"><span>800</span></div>
    <div class="rcs-swatch rcs-900"><span>900</span></div>
    <div class="rcs-swatch rcs-comp"><span>comp</span></div>
  </div>
</div>

## The alpha channel

```css
.overlay {
  /* 40% opacity version of the brand color */
  background: oklch(from var(--brand) l c h / 40%);
}
```

This is the readable replacement for the `color-mix()` alpha trick:

```css
/* color-mix approach */
background: color-mix(in oklch, var(--brand) 40%, transparent);

/* relative color approach */
background: oklch(from var(--brand) l c h / 40%);
```

Both work. The relative color version expresses the intent directly: "this color at 40% opacity."

## What you can't do yet

- **Fallbacks for older browsers** require `@supports (color: oklch(from red l c h))` — verbose but functional.
- **Derived colors in custom properties** require `@property` registration if you want them to animate (the browser needs to know the type).
- **HSL relative colors** work too, but don't use them. Perceptual uniformity is the point.

The preprocessor solved a real problem. So did the color library. Both introduced a compile step, a build artifact, a dependency version to maintain. When the platform gives you the same result without the overhead, the honest move is to use it.

One seed. A whole system.
