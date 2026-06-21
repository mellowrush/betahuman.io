---
title: "Anchor positioning: or how we stopped computing coordinates"
date: "2024-06-06"
category: CSS
description: "CSS Anchor Positioning shipped in Chrome 125. Float an element relative to any other element in the DOM — through scroll, resize, overflow — without a single line of JavaScript."
tags:
  - css
  - layout
  - tooltip
  - positioning
  - no-js
draft: false
---

We've been computing tooltip positions with JavaScript for twenty-five years. `getBoundingClientRect()`. Scroll offsets. Viewport clamping. Resize listeners. Popper.js, Floating UI, Tippy — whichever library the team agreed to maintain this year.

None of this was necessary. It was just the only option available.

CSS Anchor Positioning shipped in Chrome 125. Any element, positioned relative to any other element in the DOM — through scroll, through `overflow: hidden`, through resize — in pure CSS.

> **Browser support:** Chrome 125+. Firefox and Safari: check [MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/anchor-name) — implementation status moves faster than this article. Use `@supports` in the meantime.

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

`anchor-name` registers the anchor. `position-anchor` connects to it. The `anchor()` function resolves to the anchor's geometric edges — `top`, `bottom`, `left`, `right`, `center` — at paint time.

No scroll offset calculation. No `getBoundingClientRect`. No resize listener. The browser tracks the geometry.

> **To test:** Paste this into a local HTML file and open in Chrome 125+. The tooltip positions relative to the button with no JS at all.

## Automatic position fallback

The part that actually ships to production:

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

The browser tries each `@position-try` block in order. Overflows the viewport? Next option. Automatic viewport-aware reflow. The most annoyingly difficult thing to implement in JavaScript. Now a CSS declaration.

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

The popover handles show/hide, light-dismiss, focus management, top-layer rendering. Anchor positioning handles where. Together: a full tooltip pattern, zero JavaScript.

## The library inventory

Popper.js: 19KB gzipped. Floating UI: 12KB gzipped. Tippy.js (built on Popper): 30KB+.

The CSS-native version: approximately 20 lines.

The library wasn't solving a conceptual problem. It was covering the gap between what CSS offered and what the requirements were. That gap is gone.

Use `@supports` until Firefox and Safari land it:

```css
@supports (anchor-name: --test) {
  /* anchor positioning styles */
}
```

The question isn't whether to learn this. The question is how quickly the team will accept "we can delete Floating UI now" as a pull request title.
