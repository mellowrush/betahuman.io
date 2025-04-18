---
title: "@function in CSS: what design tokens want to be"
date: "2025-04-18"
category: CSS
description: The CSS @function proposal is Stage 1. If it ships, it rewrites how design systems use custom properties. Here's what the proposal looks like and why it matters even before it lands.
tags:
  - css
  - design-tokens
  - proposal
  - future
draft: false
---

Every design system eventually arrives at the same problem. Custom properties store values. But values aren't relationships. And relationships are what design systems are actually selling.

`--space-4: 16px` is a value. "Four steps up the spacing scale" is a relationship. The value is brittle — it breaks if the base unit changes. The relationship is robust — it recalculates correctly regardless of what the base unit becomes.

Sass has `@mixin` and `@function` for this reason. They let you define relationships, not just values. The CSS `@function` proposal, currently Stage 1 in the CSS Working Group, brings this capability to native CSS.

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

`--space(4)` is not a variable lookup. It's a function call that executes the calculation. Change the base unit (the `0.25rem` multiplier), and every call site updates automatically.

## What this enables that custom properties can't

Custom properties let you define values and reference them. But you can't parameterize a custom property:

```css
/* This is valid — but the value is fixed */
:root { --space-4: 1rem; }

/* This is what you want but can't have today */
/* --space(4) */  ← not valid CSS property syntax
```

`@function` makes parameterized values native. Examples from the proposal:

```css
@function --color-tint(--base, --amount) {
  result: oklch(from var(--base) calc(l + var(--amount)) c h);
}

.button {
  background: --color-tint(var(--rust), 10%);    /* 10% lighter */
  border-color: --color-tint(var(--rust), -10%); /* 10% darker */
}
```

One function definition. Any call site can pass different arguments. The relationship (tint by N%) is defined once and reused everywhere.

```css
@function --clamp-type(--min, --max) {
  result: clamp(var(--min), 4vw, var(--max));
}

h1 { font-size: --clamp-type(1.8rem, 3rem); }
h2 { font-size: --clamp-type(1.4rem, 2.2rem); }
h3 { font-size: --clamp-type(1.1rem, 1.6rem); }
```

A fluid type scale defined once, applied with different min/max values per heading level.

## The Sass comparison

Sass functions do this today:

```scss
@function space($steps) {
  @return $steps * 0.25rem;
}
```

The difference: Sass runs at build time. The output is static CSS. If you need the function to respond to runtime values — user settings, `@media` queries, custom property overrides — Sass can't do it. It's already compiled.

CSS `@function` runs in the browser. It can reference other custom properties, respond to cascade changes, and recalculate when the underlying values change.

```css
/* This is the build-time version (Sass) */
/* padding: space(4); → padding: 1rem; → static */

/* This is the runtime version (CSS @function) */
/* padding: --space(4); → calc(4 * var(--base-unit)); → recalculates when --base-unit changes */
```

If a user increases their browser font size, `--base-unit` (if defined in terms of `rem`) responds. The function recalculates. The spacing adjusts. The Sass version doesn't.

## The current state (April 2025)

Stage 1. No browser implementation. The proposal is being actively discussed in the CSSWG. The syntax above is based on the current draft and may change significantly before any implementation ships.

The reason to pay attention now: Stage 1 proposals in CSS often move faster than you'd expect once the Working Group reaches rough consensus. `@property` went from proposal to Chrome implementation in under a year. Anchor Positioning went from concept to Chrome 125 with a fairly compressed timeline relative to its complexity.

The design system implications are significant enough that it's worth watching the proposal closely and experimenting with polyfills as they become available.

The problem it solves is real. The Sass function exists because CSS couldn't. If CSS can — if parameterized, runtime-evaluated, cascade-aware functions become native — the compile step disappears and the relationship between design intent and rendered output gets one level shorter.

Shorter paths from intent to output. That's the whole project, really.
