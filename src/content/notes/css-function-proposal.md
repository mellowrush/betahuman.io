---
title: "@function in CSS: what design tokens want to be"
date: "2025-04-18"
category: CSS
description: "The CSS @function proposal is Stage 1. If it ships, it rewrites how design systems use custom properties. Here's what the proposal looks like and why it matters even before it lands."
tags:
  - css
  - design-tokens
  - proposal
  - future
draft: false
---

`--space-4: 16px` is a value. "Four steps up the spacing scale" is a relationship. Values are brittle — they break when the base unit changes. Relationships recalculate correctly regardless of what the base unit becomes.

Sass has `@function` for exactly this reason. The CSS `@function` proposal, Stage 1 in the CSS Working Group, brings this to native CSS — with one meaningful difference: it runs at runtime, not build time.

The proposed syntax:

```css
@function --space(--steps) {
  result: calc(var(--steps) * 0.25rem);
}

.card {
  padding: --space(4);           /* 1rem */
  margin-bottom: --space(8);     /* 2rem */
  border-radius: --space(1.5);   /* 0.375rem */
}
```

`--space(4)` is not a variable lookup. It's a function call. Change `0.25rem` to `0.3rem` and every call site updates.

> **Status (April 2025):** Stage 1. No browser implementation. Syntax is based on the current draft and may change. Check the [CSS Working Group issue](https://github.com/w3c/csswg-drafts/issues/9350) and [MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/@function) for current status.

## What this enables that custom properties can't

Custom properties store values and let you reference them. They can't be parameterized:

```css
/* Valid — but the value is fixed */
:root { --space-4: 1rem; }

/* What you want but can't have today */
/* --space(4) */  ← not valid CSS property syntax
```

`@function` makes parameterized values native. From the proposal:

```css
@function --color-tint(--base, --amount) {
  result: oklch(from var(--base) calc(l + var(--amount)) c h);
}

.button {
  background: --color-tint(var(--rust), 10%);    /* 10% lighter */
  border-color: --color-tint(var(--rust), -10%); /* 10% darker */
}
```

One definition. Any call site passes different arguments. The tint relationship is defined once and reused everywhere without duplication.

```css
@function --clamp-type(--min, --max) {
  result: clamp(var(--min), 4vw, var(--max));
}

h1 { font-size: --clamp-type(1.8rem, 3rem); }
h2 { font-size: --clamp-type(1.4rem, 2.2rem); }
h3 { font-size: --clamp-type(1.1rem, 1.6rem); }
```

A fluid type scale, defined once, with different bounds per level.

## The Sass comparison

Sass functions do this today:

```scss
@function space($steps) {
  @return $steps * 0.25rem;
}
```

The difference: Sass runs at build time. The output is static CSS. If you need the function to respond to runtime values — user settings, `@media` queries, custom property overrides — Sass can't do it. It's already compiled.

CSS `@function` runs in the browser. It can reference other custom properties, respond to cascade changes and recalculate when underlying values change.

```css
/* Build-time (Sass) */
/* padding: space(4); → padding: 1rem; → static */

/* Runtime (CSS @function) */
/* padding: --space(4); → calc(4 * var(--base-unit)); → recalculates when --base-unit changes */
```

If a user increases their browser font size, `--base-unit` (defined in `rem`) responds. The function recalculates. The spacing adjusts. The Sass version doesn't.

## The current state

Stage 1. No browser implementation as of this writing. The proposal is actively discussed in the CSSWG; the syntax above may change before anything ships.

The reason to watch it now: Stage 1 proposals in CSS move faster once the Working Group reaches rough consensus. `@property` went from proposal to Chrome implementation in under a year. Anchor Positioning had a compressed timeline relative to its complexity.

The problem this solves is real. The Sass function exists because CSS couldn't. If CSS can — parameterized, runtime-evaluated, cascade-aware functions in native CSS — the compile step disappears and the path from design intent to rendered output gets one step shorter.

Shorter paths from intent to output. That's the whole project, really.
