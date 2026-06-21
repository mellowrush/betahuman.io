---
title: The Popover API walks in and the JS bundle gets smaller
date: "2024-05-09"
category: Browser APIs
description: "The Popover API became baseline in April 2024. Native tooltips, modals, dropdowns — with focus management, light-dismiss, and ::backdrop. No JavaScript required for the basic case."
tags:
  - html
  - browser-api
  - components
  - no-js
  - accessibility
draft: false
---

Every ten years or so, the web platform quietly ships something that makes you look at three years of component library work and feel like you've been very busy doing something that turns out to have a native version.

The Popover API became baseline in April 2024. The `[popover]` attribute gives you a floating layer with focus trapping, light-dismiss (click outside to close), top-layer rendering (above everything, including fixed elements and `z-index` stacks), and a `::backdrop` pseudo-element. The browser handles the plumbing.

## The minimum viable tooltip

```html
<button popovertarget="my-tooltip" popovertargetaction="toggle">
  What does this button do?
</button>

<div id="my-tooltip" popover>
  It does the thing. You know the thing.
</div>
```

Click the button. The popover appears. Click outside. The popover disappears. No `document.addEventListener`. No click-outside detection. No `z-index: 9999`. No JavaScript.

## A modal dialog without the boilerplate

```html
<button popovertarget="confirm-dialog">Delete account</button>

<div id="confirm-dialog" popover="manual">
  <h2>Are you sure?</h2>
  <p>This cannot be undone. Your data will be gone. Your settings, gone. Your carefully curated playlists — also gone.</p>
  <button popovertarget="confirm-dialog" popovertargetaction="hide">Cancel</button>
  <button id="confirm-delete">Yes, delete everything</button>
</div>
```

`popover="manual"` disables light-dismiss — the modal only closes when explicitly dismissed. Focus is trapped inside. The `::backdrop` is rendered automatically.

```css
[popover] {
  border: none;
  border-radius: 8px;
  padding: 2rem;
  max-width: 420px;
  box-shadow: 0 20px 60px rgba(0,0,0,0.2);
}

[popover]::backdrop {
  background: rgba(0, 0, 0, 0.4);
  backdrop-filter: blur(2px);
}
```

## The `@starting-style` animation integration

Pop it open with an animation. Close it with one. This requires `@starting-style` (covered separately) and `transition-behavior: allow-discrete`:

```css
[popover] {
  opacity: 1;
  transform: translateY(0) scale(1);
  transition:
    opacity 200ms ease,
    transform 200ms ease,
    overlay 200ms allow-discrete,
    display 200ms allow-discrete;
}

[popover]:not(:popover-open) {
  opacity: 0;
  transform: translateY(-8px) scale(0.97);
}

@starting-style {
  [popover]:popover-open {
    opacity: 0;
    transform: translateY(-8px) scale(0.97);
  }
}
```

The `:popover-open` pseudo-class selects the popover when it's in its open state. `@starting-style` defines where the enter transition begins. The result: smooth fade-scale in, fade-scale out, with the element actually leaving the DOM (not just hiding) at the end of the exit transition.

## Where it falls short

Honest inventory:

- **Positioning**: The Popover API doesn't handle positioning relative to the trigger. The popover appears centered in the viewport by default. For tooltips that need to appear *next to* their button, you need CSS Anchor Positioning (covered separately) or a small bit of JavaScript to set `style.top/left`.
- **Nested popovers**: Multiple popovers can be stacked, but the auto-dismiss behavior gets complex with nested hierarchies. Test it.
- **Animations on close**: Requires `allow-discrete` and `@starting-style` to work correctly. Without those, the element disappears instantly.
- **Accessibility role**: The `[popover]` attribute doesn't imply an ARIA role. For a dialog, add `role="dialog"` and `aria-modal="true"`. For a menu, add `role="menu"`.

## The JS you still write

```javascript
// That's it. This is the JavaScript you write for a tooltip.
document.getElementById('confirm-delete').addEventListener('click', async () => {
  await deleteAccount();
  document.getElementById('confirm-dialog').hidePopover();
});
```

The plumbing is handled. What's left is the actual business logic — which is what the JavaScript should have been doing in the first place instead of managing focus traps.

The component library version of a modal is roughly 200–400 lines of JavaScript: focus trap, click-outside handler, scroll lock, keyboard handling, z-index management, portal rendering, animation state. The HTML-native version is three lines of markup, two CSS blocks, and one line of JavaScript when you need to respond to the user's actual decision.

That ratio has changed.
