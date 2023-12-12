---
title: "@scope: component CSS without the specificity bloodsport"
date: "2023-12-12"
category: CSS
description: BEM existed because CSS had no way to limit a selector's reach. CSS @scope, shipping in Chrome 118, removes that excuse.
tags:
  - css
  - architecture
  - components
  - specificity
draft: false
---

BEM was a treaty. Not an elegant one — nobody looks at `card__header--modifier-active` and feels good about their life — but a necessary one. When CSS selectors have infinite reach, you write conventions to pretend they don't.

`@scope` arrived in Chrome 118. It gives selectors an actual boundary.

```css
@scope (.card) {
  :scope {
    padding: 1.5rem;
    border-radius: 6px;
    background: #fff;
  }

  .title {
    font-size: 1.25rem;
    font-weight: 600;
  }

  .body {
    color: #555;
  }
}
```

`.title` and `.body` inside that block only match elements that are *descendants of* `.card`. No specificity games. No `__` notation. No cascade anxiety.

## The donut scope

This is the part that doesn't get enough attention. `@scope` accepts two arguments: a **scope root** and a **scope limit**.

```css
@scope (.card) to (.card-footer) {
  a {
    color: blue;
    text-decoration: underline;
  }
}
```

This styles links inside `.card` — but *stops* at `.card-footer`. The selector never enters the footer. You've defined a donut-shaped region of the DOM and the style only fills the hole.

This is genuinely new. Previously, the only way to exclude a subtree from a selector was to override styles inside it, which created the specificity debt you were trying to avoid in the first place.

## Against BEM as religion

BEM is fine. I'm not here to be rude about it. But it's worth being honest that BEM's complexity scales with the number of developers who need to agree on naming conventions, and those agreements are enforced by culture rather than tooling. Someone will eventually name something `.card-body` instead of `.card__body` and everything downstream will quietly misbehave.

`@scope` enforces boundaries with syntax instead of convention. The difference between a team rule and a compiler error.

A rewrite comparison: take a card component with nested elements, rewrite it from BEM to `@scope`:

**Before:**

```css
.card { padding: 1.5rem; }
.card__title { font-size: 1.25rem; }
.card__body { color: #555; }
.card__footer { border-top: 1px solid #e0e0e0; }
.card__footer .card__link { color: inherit; }
```

**After:**

```css
@scope (.card) to (.footer) {
  :scope     { padding: 1.5rem; }
  .title     { font-size: 1.25rem; }
  .body      { color: #555; }
  .footer    { border-top: 1px solid #e0e0e0; }
  .footer a  { color: inherit; }
}
```

The class names became shorter. The structural guarantee became stronger. Nobody had to agree on the `__` separator.

## Where it gets interesting: nested scope

```css
@scope (.page) {
  @scope (.card) {
    .title {
      /* only matches .title inside .card inside .page */
    }
  }
}
```

Nested `@scope` creates predictably narrow selectors. Deeply nested component trees — think design systems building on design systems — finally have a structural story that doesn't collapse under its own weight.

## The tradeoffs

The honest ones:

- **Proximity wins over specificity.** Inside nested `@scope` blocks, the closest scope root wins. This is intentional but initially counterintuitive if you're used to specificity as the only cascade tie-breaker.
- **No Firefox yet** (as of this writing, December 2023). It's in progress and behind a flag.
- **Shadow DOM still has advantages** for third-party components where you genuinely don't want any outside styles to leak in. `@scope` limits your own selectors; it doesn't block external ones.

`@scope` doesn't kill BEM. It just makes BEM optional. That's enough.
