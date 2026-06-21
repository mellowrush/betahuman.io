---
title: "Font pairing isn't a specimen problem"
date: "2025-02-06"
category: Weird
description: "Every font pairing tool shows you specimens in a vacuum. Typefaces read completely differently at UI scale, in dark mode, at 13px. Here's the pairing lab I've been building to fix that."
tags:
  - typography
  - css
  - tools
  - design
  - fonts
draft: false
---

Every font pairing website operates on the same premise: show two typefaces at large display size, in neutral beige, in ideal lighting conditions, doing the most flattering thing they know how to do. 

This is like choosing a suit based on how it looks on a mannequin in a window. The mannequin doesn't commute. It doesn't sit down. Its proportions are aspirational in a way that will not survive contact with your actual shoulders.

Typefaces behave. They have moods at different sizes. A serif that reads beautifully at 48px becomes illegible at 13px. A geometric sans that looks clean and modern in a specimen looks cold and unfriendly in a form label. You don't discover this until the font is in production and the design review has moved on.

The pairing lab I've been building for the past month shows combinations in context. Not "Heading / Body" in a specimen. Actual components:

- A card with a title, metadata, and body text
- A navigation bar with primary and secondary links  
- A form with labels, inputs, and helper text
- A data table with column headers and cell content
- A code block (mono pairing matters here)

Toggle dark mode. Change the base size. Adjust the line height. See how the combination holds up when the conditions change.

## The technical approach

Loading fonts dynamically from Google Fonts API:

```javascript
async function loadFontPair(display, body) {
  const url = `https://fonts.googleapis.com/css2?family=${
    encodeURIComponent(display)
  }:ital,wght@0,400;0,700;1,400&family=${
    encodeURIComponent(body)
  }:ital,wght@0,400;0,500;0,700;1,400&display=swap`;

  // Check if already loaded
  if (document.querySelector(`link[href="${url}"]`)) return;

  return new Promise((resolve, reject) => {
    const link = document.createElement('link');
    link.rel = 'stylesheet';
    link.href = url;
    link.onload = resolve;
    link.onerror = reject;
    document.head.appendChild(link);
  });
}

async function applyPairing(display, body) {
  await loadFontPair(display, body);

  document.documentElement.style.setProperty('--font-display', `"${display}", serif`);
  document.documentElement.style.setProperty('--font-body', `"${body}", sans-serif`);
}
```

The component kit uses CSS custom properties for font families:

```css
.preview-card h2 {
  font-family: var(--font-display);
  font-size: clamp(1.2rem, 3vw, 1.8rem);
  font-weight: 400;
  letter-spacing: -0.02em;
}

.preview-card p {
  font-family: var(--font-body);
  font-size: 0.9375rem;
  line-height: 1.6;
}

.preview-nav .primary-link {
  font-family: var(--font-body);
  font-size: 0.875rem;
  font-weight: 500;
  letter-spacing: 0.01em;
}

.preview-form label {
  font-family: var(--font-body);
  font-size: 0.8125rem;
  font-weight: 500;
}
```

Font swap is a style property update. No page reload. Instant feedback.

## The "bad pairing" detector

This is the opinionated part that makes the tool more than a playground.

The heuristics I've implemented:

**1. Contrast ratio check**: Display and body fonts should be visually distinct. Two geometric sans-serifs next to each other don't create hierarchy — they create sameness. The detector computes "personality distance" between fonts by classification (serif/sans/display/mono) and weight range.

**2. x-height matching**: Fonts with mismatched x-heights look unbalanced at the same `font-size`. A font with a tall x-height next to one with a short x-height needs `font-size` compensation to match optical size.

**3. Stroke contrast**: High-stroke-contrast serifs (Didot, Bodoni) pair badly with heavy geometric sans-serifs because both are visually demanding. The eye has nowhere to rest.

```javascript
function scorePairing(displayFont, bodyFont) {
  const issues = [];

  // Classification contrast check
  if (displayFont.classification === bodyFont.classification &&
      displayFont.classification !== 'slab-serif') {
    issues.push({
      severity: 'warning',
      message: `Both fonts are ${displayFont.classification}. Consider a serif/sans contrast.`
    });
  }

  // x-height ratio check
  const xHeightRatio = displayFont.xHeight / bodyFont.xHeight;
  if (xHeightRatio > 1.3 || xHeightRatio < 0.7) {
    issues.push({
      severity: 'info',
      message: `x-height difference is significant. Body font may need size adjustment.`
    });
  }

  // Stroke contrast check
  if (displayFont.strokeContrast === 'high' && bodyFont.strokeContrast === 'high') {
    issues.push({
      severity: 'warning',
      message: `Both fonts have high stroke contrast. Consider a low-contrast body font.`
    });
  }

  return issues;
}
```

The font metadata (classification, x-height, stroke contrast) comes from a curated JSON file. The detector doesn't claim to be right. It claims to be a useful second opinion.

## What I haven't figured out yet

The Google Fonts API approach works for the 1,500+ fonts they host. Anything outside that — commercially licensed fonts, variable fonts from other sources, uploaded WOFF2 files — requires a different approach. The `FontFace` API handles this, but the UX for "upload your font" is more complex.

```javascript
// For custom fonts: FontFace API
const font = new FontFace('CustomFont', 'url(/fonts/custom.woff2)');
await font.load();
document.fonts.add(font);
```

The tool currently works for Google Fonts exploration. The custom-font path is stubbed.

The other gap: the detector heuristics are based on type classification metadata I assembled manually for about 200 fonts. Scaling that coverage requires either crowdsourcing or pulling from a type data API that doesn't currently exist in a form I can use.

Both problems are solvable. Neither is urgent. The tool is useful at its current scope.

The specimen is a lie by omission. The context is where the pairing actually happens. If you're going to choose a typeface, at least look at it in the situation it'll have to live in.
