---
title: "invoketarget: when HTML finally handles its own interactions"
date: "2025-03-12"
category: Browser APIs
description: "Invoker Commands are arriving in Chrome. invoketarget and invokeaction let a button target a popover or dialog without any JavaScript event listener. Here's what the pattern looks like and why it matters."
tags:
  - html
  - browser-api
  - progressive-enhancement
  - no-js
  - accessibility
draft: false
---

The smallest JavaScript component you've ever been asked to write:

```javascript
document.getElementById('open-btn').addEventListener('click', () => {
  document.getElementById('my-dialog').showModal();
});
```

No logic. No transformation. No side effect. Just "button clicked, dialog opened." The HTML already knows about both elements. The relationship is self-evident. And yet: JavaScript.

Invoker Commands — shipping in Chrome 135 — let you express that relationship in HTML directly:

```html
<button invoketarget="my-dialog" invokeaction="showModal">
  Open dialog
</button>

<dialog id="my-dialog">
  <h2>You didn't need JavaScript to open this.</h2>
  <button invoketarget="my-dialog" invokeaction="close">Close</button>
</dialog>
```

No JavaScript. No event listener. The button targets the dialog by ID and specifies the action. The browser makes the connection.

> **Browser support:** Chrome 135+ (April 2025). Firefox: expressed support, not yet landed. Safari: no public position. Check [MDN](https://developer.mozilla.org/en-US/docs/Web/API/Invoker_Commands_API) for current status. Use the feature detection pattern in the progressive enhancement section below.

<div class="ic-demo" role="region" aria-label="Live demo: Invoker Commands dialog">
  <style>
    .ic-demo {
      border: 1px solid var(--bh-border-strong, #d4c8c0);
      border-radius: 8px;
      padding: 1rem 1.25rem;
      margin: 1.75rem 0;
      background: var(--bh-bg, #faf7f4);
    }
    .ic-demo-label {
      font-family: monospace;
      font-size: 0.7rem;
      letter-spacing: 0.08em;
      text-transform: uppercase;
      color: var(--bh-text-faint, #9a8f87);
      margin-bottom: 0.875rem;
      display: block;
    }
    .ic-open-btn {
      background: var(--accent, #c4522a);
      color: white;
      border: none;
      border-radius: 4px;
      padding: 0.5rem 1rem;
      font-size: 0.875rem;
      cursor: pointer;
      font-family: inherit;
    }
    .ic-dialog {
      border: 1px solid var(--bh-border-strong, #d4c8c0);
      border-radius: 8px;
      padding: 1.25rem 1.5rem;
      background: var(--bh-bg, #faf7f4);
      max-width: 320px;
    }
    .ic-dialog h3 {
      margin: 0 0 0.5rem;
      font-size: 1rem;
    }
    .ic-dialog p {
      font-size: 0.875rem;
      color: var(--bh-text-muted, #6b5e57);
      margin: 0 0 1rem;
    }
    .ic-close-btn {
      background: var(--bh-bg-muted, #f0ebe6);
      border: 1px solid var(--bh-border, #e0d9d4);
      border-radius: 4px;
      padding: 0.375rem 0.75rem;
      font-size: 0.8125rem;
      cursor: pointer;
      font-family: inherit;
    }
    .ic-dialog::backdrop {
      background: rgba(0,0,0,0.4);
    }
  </style>
  <span class="ic-demo-label">↓ Chrome 135+ uses invoketarget natively; JS fallback handles others</span>

  <button
    id="ic-open"
    class="ic-open-btn"
    invoketarget="ic-dialog"
    invokeaction="showModal">
    Open dialog
  </button>

  <dialog id="ic-dialog" class="ic-dialog">
    <h3>No JS needed (in Chrome 135+)</h3>
    <p>This opened via <code>invoketarget</code> + <code>invokeaction="showModal"</code>. In other browsers, a JS fallback fired instead.</p>
    <button
      class="ic-close-btn"
      invoketarget="ic-dialog"
      invokeaction="close"
      id="ic-close">
      Close
    </button>
  </dialog>

  <script>
    // Fallback for browsers without Invoker Commands
    if (!('invokeTargetElement' in HTMLButtonElement.prototype)) {
      document.getElementById('ic-open').addEventListener('click', () => {
        document.getElementById('ic-dialog').showModal();
      });
      document.getElementById('ic-close').addEventListener('click', () => {
        document.getElementById('ic-dialog').close();
      });
    }
  </script>
</div>

## The actions

For `<dialog>`:
- `invokeaction="showModal"` — opens as a modal with backdrop
- `invokeaction="show"` — opens as non-modal
- `invokeaction="close"` — closes the dialog

For `[popover]`:
- `invokeaction="togglePopover"` — toggles open/closed (this is what `popovertarget` already does)
- `invokeaction="showPopover"` — opens
- `invokeaction="hidePopover"` — closes

Custom elements can handle invoke events too:

```javascript
myWidget.addEventListener('invoke', (event) => {
  if (event.action === 'increment') {
    this.value++;
  }
});
```

Any custom element can become an invokable target, with buttons anywhere in the DOM triggering specific named actions.

## Why this matters beyond convenience

The boring argument: less JavaScript. Fewer event listeners. Smaller bundles.

The more interesting argument: HTML declarative relationships are serializable, inspectable, and accessible by default. When a button's relationship to its target is expressed in HTML attributes, assistive technologies can understand that relationship structurally — "this button opens the dialog" — from the markup, rather than inferring it from runtime behavior.

JavaScript wiring makes the relationship implicit: buried in event listeners, closures, and runtime state. Invoker Commands make it explicit and structural.

This is the same argument that justified `<details>/<summary>`, `<dialog>`, `[popover]`, and every native HTML interactive element: when the browser understands the interaction model, it can expose it to accessibility APIs, implement keyboard handling correctly, and maintain state without JavaScript tracking it.

## Progressive enhancement

```html
<button id="open-btn" invoketarget="my-dialog" invokeaction="showModal">
  Open
</button>
```

```javascript
// Fallback for browsers without Invoker Commands support
if (!('invokeTargetElement' in HTMLButtonElement.prototype)) {
  document.getElementById('open-btn').addEventListener('click', () => {
    document.getElementById('my-dialog').showModal();
  });
}
```

The six lines of JavaScript that wired a button to a dialog will eventually be unnecessary. Not yet. But the direction is clear.

The browser is getting better at handling its own interactions. The JavaScript we write is, slowly, becoming the JavaScript we actually need to write.
