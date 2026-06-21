---
title: "calc-size(): the fix for the hack we've been shipping for a decade"
date: "2024-09-05"
category: CSS
description: "Animating height to auto has been impossible since CSS animations were invented. Chrome 129 ships calc-size(), which makes it possible. Here's the two-line fix and the surrounding context."
tags:
  - css
  - animation
  - accordion
  - layout
draft: false
---

It's 2016. You're building an accordion. Product wants "smooth expand and collapse." You write `transition: height 300ms ease`, set `height: auto` on the expanded state, and the universe informs you that CSS cannot interpolate between a number and a keyword.

So you do what we all did. The measurement dance:

```javascript
// The accordion hack, preserved in amber
function expand(panel) {
  const fullHeight = panel.scrollHeight; // read the natural height
  panel.style.height = '0';
  requestAnimationFrame(() => {
    panel.style.height = `${fullHeight}px`;
    panel.addEventListener('transitionend', () => {
      panel.style.height = 'auto'; // set to auto after animation so it's responsive
    }, { once: true });
  });
}

function collapse(panel) {
  panel.style.height = `${panel.scrollHeight}px`;
  requestAnimationFrame(() => {
    panel.style.height = '0';
  });
}
```

I have written this, roughly, forty times. Copy-pasted into projects. Found it in design system codebases. Watched junior developers discover it and get the specific expression of someone learning that what looked native was actually a workaround the whole time.

Chrome 129, August 2024. `calc-size()`:

```css
.accordion-panel {
  height: 0;
  overflow: hidden;
  transition: height 300ms cubic-bezier(0.22, 0.61, 0.36, 1);
}

.accordion-panel.is-open {
  height: calc-size(auto, size);
}
```

`calc-size(auto, size)` resolves to the element's natural `auto` height as an interpolatable value. The browser handles the measurement. No JavaScript. No `scrollHeight`. No rAF ceremony.

> **Browser support:** Chrome 129+ (August 2024). Firefox and Safari: check [MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/calc-size) — neither has shipped this as of late 2024. Use `@supports (height: calc-size(auto, size))` for progressive enhancement.

## What `size` means

Inside `calc-size()`, `size` represents the resolved value of the keyword passed as the first argument:

```css
/* Equivalent to the element's min-content height */
height: calc-size(min-content, size);

/* The element's fit-content height */
height: calc-size(fit-content, size);

/* 80% of the element's auto height */
height: calc-size(auto, size * 0.8);

/* The element's auto height plus 2rem of extra space */
height: calc-size(auto, size + 2rem);
```

You can do arithmetic with it. That's the `calc` in `calc-size`.

## A complete CSS accordion

```html
<div class="accordion">
  <button class="accordion-trigger" aria-expanded="false" aria-controls="panel-1">
    Section one
  </button>
  <div class="accordion-panel" id="panel-1" role="region">
    <div class="accordion-panel-inner">
      Content here.
    </div>
  </div>
</div>
```

```css
.accordion-panel {
  height: 0;
  overflow: hidden;
  transition: height 280ms cubic-bezier(0.22, 0.61, 0.36, 1);
}

.accordion-panel.is-open {
  height: calc-size(auto, size);
}

@media (prefers-reduced-motion: reduce) {
  .accordion-panel { transition: none; }
}
```

```javascript
// The only JavaScript you still write
document.querySelectorAll('.accordion-trigger').forEach(trigger => {
  trigger.addEventListener('click', () => {
    const panel = document.getElementById(trigger.getAttribute('aria-controls'));
    const isOpen = trigger.getAttribute('aria-expanded') === 'true';

    trigger.setAttribute('aria-expanded', String(!isOpen));
    panel.classList.toggle('is-open', !isOpen);
  });
});
```

Twelve lines. Every line is business logic: ARIA state, class toggle. No measurement, no geometry, no frame timing. The CSS does the interpolation.

## The companion: `interpolate-size`

Chrome 129 also shipped `interpolate-size: allow-keywords`:

```css
:root {
  interpolate-size: allow-keywords;
}

.accordion-panel {
  height: 0;
  transition: height 300ms ease;
}

.accordion-panel.is-open {
  height: auto;
}
```

With `interpolate-size: allow-keywords` on an ancestor, `auto`, `min-content`, and `fit-content` become interpolatable directly. No `calc-size()` wrapper.

`calc-size()` is explicit: you opt in per property. `interpolate-size` is implicit: it enables keyword interpolation for everything in scope. Both work. `calc-size()` is surgical; `interpolate-size` is convenient.

> See also: the dedicated note on [`interpolate-size`](/notes/interpolate-size-keyword-animation) for a deeper treatment of the keyword interpolation approach.

## The decade we spent on this

The accordion problem is legitimate. It affects nearly every developer who has ever built a UI component. The `scrollHeight` hack is correct and reliable. It's not broken. It just shouldn't have existed.

The platform had the natural height, the transition mechanism, and the intent. `calc-size()` is the bridge that finally connects them — and it's worth understanding why it works, because the `size` keyword is more flexible than the accordion demo suggests.

Play with the arithmetic forms. `calc-size(auto, size * 0.8)` for partial reveals. `calc-size(auto, size + 2rem)` for breathing room. The keyword is a variable, not a fixed value.

Now delete the measurement code.
