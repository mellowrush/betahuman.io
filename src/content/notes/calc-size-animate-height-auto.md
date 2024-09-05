---
title: "calc-size(): the fix for the hack we've been shipping for a decade"
date: "2024-09-05"
category: CSS
description: Animating height to auto has been impossible since CSS animations were invented. Chrome 129 ships calc-size(), which makes it possible. Here's the two-line fix and the surrounding context.
tags:
  - css
  - animation
  - accordion
  - layout
draft: false
---

Let me paint you a picture. It's 2016. You're building an accordion component. Product wants "smooth expand and collapse." You go to write `transition: height 300ms ease`, set `height: auto` on the expanded state, and then the universe breaks down and informs you that CSS cannot interpolate between a number and a keyword.

So you do what we all did. You do the measurement dance:

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

This is the canonical implementation. I have written it, roughly, forty times. I've copy-pasted it into projects. I've seen it in design system codebases. I've watched junior developers discover it and have the specific expression of someone learning that the thing they thought was native was actually a workaround the whole time.

Chrome 129, shipped in August 2024. `calc-size()`:

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

`calc-size(auto, size)` resolves to the element's natural `auto` height, made available as a value the transition engine can interpolate *to*. The browser handles the measurement. No JavaScript. No `scrollHeight`. No rAF ceremony.

## What `size` means

Inside `calc-size()`, `size` is a keyword that represents the resolved value of the keyword you passed as the first argument. So:

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

The JavaScript is twelve lines, and every line of it is business logic — managing ARIA state and toggling a class. There's no measurement, no geometry, no frame timing. The CSS does the interpolation.

## The companion: `interpolate-size`

Chrome 129 also shipped `interpolate-size: allow-keywords`, which is a different mechanism for the same problem:

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

With `interpolate-size: allow-keywords` set on an ancestor (or `:root`), keyword values like `auto`, `min-content`, and `fit-content` become interpolatable directly. No `calc-size()` wrapper needed.

The difference: `calc-size()` is explicit — you opt in per property. `interpolate-size` is implicit — it enables keyword interpolation for all properties in scope. Both work. `calc-size()` is more surgical; `interpolate-size` is more convenient.

## The decade we spent on this

I want to be clear about something. The accordion problem is a good problem. It affects nearly every web developer who has ever built a UI component. The hack — `scrollHeight` measurement in a rAF — is correct and reliable. It's not broken. It just shouldn't have existed.

The platform had the information (the element's natural height) and the mechanism (CSS transitions) and the intent (animate to auto). What it lacked for ten years was a bridge between them.

`calc-size()` is that bridge. Late, useful, and — if you let yourself think about it for a moment — mildly absurd that it took this long.

Now delete the measurement code. You earned it.
