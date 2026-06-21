---
title: "QR codes have 30% error correction. I intended to use it."
date: "2025-01-15"
category: Weird
description: "QR codes embed error correction that lets 30% of the code be corrupted and still scan. That's a canvas. Here's what I built and what I learned."
tags:
  - canvas
  - qr-code
  - experiment
  - art
  - svg
draft: false
---

QR codes have error correction baked in — not as a feature, as a requirement. The spec mandates four levels: L (7%), M (15%), Q (25%), H (30%). The code was designed for industrial environments where a label might get smudged or partially obscured.

Error correction level H means 30% of the code's data codewords can be wrong and it will still scan.

Thirty percent is a lot of real estate.

The thought experiment: replace 30% of the QR pattern with deliberate image content, keep the rest intact. You'd have a QR code that is also, visually, something else — a logotype, an icon, a piece of art. And it would still scan.

This is not a new idea. There are apps and services that do this. The interesting question — the one I couldn't find a clean answer to — is whether you can do it **client-side, purely in `<canvas>` or SVG, with a clean API**.

## The QR code structure

A QR code has several structural regions that are not part of the error-correctable data:

- **Finder patterns**: the three squares in the corners. Non-negotiable. Touch these and it won't scan.
- **Timing patterns**: alternating black/white strips between finder patterns. Required.
- **Alignment patterns**: appear in larger QR codes. Required.
- **Format information**: stores error correction level and mask pattern. Required.
- **Data + error correction**: everything else. Up to 30% of this is replaceable.

The safe zone for artistic modification is the data region. Everything outside the structural markers can be replaced.

## A client-side implementation

The approach: generate the QR matrix, identify the structural zones, overlay image content in the data zones.

```javascript
// npm install qrcode-generator
import QRCode from 'qrcode-generator';

function generateArtisticQR(canvas, url, imageSource) {
  const qr = QRCode(0, 'H');  // error correction level H
  qr.addData(url);
  qr.make();

  const ctx = canvas.getContext('2d');
  const size = qr.getModuleCount();
  const cellSize = canvas.width / size;

  // Draw the standard QR code first
  for (let row = 0; row < size; row++) {
    for (let col = 0; col < size; col++) {
      ctx.fillStyle = qr.isDark(row, col) ? '#000' : '#fff';
      ctx.fillRect(col * cellSize, row * cellSize, cellSize, cellSize);
    }
  }

  // Overlay the artistic image — error correction handles the rest
  const img = new Image();
  img.onload = () => {
    ctx.save();

    // Center the overlay, sized to ~35% of the QR (stays within error correction budget)
    const overlaySize = canvas.width * 0.35;
    const x = (canvas.width - overlaySize) / 2;
    const y = (canvas.height - overlaySize) / 2;

    ctx.clearRect(x, y, overlaySize, overlaySize);
    ctx.drawImage(img, x, y, overlaySize, overlaySize);

    ctx.restore();
  };
  img.src = imageSource;
}
```

This gets you a QR code with an image centered in the data zone. The finder patterns (corners) are untouched. Error correction at level H absorbs the modifications.

## What I actually found when I tested it

It works. With caveats.

**Reliably scans when:** the overlay occupies under 28% of the total module count and stays clear of finder/timing/alignment patterns.

**Fails when:** the overlay has high contrast at its edges. Sharp borders between the overlay and QR pattern confuse some scanners. A soft-edged or circular overlay helps significantly.

**Scanner variability:** iPhone native camera (iOS 16+) is more tolerant than older QR readers. Zebra industrial scanners — used in retail and logistics — are strict about the spec and fail artistic codes more often.

**The sweet spot:** a circular logo in the center, sized to roughly 28-30% of the total width, with rounded or feathered edges. The most common commercial implementation (QR with a company logo center) is built on exactly this principle.

## The SVG approach

Canvas gives pixel-level control. SVG gives scalable output:

```javascript
function generateSVGQR(url, logoSrc) {
  const qr = QRCode(0, 'H');
  qr.addData(url);
  qr.make();

  const size = qr.getModuleCount();
  const cellSize = 10;
  const totalSize = size * cellSize;

  let paths = '';
  for (let row = 0; row < size; row++) {
    for (let col = 0; col < size; col++) {
      if (qr.isDark(row, col)) {
        paths += `<rect x="${col * cellSize}" y="${row * cellSize}" width="${cellSize}" height="${cellSize}" fill="black"/>`;
      }
    }
  }

  // Center logo — clipped to a circle
  const logoSize = totalSize * 0.28;
  const logoX = (totalSize - logoSize) / 2;
  const logoY = (totalSize - logoSize) / 2;
  const r = logoSize / 2;

  return `
    <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 ${totalSize} ${totalSize}">
      <defs>
        <clipPath id="logo-clip">
          <circle cx="${logoX + r}" cy="${logoY + r}" r="${r}"/>
        </clipPath>
      </defs>
      <rect width="${totalSize}" height="${totalSize}" fill="white"/>
      ${paths}
      <image href="${logoSrc}" x="${logoX}" y="${logoY}"
             width="${logoSize}" height="${logoSize}"
             clip-path="url(#logo-clip)"/>
    </svg>
  `;
}
```

> **Note:** The original version used a CSS `circle()` function directly in the `clip-path` attribute of the SVG `<image>`. This relies on CSS clip-path support in SVG elements, which is inconsistent across browsers. The version above uses a proper SVG `<clipPath>` element instead — more verbose but universally supported.

The SVG version scales infinitely. The output is a single string — trivial to export, embed, or download.

## What I didn't finish

I built a working prototype. What I didn't build was the thing worth shipping: a clean UI that lets someone provide a URL and a logo and generates a verified-scannable artistic QR code with a "test scan before you download" step.

That step is the important part. Without it, someone will generate a code, print it on a thousand business cards, and discover at the coffee shop that their logo was 31% coverage and it doesn't scan.

That part is still in the todo list. It requires integrating a client-side QR scanner (`zxing-js` works) that scans the generated canvas before letting you export. Simple in theory. I got distracted by the SVG approach and haven't circled back.

This is what building-in-public is actually for: the incomplete thing documented completely.
