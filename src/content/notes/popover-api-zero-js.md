---
title: The Popover API walks in and the JS bundle gets smaller
date: "2024-05-09"
category: Browser APIs
description: "The Popover API became baseline in April 2024. Native tooltips, modals, dropdowns — with focus management, light-dismiss and ::backdrop. No JavaScript required for the basic case."
tags:
  - html
  - browser-api
  - components
  - no-js
  - accessibility
draft: false
---

The Popover API became Baseline in April 2024. The `[popover]` attribute gives you a floating layer with light-dismiss (click outside to close), top-layer rendering (above everything, including fixed elements and `z-index` stacks) and a `::backdrop` pseudo-element. The browser handles the plumbing.

> **Browser support:** Baseline April 2024. Chrome 114+, Firefox 125+, Safari 17+. [MDN reference](https://developer.mozilla.org/en-US/docs/Web/API/Popover_API).

## The minimum viable tooltip

```html
<button popovertarget="my-tooltip" popovertargetaction="toggle">
  What does this button do?
</button>

<div id="my-tooltip" popover>
  It does the thing. You know the thing.
</div>
```

<div class="pop-demo" role="region" aria-label="Live demo: Popover API">
  <style>
    .pop-demo {
      border: 1px solid var(--bh-border-strong, #d4c8c0);
      border-radius: 8px;
      padding: 1rem 1.25rem;
      margin: 1.75rem 0;
      background: var(--bh-bg, #faf7f4);
    }
    .pop-demo-label {
      font-family: monospace;
      font-size: 0.7rem;
      letter-spacing: 0.08em;
      text-transform: uppercase;
      color: var(--bh-text-faint, #9a8f87);
      margin-bottom: 0.875rem;
      display: block;
    }
    .pop-trigger {
      background: var(--bh-bg-muted, #f0ebe6);
      border: 1px solid var(--bh-border, #e0d9d4);
      border-radius: 4px;
      padding: 0.5rem 0.875rem;
      font-size: 0.875rem;
      cursor: pointer;
      font-family: inherit;
      color: var(--bh-text, #2a1f1b);
    }
    #pop-demo-tip {
      border: 1px solid var(--bh-border-strong, #d4c8c0);
      border-radius: 6px;
      padding: 0.75rem 1rem;
      background: var(--bh-bg, #faf7f4);
      font-size: 0.875rem;
      color: var(--bh-text, #2a1f1b);
      box-shadow: 0 4px 16px rgba(0,0,0,0.12);
      max-width: 240px;
    }
  </style>
  <span class="pop-demo-label">↓ Click — light-dismiss and top-layer rendering, no JS</span>
  <button class="pop-trigger" popovertarget="pop-demo-tip" popovertargetaction="toggle">
    Open tooltip
  </button>
  <div id="pop-demo-tip" popover>
    It does the thing. No <code>z-index: 9999</code>. No click-outside handler. No JavaScript.
  </div>
</div>

Click the button. The popover appears. Click outside. It disappears. No `document.addEventListener`. No click-outside detection. No `z-index: 9999`. No JavaScript.

## A modal dialog without the boilerplate

```html
<button popovertarget="confirm-dialog">Delete account</button>

<div id="confirm-dialog" popover="manual" role="dialog" aria-modal="true" aria-labelledby="confirm-heading">
  <h2 id="confirm-heading">Are you sure?</h2>
  <p>This cannot be undone. Your data will be gone. Your settings, gone. Your carefully curated playlists — also gone.</p>
  <button popovertarget="confirm-dialog" popovertargetaction="hide">Cancel</button>
  <button id="confirm-delete">Yes, delete everything</button>
</div>
```

`popover="manual"` disables light-dismiss — the modal only closes when explicitly dismissed.

> **Accessibility note:** `[popover]` does NOT trap focus the way `<dialog>` does. For a true modal that prevents keyboard escape, use `<dialog>` with `.showModal()`. For non-modal popovers (tooltips, dropdowns, menus), the popover API is appropriate. Always add `role="dialog"` + `aria-modal="true"` for popover-based dialogs and ensure keyboard focus is managed manually if you need full trapping.

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

The `:popover-open` pseudo-class selects the popover in its open state. `@starting-style` defines where the enter transition begins. Result: smooth fade-scale in, fade-scale out, with the element actually leaving the DOM at the end of the exit transition, not just hiding.

## Where it falls short

- **Positioning.** The popover appears centered in the viewport by default. For tooltips that need to appear *next to* their trigger, you need CSS Anchor Positioning or JavaScript to set `style.top/left`.
- **Focus trapping.** `[popover]` doesn't trap focus. Use `<dialog>` for that.
- **Nested popovers.** Multiple popovers can stack, but auto-dismiss behavior gets complex with nested hierarchies. Test it.
- **Accessibility role.** `[popover]` doesn't imply an ARIA role. Add `role="dialog"` for dialogs, `role="menu"` for menus.

## The JS you still write

```javascript
// That's it.
document.getElementById('confirm-delete').addEventListener('click', async () => {
  await deleteAccount();
  document.getElementById('confirm-dialog').hidePopover();
});
```

The plumbing is handled. What's left is the actual business logic — which is what the JavaScript should have been doing instead of managing focus traps.

The component library version of a modal is roughly 200–400 lines: focus trap, click-outside handler, scroll lock, keyboard handling, z-index management, portal rendering, animation state. The HTML-native version is three lines of markup, two CSS blocks and one line of JavaScript when you need to respond to the user's actual decision.

That ratio has changed.
