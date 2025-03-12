---
title: "invoketarget: when HTML finally handles its own interactions"
date: "2025-03-12"
category: Browser APIs
description: Invoker Commands are arriving in Chrome. invoketarget and invokeaction let a button target a popover or dialog without any JavaScript event listener. Here's what the pattern looks like and why it matters.
tags:
  - html
  - browser-api
  - progressive-enhancement
  - no-js
  - accessibility
draft: false
---

Let me describe the smallest JavaScript component you've ever been asked to write.

```javascript
document.getElementById('open-btn').addEventListener('click', () => {
  document.getElementById('my-dialog').showModal();
});
```

Six lines, including the closing braces. The entire purpose of this code is to wire a button to a dialog. There's no logic. No transformation. No side effect. Just "button clicked, dialog opened." The HTML knows about both elements. The relationship is self-evident. And yet: JavaScript.

Invoker Commands — currently shipping in Chrome 135 — let you express that relationship in HTML directly:

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

Which means any custom element can become an invokable target, with buttons anywhere in the DOM triggering specific named actions.

## Why this matters beyond convenience

The boring argument: less JavaScript. Fewer event listeners. Smaller bundles.

The more interesting argument: HTML declarative relationships are serializable, inspectable, and accessible by default. When a button's relationship to its target is expressed in HTML attributes, assistive technologies can understand that relationship structurally. Screen readers can announce "this button opens the dialog" based on the markup rather than inferring it from behavior.

The JavaScript wiring model means the relationship between interactive elements is implicit — buried in event listeners, closures, and runtime state. Invoker Commands make the relationship explicit and structural.

This is the same argument that justified `<details>/<summary>`, `<dialog>`, `[popover]`, and every native HTML interactive element: when the browser understands the interaction model, it can expose it to accessibility APIs, implement keyboard handling correctly, and maintain state without JavaScript needing to track it.

## The current state (March 2025)

Chrome 135, shipping now. Firefox has expressed support but hasn't landed it. Safari position: not stated publicly.

Progressive enhancement is the correct stance: write `invoketarget` attributes on buttons that also have JavaScript fallbacks. The JavaScript listener fires only if Invoker Commands aren't supported. The native handler fires if they are.

```html
<button id="open-btn" invoketarget="my-dialog" invokeaction="showModal">
  Open
</button>
```

```javascript
// Fallback for browsers without Invoker Commands support
if (!HTMLButtonElement.prototype.hasOwnProperty('invokeTargetElement')) {
  document.getElementById('open-btn').addEventListener('click', () => {
    document.getElementById('my-dialog').showModal();
  });
}
```

The six lines of JavaScript that wired a button to a dialog will eventually be unnecessary. We're not there yet. But the destination is clear.

The browser is getting better at handling its own interactions. That's the recurring theme of the last two years. The JavaScript we write is, slowly, becoming the JavaScript we actually need to write.
