---
title: The screen is the color source
date: "2024-11-07"
category: Browser APIs
description: "The EyeDropper API lets you sample any pixel on the screen from a web page. Combined with CSS relative colors, you can build a palette tool that generates harmonies from anything the user can see."
tags:
  - browser-api
  - color
  - tool
  - javascript
  - css
draft: false
---

The EyeDropper API, available in Chrome 95+, puts an OS-level color picker inside a web application. It samples any pixel on the screen — not just within the browser, but anywhere the cursor can reach. The user clicks, the screen briefly enters sampling mode, the cursor becomes a magnifier and the selected color comes back as a hex value.

```javascript
async function pickColor() {
  if (!window.EyeDropper) {
    alert('EyeDropper API not supported in this browser.');
    return;
  }

  const eyeDropper = new EyeDropper();

  try {
    const result = await eyeDropper.open();
    return result.sRGBHex; // e.g., '#c4522a'
  } catch (err) {
    // User cancelled (pressed Escape)
    return null;
  }
}
```

`eyeDropper.open()` returns a promise that resolves when the user clicks, or rejects when they cancel with Escape. The selected color is a hex string.

> **Browser support:** Chrome 95+. Firefox: implementation in progress. Safari: not committed. Check [MDN](https://developer.mozilla.org/en-US/docs/Web/API/EyeDropper). Include a `<input type="color">` fallback for non-Chrome browsers — see the fallback section below.

## Building a harmony generator

The useful combination: EyeDropper for sampling, CSS relative colors for generating a full palette from the seed color.

```javascript
async function generatePalette() {
  const hex = await pickColor();
  if (!hex) return;

  // Convert hex to oklch for perceptually uniform manipulation
  const seed = hexToOklch(hex);

  // Set as a CSS custom property — CSS relative colors handle the rest
  document.documentElement.style.setProperty('--seed', oklchToCss(seed));
}

function hexToOklch(hex) {
  // hex → linearized sRGB → LMS cone space → OKLab → OKLCh
  const r = parseInt(hex.slice(1, 3), 16) / 255;
  const g = parseInt(hex.slice(3, 5), 16) / 255;
  const b = parseInt(hex.slice(5, 7), 16) / 255;

  // Linearize (gamma correction)
  const rl = r > 0.04045 ? Math.pow((r + 0.055) / 1.055, 2.4) : r / 12.92;
  const gl = g > 0.04045 ? Math.pow((g + 0.055) / 1.055, 2.4) : g / 12.92;
  const bl = b > 0.04045 ? Math.pow((b + 0.055) / 1.055, 2.4) : b / 12.92;

  // sRGB → LMS (M1 matrix from OKLab spec)
  const l  = 0.4122214708 * rl + 0.5363325363 * gl + 0.0514459929 * bl;
  const m  = 0.2119034982 * rl + 0.6806995451 * gl + 0.1073969566 * bl;
  const s  = 0.0883024619 * rl + 0.2817188376 * gl + 0.6299787005 * bl;

  // Cube root
  const l_ = Math.cbrt(l);
  const m_ = Math.cbrt(m);
  const s_ = Math.cbrt(s);

  // LMS^(1/3) → OKLab (M2 matrix)
  const Lab_l = 0.2104542553 * l_ + 0.7936177850 * m_ - 0.0040720468 * s_;
  const Lab_a = 1.9779984951 * l_ - 2.4285922050 * m_ + 0.4505937099 * s_;
  const Lab_b = 0.0259040371 * l_ + 0.7827717662 * m_ - 0.8086757660 * s_;

  // OKLab → OKLCh (polar form)
  const C = Math.sqrt(Lab_a ** 2 + Lab_b ** 2);
  const H = Math.atan2(Lab_b, Lab_a) * 180 / Math.PI;

  return { l: Lab_l * 100, c: C, h: H < 0 ? H + 360 : H };
}

function oklchToCss({ l, c, h }) {
  return `oklch(${l.toFixed(1)}% ${c.toFixed(3)} ${h.toFixed(1)})`;
}
```

Once the seed is set as a CSS custom property, the palette expands through CSS:

```css
:root {
  --seed: oklch(52% 0.18 30); /* set dynamically by JS */

  /* Tints */
  --palette-50:  oklch(from var(--seed) 96% 0.03 h);
  --palette-100: oklch(from var(--seed) 90% 0.06 h);
  --palette-200: oklch(from var(--seed) 82% 0.10 h);
  --palette-300: oklch(from var(--seed) 72% 0.14 h);

  /* Base */
  --palette-500: oklch(from var(--seed) l c h);

  /* Shades */
  --palette-700: oklch(from var(--seed) calc(l - 15%) c h);
  --palette-800: oklch(from var(--seed) calc(l - 25%) c h);
  --palette-900: oklch(from var(--seed) calc(l - 35%) c h);

  /* Complement — 180° rotation */
  --palette-complement: oklch(from var(--seed) l c calc(h + 180));

  /* Analogous — adjacent hues */
  --palette-warm:  oklch(from var(--seed) l c calc(h - 30));
  --palette-cool:  oklch(from var(--seed) l c calc(h + 30));
}
```

When the user picks a new color, one `setProperty` call updates `--seed` and every derived color updates through CSS's own cascade. No re-running a color library. No generating twenty properties. One source, everything derived.

## What `<input type="color">` doesn't do

`<input type="color">` is native, accessible and useful. But it's limited to the browser window. EyeDropper reaches the whole screen.

The design workflow this enables: open a reference image in one window, open the tool in another, sample directly from the reference. No Figma, no separate color picker, no translation step. The screen is the palette.

## The gap: Firefox and Safari

EyeDropper is Chrome-only as of this writing (November 2024). Firefox implementation is in progress. Safari hasn't committed.

For tools targeting designers and developers — a category that skews Chrome — this is manageable. Include the graceful fallback:

```javascript
async function pickColorWithFallback() {
  if (window.EyeDropper) {
    const dropper = new EyeDropper();
    const result = await dropper.open().catch(() => null);
    return result?.sRGBHex ?? null;
  }

  // Fallback: hidden color input
  return new Promise(resolve => {
    const input = document.createElement('input');
    input.type = 'color';
    input.addEventListener('input', () => resolve(input.value), { once: true });
    input.click();
  });
}
```

EyeDropper gives you the pixel. Relative colors give you the system. What you build with it is the interesting part.
