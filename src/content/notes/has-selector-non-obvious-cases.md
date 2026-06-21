---
title: ":has() and the cases nobody's writing about"
date: "2024-02-07"
category: CSS
description: "Everyone covered :has() as \"the CSS parent selector.\" That framing undersells it by about 80%."
tags:
  - css
  - selectors
  - no-js
  - has
draft: false
---

When `:has()` became baseline in December 2023, every article led with: *"CSS finally gets a parent selector."*

Which is true, and the least interesting thing about it.

Yes, you can style a parent based on its children. That's useful. But `:has()` is a relational selector — it queries DOM relationships in any direction, not just upward. The "parent selector" framing is the equivalent of introducing the iPhone as "a phone that also plays music."

Here are the cases that actually change how I write interfaces.

> **Browser support:** Baseline December 2023. Chrome 105+, Firefox 121+, Safari 15.4+. Ship it.

## Style a form that contains an invalid input

```css
form:has(input:invalid) .form-submit {
  opacity: 0.5;
  cursor: not-allowed;
  pointer-events: none;
}

form:has(input:invalid) .error-banner {
  display: block;
}
```

No JavaScript. No onChange handlers. No validity checking on submit. The browser already tracks validity state — `:has()` lets you query it from a parent.

## Cards that know what they contain

```css
/* A card with an image gets more padding on the text */
.card:has(img) .card-body {
  padding-top: 0.75rem;
}

/* A card without an image gets a decorative border */
.card:not(:has(img)) {
  border-left: 3px solid var(--accent);
}

/* A featured card gets a different background entirely */
.card:has(.badge--featured) {
  background: var(--surface-elevated);
  box-shadow: 0 4px 20px rgba(0,0,0,0.08);
}
```

Context-aware component styling that previously required conditional class application from JavaScript. The markup drives the style based on what it actually contains.

## A tab system — with caveats

Pure HTML and CSS tabs:

```html
<div class="tabs">
  <input type="radio" name="tabs" id="tab-1" checked hidden>
  <input type="radio" name="tabs" id="tab-2" hidden>
  <input type="radio" name="tabs" id="tab-3" hidden>

  <nav class="tab-nav">
    <label for="tab-1">Overview</label>
    <label for="tab-2">Details</label>
    <label for="tab-3">Reviews</label>
  </nav>

  <div class="tab-panel" id="panel-1">Overview content.</div>
  <div class="tab-panel" id="panel-2">Details content.</div>
  <div class="tab-panel" id="panel-3">Reviews content.</div>
</div>
```

```css
.tab-panel { display: none; }

.tabs:has(#tab-1:checked) #panel-1,
.tabs:has(#tab-2:checked) #panel-2,
.tabs:has(#tab-3:checked) #panel-3 {
  display: block;
}

.tabs:has(#tab-1:checked) label[for="tab-1"],
.tabs:has(#tab-2:checked) label[for="tab-2"],
.tabs:has(#tab-3:checked) label[for="tab-3"] {
  font-weight: 700;
  border-bottom: 2px solid var(--accent);
}
```

The `<input type="radio">` elements are hidden. Their checked state drives which panel is visible and which label is active. State lives in the DOM, not in JavaScript.

> **Accessibility warning:** This pattern looks like tabs but isn't semantically tabs. Screen readers encounter hidden radio inputs — not `role="tab"` and `role="tabpanel"`. For a production tab component with proper ARIA semantics and arrow key navigation, you still need JavaScript. Use this for visual prototyping, not for user-facing UI where keyboard and screen reader support matter.

## Quantity queries

```css
/* Style a list differently when it has more than 5 items */
.list:has(li:nth-child(6)) {
  columns: 2;
  column-gap: 2rem;
}

/* If there's only one item, don't show the bullet */
.list:not(:has(li:nth-child(2))) li {
  list-style: none;
}
```

For years, quantity queries required JavaScript to count children and apply a class. `:has()` combined with `:nth-child()` is purely declarative.

## The honest reckoning

I use these patterns, get excited, write about them, and the caveats end up in footnotes. Putting them here instead:

- **Performance.** `:has()` can be expensive in large DOMs. A selector that triggers paint invalidation on a parent on every keystroke inside a child will hurt. Use it for structural styling decisions, not real-time interactive effects.
- **No `:has()` inside `:has()`.** You can't nest them — `:has(:has(.foo))` is invalid per the spec.
- **Specificity matches the argument.** `:has(.foo)` has the specificity of `.foo`. Usually fine, but worth knowing.

`:has()` changes the basic terms of how CSS and JavaScript divide the work of state-driven UI. Most of us are still writing JavaScript for things CSS now handles. That's not a moral failing. It's just a lag. The tooling catches up last.
