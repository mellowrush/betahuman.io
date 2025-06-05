---
title: "interpolate-size: the other way to animate to auto"
date: "2025-06-05"
category: CSS
description: interpolate-size: allow-keywords makes CSS keyword values like auto, min-content, and fit-content interpolatable. It's simpler than calc-size() and riskier. Here's when to use which.
tags:
  - css
  - animation
  - layout
  - height-auto
draft: false
---

I covered `calc-size()` in September. If you read that and thought "that's clever but the extra wrapper function is annoying" — this is the one you were waiting for.

`interpolate-size: allow-keywords` is an opt-in property that makes CSS keyword values like `auto`, `min-content`, `max-content`, and `fit-content` directly interpolatable in transitions and animations. No wrapper function. Set it, and `height: auto` becomes animatable everywhere in scope.

```css
:root {
  interpolate-size: allow-keywords;
}

.accordion-panel {
  height: 0;
  overflow: hidden;
  transition: height 300ms ease;
}

.accordion-panel.is-open {
  height: auto;
}
```

That's it. `height: auto` transitions. Chrome 129+, shipping now.

## The difference from `calc-size()`

Two things doing the same job raises the obvious question: why two things?

`calc-size()` is **explicit and surgical**. You opt in per property, per value. If you write `height: calc-size(auto, size)`, that specific value is interpolatable. Nothing else changes.

`interpolate-size` is **implicit and broad**. Set it on an ancestor and keyword interpolation is enabled for *all* animatable properties in scope. `width: auto`, `height: min-content`, `max-height: fit-content` — all become interpolatable.

The broader scope is the convenience. It's also the risk.

## The risk

Some CSS properties that accept keyword values are currently *not supposed to* animate between keywords. The spec for `display` is one: transitioning to/from `display: none` requires `transition-behavior: allow-discrete` because discrete property animation is explicitly opt-in. `interpolate-size` doesn't override that — `display` still needs `allow-discrete`.

But there may be other properties where implicit keyword interpolation produces unexpected behavior. Until browser implementations mature and edge cases are documented, `interpolate-size: allow-keywords` on `:root` is a broad promise. Prefer scoping it:

```css
/* Less risky: scope to the component, not the whole page */
.accordion {
  interpolate-size: allow-keywords;
}

/* More risky: enables on everything everywhere */
:root {
  interpolate-size: allow-keywords;
}
```

The scoped version gives you the feature where you want it without the implicit effects elsewhere.

## Side-by-side: which approach to use

```css
/* Option A: calc-size() */
.panel.is-open {
  height: calc-size(auto, size);
}
/* Pros: explicit, surgical, no implicit scope effects */
/* Cons: verbose, requires calc-size() wrapper on every property */

/* Option B: interpolate-size */
.panel-container {
  interpolate-size: allow-keywords;
}
.panel.is-open {
  height: auto; /* inherits interpolation from container */
}
/* Pros: cleaner syntax, auto and other keywords just work */
/* Cons: implicit scope, potential unexpected effects on other properties */
```

My current preference: `calc-size()` for components in a design system (the explicitness avoids surprises when the component is dropped into an unknown context). `interpolate-size` for page-level layouts where you own the full scope and aren't shipping to a consumer.

## The larger pattern

Both `calc-size()` and `interpolate-size` exist because CSS keyword sizing — `auto`, `min-content`, `fit-content` — was never part of the original animation model. Keyword values don't have numeric representations the browser can interpolate between. Adding animation support to them required either:

a) A wrapper function that resolves the keyword to a number first (calc-size), or  
b) An opt-in signal that tells the browser "yes, try to resolve and interpolate these" (interpolate-size)

The CSS Working Group shipped both. They serve slightly different authoring contexts. That's either thoughtful or indecisive, depending on how you feel about options.

I've made my peace with options. The accordion hack is dead either way.
