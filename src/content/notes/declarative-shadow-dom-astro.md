---
title: Web components that don't need a client to wake up
date: "2024-04-03"
category: Browser APIs
description: "Declarative Shadow DOM streams shadow roots from the server. No JavaScript to hydrate. Here's what a zero-client-JS web component looks like in an Astro project."
tags:
  - web-components
  - astro
  - ssr
  - shadow-dom
draft: false
---

Web components have always been a runtime concern. Define the element, register it, wait for JavaScript to run. No JavaScript, no component. That's not a design philosophy — that's a limitation masquerading as one.

Declarative Shadow DOM, shipped in Chrome 111, Firefox 123 and Safari 16.4, changes the fundamental contract. You can define a shadow root in HTML — no JavaScript — and the browser attaches it during parsing, before any script runs.

```html
<my-card>
  <template shadowrootmode="open">
    <style>
      :host {
        display: block;
        padding: 1.5rem;
        border-radius: 8px;
        border: 1px solid #e0e0e0;
      }
      .title { font-size: 1.25rem; font-weight: 600; }
      .body  { color: #555; margin-top: 0.5rem; }
    </style>
    <slot name="title">
      <span class="title">Default title</span>
    </slot>
    <div class="body">
      <slot></slot>
    </div>
  </template>

  <span slot="title">The actual title</span>
  <p>The actual body content goes here.</p>
</my-card>
```

This HTML, served from the server, renders a styled, encapsulated component with zero JavaScript. Shadow root attached during parsing. Style encapsulation works. Slot projection works. No custom element registry required for initial render.

> **Browser support:** Chrome 111+, Firefox 123+, Safari 16.4+. Baseline 2024. Check [MDN](https://developer.mozilla.org/en-US/docs/Web/API/Web_components/Using_shadow_DOM#declarative_shadow_dom) for the full table.

## In an Astro component

Astro's island architecture is built around a simple premise: most components should render on the server with zero client JavaScript and only hydrate where interactivity is actually needed. Declarative Shadow DOM fits this exactly.

```astro
---
interface Props {
  title: string;
  category?: string;
}
const { title, category } = Astro.props;
---

<bh-card>
  <template shadowrootmode="open">
    <style>
      :host {
        display: block;
        font-family: system-ui, sans-serif;
        border-radius: 6px;
        overflow: hidden;
        border: 1px solid oklch(90% 0.02 80);
      }

      .header {
        padding: 1rem 1.25rem 0.5rem;
        border-bottom: 1px solid oklch(92% 0.02 80);
      }

      .title {
        font-size: 1.1rem;
        font-weight: 600;
        margin: 0;
      }

      .category {
        font-size: 0.75rem;
        text-transform: uppercase;
        letter-spacing: 0.08em;
        color: oklch(55% 0.02 80);
        margin-top: 0.25rem;
      }

      .content {
        padding: 1.25rem;
      }
    </style>

    <div class="header">
      <p class="title">{title}</p>
      {category && <p class="category">{category}</p>}
    </div>
    <div class="content">
      <slot></slot>
    </div>
  </template>

  <slot />
</bh-card>
```

No `client:load`. No `client:idle`. Zero JavaScript shipped for this component. Astro processes `{title}` and `{category}` at build time, so the `<template>` contains fully-rendered HTML by the time it hits the browser. Styles are encapsulated inside the shadow root.

## The enhancement path

The component above does zero JavaScript now. If you need interactivity later — say, the card should toggle an expanded state — you add a custom element definition progressively:

```javascript
// card.js — loaded with client:idle or client:load
customElements.define('bh-card', class extends HTMLElement {
  constructor() {
    super();
    // Shadow root already exists from DSD. Don't create another.
    const shadow = this.shadowRoot;
    shadow.querySelector('.header').addEventListener('click', () => {
      this.toggleAttribute('expanded');
    });
  }
});
```

The `shadowRoot` is already attached. The custom element definition enhances it without replacing it. No flash of unstyled content. No hydration gap.

## What this makes possible

DSD doesn't solve everything about web components. Custom element registration, observed attributes and property reflection still require JavaScript. Complex interactive widgets still need a script.

But for structural, style-encapsulated layout components — cards, banners, section wrappers, media objects — you can now ship them as SSR'd HTML with style isolation, zero runtime cost and a clear path to enhancement if interactivity becomes necessary later.

The criticism that web components require JavaScript was one of the fundamental objections to using them on content sites. DSD removes that objection for a meaningful slice of use cases. Not all of them. Enough of them.
