---
title: The first letter matters. It always has.
date: "2024-04-14"
category: CSS
description: "initial-letter is now baseline. Native drop caps, properly baseline-aligned, no hacks. Two lines of CSS that took decades to arrive."
tags:
  - css
  - typography
  - editorial
draft: false
---

The first word of a great novel is a promise. Not about plot, not about character — about the *contract* the author is willing to enter. Before the reader understands anything, they have already understood: this person takes language seriously.

Illuminated manuscripts understood this in the ninth century. Every significant chapter, every opening stanza — the initial letter was drawn large, adorned, weight-bearing. The reader's eye landed on it first. The rest of the text flowed around it like a river around a stone.

CSS got `initial-letter` baseline support in 2024. Better late.

```css
p::first-letter {
  initial-letter: 3;
  font-family: Georgia, serif;
  font-weight: 700;
  color: var(--accent, #c4522a);
  margin-right: 0.1em;
}
```

The number `3` means the drop cap occupies three lines of body text, with its baseline aligned to the baseline of the third line. The browser handles the geometry. Previously, you'd approximate this with:

```css
/* The hack that was never quite right */
p::first-letter {
  font-size: 4em;
  float: left;
  line-height: 0.8;        /* guess */
  margin-top: 0.1em;       /* guess */
  margin-right: 0.05em;    /* guess */
  padding: 0.05em;         /* guess */
}
```

Every value was a guess calibrated to a specific font, a specific line height, a specific body font size. Change any of those three things and your drop cap was misaligned.

`initial-letter: 3` is not a guess. It is a specification. The browser baseline-aligns the cap to the third line, regardless of font, size, or line height. The geometry follows from the content.

## Raised caps

```css
p::first-letter {
  initial-letter: 2 1;
}
```

Two arguments: `initial-letter: <size> <sink>`. Size of 2 means the letter is 2 lines tall. Sink of 1 means its *baseline* sits on the *first* baseline — the letter rises above the text rather than dropping below it. This is the "raised cap" or "elevated cap" style.

```css
/* 3 lines tall, sitting on line 1 — floats above */
p::first-letter {
  initial-letter: 3 1;
  margin-bottom: 0.2em;
}
```

## Edge cases worth knowing

The spec and the browser implementations disagree on several things that matter in practice:

- **Punctuation preceding the letter** (e.g., `"The dog…"`): the opening quote mark *should* be included in the initial letter per spec. Browser behavior varies. If your content uses opening quotation marks, test this.
- **Inline elements at the start of a paragraph** break `::first-letter` selection in some browsers. If you're wrapping the opening in a `<span>` for any reason, `initial-letter` may not apply.
- **RTL text:** the spec accounts for it; browser support is uneven as of early 2024.

## The question behind the question

The reason I spent two hours on `initial-letter` when I have longer pieces in the queue is this: the web has been text-first since Berners-Lee typed the first `<h1>`, and our tooling for handling text seriously has lagged embarrassingly behind what print typography has known for five hundred years.

Drop caps are not decoration. They are a *signal* — to the reader, that what follows was composed with care; to the editor, that the writing is worth the treatment. The hack-based approximations we've been using since the early 2000s have never quite conveyed that signal cleanly.

Two lines of CSS, baseline-aligned, cross-browser. The feature took longer than it should have. Use it anyway.
