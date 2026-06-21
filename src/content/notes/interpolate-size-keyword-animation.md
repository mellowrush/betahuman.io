---
title: "interpolate-size: the other way to animate to auto"
date: "2025-06-05"
category: CSS
description: "interpolate-size: allow-keywords makes CSS keyword values like auto, min-content and fit-content interpolatable. It's simpler than calc-size() and riskier. Here's when to use which."
tags:
  - css
  - animation
  - layout
  - height-auto
draft: false
---

I covered `calc-size()` [in September](/notes/calc-size-animate-height-auto). If you read that and thought "that's clever but the extra wrapper function is annoying" — this is the one you were waiting for.

`interpolate-size: allow-keywords` is an opt-in property that makes CSS keyword values like `auto`, `min-content`, `max-content` and `fit-content` directly interpolatable in transitions and animations. No wrapper function. Set it and `height: auto` becomes animatable everywhere in scope.

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

`height: auto` transitions. That's the whole feature.

> **Browser support:** Chrome 129+ (August 2024). Firefox and Safari: not yet. Check [MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/interpolate-size) for current status. Use `@supports (interpolate-size: allow-keywords)` for progressive enhancement, with `calc-size()` or the float hack as fallback.

<div class="is-demo" role="region" aria-label="Live demo: interpolate-size accordion">
  <style>
    .is-demo {
      border: 1px solid var(--bh-border-strong, #d4c8c0);
      border-radius: 8px;
      padding: 1rem 1.25rem;
      margin: 1.75rem 0;
      background: var(--bh-bg, #faf7f4);
    }
    .is-demo-label {
      font-family: monospace;
      font-size: 0.7rem;
      letter-spacing: 0.08em;
      text-transform: uppercase;
      color: var(--bh-text-faint, #9a8f87);
      margin-bottom: 0.875rem;
      display: block;
    }
    .is-accordion {
      interpolate-size: allow-keywords;
    }
    .is-trigger {
      background: var(--bh-bg-muted, #f0ebe6);
      border: 1px solid var(--bh-border, #e0d9d4);
      border-radius: 4px;
      padding: 0.625rem 0.875rem;
      font-size: 0.875rem;
      cursor: pointer;
      width: 100%;
      text-align: left;
      font-family: inherit;
      color: var(--bh-text, #2a1f1b);
    }
    .is-panel {
      height: 0;
      overflow: hidden;
      transition: height 300ms ease;
    }
    .is-panel.open {
      height: auto;
    }
    .is-panel-inner {
      padding: 0.75rem 0.875rem;
      font-size: 0.875rem;
      color: var(--bh-text-muted, #6b5e57);
      line-height: 1.6;
    }
    @media (prefers-reduced-motion: reduce) {
      .is-panel { transition: none; }
    }
  </style>
  <span class="is-demo-label">↓ Click to expand — Chrome 129+ only</span>
  <div class="is-accordion">
    <button class="is-trigger" onclick="this.nextElementSibling.classList.toggle('open')">
      Toggle panel ↕
    </button>
    <div class="is-panel">
      <div class="is-panel-inner">
        <code>height: auto</code> transitions with <code>interpolate-size: allow-keywords</code> set on the container. No <code>calc-size()</code> wrapper, no <code>scrollHeight</code> measurement, no <code>requestAnimationFrame</code>. The browser resolves the keyword to a number and interpolates between them directly.
      </div>
    </div>
  </div>
</div>

## The difference from `calc-size()`

Two things doing the same job raises the obvious question: why two things?

`calc-size()` is **explicit and surgical**. You opt in per property, per value. If you write `height: calc-size(auto, size)`, that specific value is interpolatable. Nothing else changes.

`interpolate-size` is **implicit and broad**. Set it on an ancestor and keyword interpolation is enabled for *all* animatable properties in scope. `width: auto`, `height: min-content`, `max-height: fit-content` — all become interpolatable.

The broader scope is the convenience. It's also the risk.

## The risk

Some CSS properties that accept keyword values are currently *not supposed to* animate between keywords. `display` is one: transitioning to/from `display: none` requires `transition-behavior: allow-discrete` because discrete property animation is explicitly opt-in. `interpolate-size` doesn't override that.

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

## Side-by-side: which approach to use

```css
/* Option A: calc-size() */
.panel.is-open {
  height: calc-size(auto, size);
}
/* Pros: explicit, surgical, no implicit scope effects */
/* Cons: verbose, requires wrapper on every property */

/* Option B: interpolate-size */
.panel-container {
  interpolate-size: allow-keywords;
}
.panel.is-open {
  height: auto; /* inherits interpolation from container */
}
/* Pros: cleaner syntax, keywords just work */
/* Cons: implicit scope, potential unexpected effects */
```

My current preference: `calc-size()` for components in a design system — the explicitness avoids surprises when the component is dropped into an unknown context. `interpolate-size` for page-level layouts where you own the full scope and aren't shipping to a consumer.

## The larger pattern

Both approaches exist because CSS keyword sizing — `auto`, `min-content`, `fit-content` — was never part of the original animation model. Keyword values don't have numeric representations the browser can interpolate between. Adding animation support required either:

a) A wrapper function that resolves the keyword to a number first (`calc-size`), or  
b) An opt-in signal that tells the browser to try to resolve and interpolate these (`interpolate-size`)

The CSS Working Group shipped both. They serve slightly different authoring contexts. That's either thoughtful or indecisive, depending on how you feel about options.

I've made my peace with options. The accordion hack is dead either way.
