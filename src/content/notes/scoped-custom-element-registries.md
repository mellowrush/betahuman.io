---
title: "Scoped registries: web components without the naming war"
date: "2025-05-08"
category: Browser APIs
description: "Scoped custom element registries let you define custom elements scoped to a shadow root. Two components can both register x-button without conflict. This is what makes web component libraries actually composable."
tags:
  - web-components
  - browser-api
  - shadow-dom
  - future
draft: false
---

If you've ever tried to combine two web component libraries in the same project and discovered they both registered `<x-button>`, you've experienced the fundamental composability problem with the current custom element model.

The global registry is first-come, first-served. `customElements.define('x-button', ...)` succeeds exactly once. The second call for the same tag name throws. Which means: two libraries can't independently define components with the same name. They have to coordinate. In a global namespace. Across packages they don't control.

This is the naming war. It's not theoretical. It's why every web component library prefixes its elements with its brand namespace: `mwc-button`, `ui5-button`, `spectrum-button`, `fluent-button`. The prefix is a workaround for the global registry.

Scoped custom element registries, shipping in Chrome and in progress elsewhere, solve this at the spec level.

> **Browser support:** Chrome 126+ (stable, June 2024). Firefox: in progress. Safari: no public position. Check [MDN](https://developer.mozilla.org/en-US/docs/Web/API/CustomElementRegistry) for current status. Use the [WICG polyfill](https://github.com/WICG/scoped-custom-element-registries) for broader coverage, but test carefully in production.

## The model

Instead of registering to the global `customElements`, you register to a registry that's scoped to a shadow root:

```javascript
// Library A — its own isolated registry
const registryA = new CustomElementRegistry();
registryA.define('x-button', ButtonA);

// Library B — different registry, same element name, no conflict
const registryB = new CustomElementRegistry();
registryB.define('x-button', ButtonB);

// Attach each to its own shadow root
class ContainerA extends HTMLElement {
  constructor() {
    super();
    this.attachShadow({ mode: 'open', registry: registryA });
  }
}

class ContainerB extends HTMLElement {
  constructor() {
    super();
    this.attachShadow({ mode: 'open', registry: registryB });
  }
}
```

Inside `ContainerA`'s shadow tree, `<x-button>` resolves to `ButtonA`. Inside `ContainerB`'s shadow tree, it resolves to `ButtonB`. Neither knows about the other. Neither conflicts with the other.

The element name is no longer globally unique. It's locally unique — within the registry that governs the shadow root it appears in.

## What this unblocks

Design system composability. Two complete component libraries — say, your company's internal design system and a vendor's data grid library — can each include their own `<button>`, `<input>`, `<dropdown>` implementations without either renaming to avoid collision.

The consuming application picks which library goes in which shadow tree. The registries handle the rest.

```javascript
import { registry as dsRegistry } from '@company/design-system';
import { registry as gridRegistry } from '@vendor/data-grid';

class MyApp extends HTMLElement {
  constructor() {
    super();
    // The top-level app uses the company design system
    this.attachShadow({ mode: 'open', registry: dsRegistry });
  }
}

class DataPanel extends HTMLElement {
  constructor() {
    super();
    // The data panel uses the grid library's registry
    this.attachShadow({ mode: 'open', registry: gridRegistry });
  }
}
```

## The current state (May 2025)

The feature is also proposed for Declarative Shadow DOM:

```html
<!-- Proposal — not yet stable in any browser as of mid-2025 -->
<template shadowrootmode="open" shadowrootregistry="my-registry">
  <!-- Elements here resolve against my-registry -->
</template>
```

The declarative path would make server-rendered scoped registries possible — the combination that makes SSR web components with multiple libraries actually viable.

## The honest scope

This doesn't solve all web component problems. It solves the specific problem of global namespace collisions, which is real and blocks adoption in certain contexts.

It doesn't solve: slow upgrade timing, complex property reflection, the gap between DOM attributes and JavaScript properties, the ergonomic differences from framework component models, or the testing story.

But the composability problem is the one that most directly prevents web component libraries from existing alongside each other in the same application without coordination. Scoped registries are the surgical fix for that specific problem.

The naming war has been going on since custom elements shipped. The resolution is overdue. Whether "overdue" still adequately describes something that took this long is a question of vocabulary I'll leave to others.
