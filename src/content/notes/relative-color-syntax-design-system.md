---
title: One seed. A whole system.
date: "2024-01-18"
category: CSS
description: Relative color syntax in CSS means one design token can produce your entire palette. No preprocessor. No JS. No separate color library.
tags:
  - css
  - design-tokens
  - color
  - oklch
draft: false
---

Every great advertising campaign starts with a single insight — one true thing about the product that, once found, generates everything else. The tagline, the visual language, the thirty-second spot, the billboard. One seed. A whole system.

That's what CSS relative color syntax is, for color.

It shipped in Chrome 119, Firefox 128, and Safari 16.4. It allows you to derive new colors mathematically from existing ones — right inside CSS, with no preprocessor, no JavaScript, no color library.

```css
:root {
  --brand: oklch(52% 0.18 30);
}

.button {
  background: var(--brand);
  color: oklch(from var(--brand) 98% 0.02 h);          /* near-white tint of brand */
  border-color: oklch(from var(--brand) calc(l - 10%) c h); /* 10% darker */
}

.button:hover {
  background: oklch(from var(--brand) calc(l + 5%) c h);  /* 5% lighter on hover */
}
```

The syntax: `oklch(from var(--brand) l c h)` takes a source color and exposes its lightness, chroma, and hue as named variables. You can then modify any of them arithmetically.

`oklch` is the important choice here. Unlike `hsl`, oklch is *perceptually uniform* — a 10% lightness step looks the same visual distance regardless of where you are in the color range. `hsl` isn't. Yellow at 50% lightness looks much brighter than blue at 50% lightness. `oklch` corrects for that.

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

Change `--rust` to any other oklch color. Every derivative updates automatically. No Sass function. No color.js import. No design tool export step.

## The alpha channel

```css
.overlay {
  /* 40% opacity version of the brand color */
  background: oklch(from var(--brand) l c h / 40%);
}
```

This is the replacement for the `color-mix()` alpha trick:

```css
/* Old approach */
background: color-mix(in oklch, var(--brand) 40%, transparent);

/* New approach */
background: oklch(from var(--brand) l c h / 40%);
```

Both work. The relative color version is more readable because the intent — "this color at 40% opacity" — is expressed directly in the value.

## What you can't do yet

This is a live experiment, not a shipping recommendation without caveats:

- **Fallbacks for older browsers** require `@supports (color: oklch(from red l c h))` — which is verbose but functional.
- **Derived colors in custom properties** require `@property` registration if you want them to animate (the browser needs to know the type).
- **HSL relative colors** work too, but don't use them. The perceptual uniformity of oklch is the point.

The preprocessor solved a real problem. So did the color library. But both introduced a compile step, a build artifact, a dependency version to maintain.

When the platform gives you the same result without the overhead, the honest move is to use it.

One seed. A whole system.
