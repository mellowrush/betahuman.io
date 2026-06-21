---
title: A resume that scales
date: "2025-04-02"
category: Weird
description: "A PDF resume is a snapshot. An SVG resume is an argument. Here's what building one taught me about text layout, SVG font metrics and what a resume is actually trying to do."
tags:
  - svg
  - typography
  - experiment
  - tools
draft: false
---

A resume is an advertisement. It is the only form of advertising you will ever produce in which the product and the producer are the same person and the buyer is sitting directly across a table from you when they read it.

The format, then, is part of the message.

Most resumes produce PDFs. The PDF is correct for most uses: print-ready, universally readable, font-embedded. But it is also static, pixel-bound and tells the viewer exactly one thing about the person inside it: they know how to use a template or can afford a designer.

I've been building an SVG resume. Not because SVG is better in every dimension — it isn't. Because the constraints of building in SVG force decisions that a word processor hides and those forced decisions are interesting.

## What SVG gives you that PDF doesn't

- **Infinite scalability**: the document renders correctly at business card size and at poster size from the same source.
- **Inspectability**: open the file in any text editor. The content is human-readable XML.
- **Animatability**: SVG can contain CSS animations. A resume that subtly animates certain elements on hover — when viewed in a browser — is a different artifact than a resume that doesn't.
- **Embeddability**: drop it in an HTML `<img>` tag. It renders correctly, scales correctly and costs nothing in additional markup.

## The first constraint: text layout

SVG doesn't have block layout. There's no flow model. Text in SVG is positioned absolutely:

```xml
<text x="0" y="0" font-family="DM Serif Display" font-size="28" fill="#1a1814">
  Bogdan Deaconescu
</text>
```

For a heading, this is fine. For paragraphs, this is a problem. SVG `<text>` elements with multiple lines require explicit `<tspan>` elements with `x` and `dy` attributes for each line:

```xml
<text x="40" y="200" font-family="system-ui" font-size="13" fill="#5a554c">
  <tspan x="40" dy="0">Self-taught frontend developer with six years building</tspan>
  <tspan x="40" dy="18">product interfaces across advertising and product teams.</tspan>
  <tspan x="40" dy="18">Now focused on the platform itself.</tspan>
</text>
```

There is no automatic line wrapping. You calculate where lines break, or you generate the SVG programmatically from a data source.

I chose programmatic generation:

```javascript
function wrapText(ctx, text, x, y, maxWidth, lineHeight) {
  const words = text.split(' ');
  const lines = [];
  let currentLine = '';

  for (const word of words) {
    const testLine = currentLine + (currentLine ? ' ' : '') + word;
    const metrics = ctx.measureText(testLine);

    if (metrics.width > maxWidth && currentLine) {
      lines.push(currentLine);
      currentLine = word;
    } else {
      currentLine = testLine;
    }
  }
  if (currentLine) lines.push(currentLine);

  return lines.map((line, i) =>
    `<tspan x="${x}" dy="${i === 0 ? 0 : lineHeight}">${escapeXml(line)}</tspan>`
  ).join('\n');
}
```

Using an off-screen `<canvas>` context to measure text width — `ctx.measureText()` — is the only reliable way to calculate where lines break with an arbitrary font. Font metrics vary. `measureText` uses the actual rendered font.

```javascript
// The escapeXml helper — SVG text content can't contain raw < > & characters
function escapeXml(str) {
  return str
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;');
}
```

## The selectable text requirement

SVG `<text>` elements are selectable and copy-pastable, which is the critical constraint. A resume that produces unsearchable or uncopable text fails the ATS scanner and the clipboard use case. The SVG approach satisfies both — the text is real XML text, not rasterized glyphs.

```xml
<!-- This is searchable, selectable and accessible -->
<text class="job-title" x="40" y="145">
  Senior Frontend Developer
</text>

<!-- This is NOT — don't do this -->
<!-- <image href="title-rasterized.png" x="40" y="130"/> -->
```

## The animation layer

Viewed in a browser, the SVG can contain CSS animations:

```css
.timeline-connector {
  stroke-dasharray: 100;
  stroke-dashoffset: 100;
  animation: draw-line 600ms ease forwards;
  animation-delay: calc(var(--job-index, 0) * 150ms);
}

@keyframes draw-line {
  to { stroke-dashoffset: 0; }
}
```

Timeline connectors between job entries draw in sequentially. Section headers fade up. The document is alive when viewed in a browser, inert when printed or saved as a PDF (via `@media print`).

```css
@media print {
  * { animation: none !important; }
}
```

Print renders the final state — no animation, clean output.

## What I haven't solved

The honest inventory of what remains hard:

**Hyphenation**: SVG has no hyphenation support. Long words either overflow or break awkwardly. For English resumes with typical word lengths this is manageable. For German compound nouns it's a problem.

**Justified text**: SVG can approximate it by computing word spacing per line, but the result doesn't match browser text justification quality. I defaulted to left-aligned text.

**Variable fonts**: SVG supports variable fonts in modern browsers. But when the SVG is opened in Illustrator or Figma, variable font axes are often ignored. The resume should look correct in both contexts. I used static font weights as a result.

**Font embedding**: For offline or PDF-export reliability, fonts need to be embedded (base64) in the SVG. Embedded fonts significantly increase file size.

The SVG resume isn't finished. It works, it's mine and it scales. The decision to build it as an argument rather than a template is the point — a resume that is indistinguishable from ten thousand others argues that the person inside it is also indistinguishable from ten thousand others.

Whether the SVG approach is the right answer to that argument is a separate question. The constraint of building it taught me things about text layout and typographic precision that building a PDF never would have.

That's usually how the interesting problems work.
