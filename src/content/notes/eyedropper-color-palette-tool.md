---
title: The screen is the color source
date: "2024-11-07"
category: Browser APIs
description: The EyeDropper API lets you sample any pixel on the screen from a web page. Combined with CSS relative colors, you can build a palette tool that generates harmonies from anything the user can see.
tags:
  - browser-api
  - color
  - tool
  - javascript
  - css
draft: false
---

Every great image has a palette embedded in it. The problem has never been finding the colors — a designer can look at a photograph and identify its dominant tones in seconds. The problem has been extracting them into a system without losing something in translation.

The EyeDropper API, available in Chrome 95+, puts an OS-level color picker inside a web application. It samples any pixel on the screen — not just within the browser, but anywhere the cursor can reach. The user clicks, the screen briefly enters a sampling mode, the cursor becomes a magnifier, and the selected color is returned as a hex value.

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

`eyeDropper.open()` returns a promise that resolves when the user clicks or rejects when they cancel with Escape. The selected color is a hex string.

## Building a harmony generator

The interesting combination: EyeDropper for sampling, CSS relative colors for generating a full palette from the seed.

```javascript
async function generatePalette() {
  const hex = await pickColor();
  if (!hex) return;

  // Convert hex to oklch for perceptually uniform manipulation
  const seed = hexToOklch(hex);

  // Define the palette as CSS custom properties
  document.documentElement.style.setProperty('--seed', oklchToCss(seed));

  // CSS relative colors handle the rest — no JS color math needed
}

function hexToOklch(hex) {
  // Parse hex → sRGB → linear sRGB → XYZ D65 → OKLab → OKLCh
  // (This is ~30 lines of standard color math)
  const r = parseInt(hex.slice(1, 3), 16) / 255;
  const g = parseInt(hex.slice(3, 5), 16) / 255;
  const b = parseInt(hex.slice(5, 7), 16) / 255;

  // Linearize
  const rl = r > 0.04045 ? Math.pow((r + 0.055) / 1.055, 2.4) : r / 12.92;
  const gl = g > 0.04045 ? Math.pow((g + 0.055) / 1.055, 2.4) : g / 12.92;
  const bl = b > 0.04045 ? Math.pow((b + 0.055) / 1.055, 2.4) : b / 12.92;

  // sRGB → XYZ → OKLab (simplified)
  const l  = 0.4122214708 * rl + 0.5363325363 * gl + 0.0514459929 * bl;
  const m  = 0.2119034982 * rl + 0.6806995451 * gl + 0.1073969566 * bl;
  const s  = 0.0883024619 * rl + 0.2817188376 * gl + 0.6299787005 * bl;

  const l_ = Math.cbrt(l);
  const m_ = Math.cbrt(m);
  const s_ = Math.cbrt(s);

  const Lab_l = 0.2104542553 * l_ + 0.7936177850 * m_ - 0.0040720468 * s_;
  const Lab_a = 1.9779984951 * l_ - 2.4285922050 * m_ + 0.4505937099 * s_;
  const Lab_b = 0.0259040371 * l_ + 0.7827717662 * m_ - 0.8086757660 * s_;

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

  /* Analogous — adjacent */
  --palette-warm:  oklch(from var(--seed) l c calc(h - 30));
  --palette-cool:  oklch(from var(--seed) l c calc(h + 30));
}
```

When the user picks a new color, one `setProperty` call updates `--seed`, and every derived color updates through CSS's own cascade. No re-running a color library. No generating and setting twenty properties. One source, everything else derived.

## What the `<input type="color">` doesn't do

`<input type="color">` is useful. It's native, accessible, and has supported a `list` attribute for preset swatches since Chrome 99. But it's limited to the browser window. EyeDropper reaches the whole screen.

The design workflow: open a reference image in one window, open the tool in another, sample directly from the reference. No color picker, no Figma, no separate application. The screen is the palette.

## The gap: Firefox and Safari

EyeDropper is Chrome-only as of this writing (November 2024). Mozilla has positioned, the Firefox implementation status is in progress. Safari hasn't committed.

For a tool that presents itself as a power feature for designers and developers — a category that skews Chrome — this is manageable. Include a graceful fallback: `<input type="color">` as the alternative, `EyeDropper` as the enhancement.

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

The screen has always been the color source. A good eye has always been enough to identify what's there. What was missing was the extraction step — the moment of translation from pixel to value to system.

EyeDropper gives you the pixel. Relative colors give you the system. What you build with it is the interesting part.
