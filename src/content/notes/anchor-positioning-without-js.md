---
title: "Anchor positioning: or how we stopped computing coordinates"
date: "2024-06-06"
category: CSS
description: CSS Anchor Positioning shipped in Chrome 125. Float an element relative to any other element in the DOM — through scroll, resize, overflow — without a single line of JavaScript.
tags:
  - css
  - layout
  - tooltip
  - positioning
  - no-js
draft: false
---

There is a certain kind of problem that people normalize so thoroughly they forget it was ever a problem. They just accept it as part of the cost of doing business.

We've been computing tooltip positions with JavaScript for twenty-five years. Reading `getBoundingClientRect()`. Accounting for scroll offset. Clamping to viewport edges. Recalculating on resize. Reaching for Popper.js, Floating UI, Tippy, or whichever library promised to solve it this year. Maintaining those dependencies. Updating them when browsers changed their scrolling behavior.

None of this was necessary. It was just the only option available.

CSS Anchor Positioning shipped in Chrome 125. It allows any element to be positioned relative to any other element in the DOM — through scroll, through overflow: hidden, through resize — using pure CSS.

## The minimum example

```html
<button class="trigger" id="my-anchor">See details</button>
<div class="tooltip" popover>More information here.</div>
```

```css
.trigger {
  anchor-name: --my-anchor;
}

.tooltip {
  position: absolute;
  position-anchor: --my-anchor;

  /* Position the tooltip above the anchor */
  bottom: calc(anchor(top) + 8px);
  left: anchor(center);
  translate: -50% 0;
}
```

`anchor-name` registers an element as a named anchor. `position-anchor` connects a positioned element to that anchor. The `anchor()` function then resolves to the anchor's geometric edges — `top`, `bottom`, `left`, `right`, `center` — at the moment the tooltip needs to paint.

No scroll offset calculation. No `getBoundingClientRect`. No resize listener. The browser tracks the geometry.

## Automatic position fallback

The feature that makes this genuinely useful beyond the demo:

```css
.tooltip {
  position: absolute;
  position-anchor: --my-anchor;

  position-try-fallbacks:
    --tooltip-above,
    --tooltip-below,
    --tooltip-left;
}

@position-try --tooltip-above {
  bottom: calc(anchor(top) + 8px);
  left: anchor(center);
  translate: -50% 0;
}

@position-try --tooltip-below {
  top: calc(anchor(bottom) + 8px);
  left: anchor(center);
  translate: -50% 0;
}

@position-try --tooltip-left {
  right: calc(calc(100vw - anchor(left)) + 8px);
  top: anchor(center);
  translate: 0 -50%;
}
```

The browser tries each `@position-try` block in order. If applying that block would cause the positioned element to overflow the viewport, it moves to the next option. Automatic reflow to stay in view — the single most annoying thing to implement in JavaScript — is now a CSS declaration.

## Working with the Popover API

The natural pairing:

```html
<button
  class="help-trigger"
  popovertarget="help-tip"
  popovertargetaction="toggle">
  ?
</button>

<div id="help-tip" popover class="tooltip">
  This field accepts ISO 8601 date strings.
</div>
```

```css
.help-trigger {
  anchor-name: --help-anchor;
}

.tooltip[popover] {
  position-anchor: --help-anchor;
  top: calc(anchor(bottom) + 6px);
  left: anchor(center);
  translate: -50% 0;
  margin: 0;
}
```

The popover handles: show/hide, light-dismiss, focus management, top-layer rendering. Anchor positioning handles: "next to the button, not somewhere arbitrary." Together they cover the full tooltip use case with no JavaScript at all.

## The library inventory

Popper.js: 19KB gzipped. Floating UI: 12KB gzipped. Tippy.js (built on Popper): 30KB+.

The CSS-native version: approximately 20 lines.

The library wasn't solving a conceptual problem that CSS couldn't. It was solving the gap between what CSS offered and what the positioning requirements actually were. That gap is now, for the large majority of use cases, gone.

Chrome 125+. Firefox is implementing. Progressive enhancement with `@supports` until cross-browser:

```css
@supports (anchor-name: --test) {
  /* anchor positioning styles */
}
```

The question isn't whether to learn this. The question is how quickly the team will accept "we can delete Floating UI now" as a pull request title.
