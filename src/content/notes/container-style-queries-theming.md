---
title: "Container style queries: component theming without prop drilling"
date: "2024-07-18"
category: CSS
description: "Container style queries let you query a custom property value on a container and style descendants based on it. Component variants through CSS, with no class changes in the markup."
tags:
  - css
  - components
  - design-system
  - custom-properties
draft: false
---

In most design systems, there's a moment where you look at your component API and realize it has sixteen props, twelve of which are just different ways to say "make it look like the other version." 

`variant`, `intent`, `tone`, `appearance`, `emphasis`, `size`, `density`, `level`. Pick your taxonomy. The component still ships six times its useful surface area in prop surface area, and every consumer has to read the docs to know which combination of props produces the thing they saw in the Figma file.

CSS Container Style Queries ship in Chrome 111+, with style query support (distinct from size queries) becoming available in Chrome 117. They let you do this:

```html
<!-- Set a context variable on the container -->
<section style="--card-variant: featured">
  <div class="card">
    <h2 class="card-title">This card is featured</h2>
    <p class="card-body">It looks different. The markup didn't change.</p>
  </div>
</section>
```

```css
/* The card knows nothing about its context except what CSS tells it */
.card {
  container-type: style;
  padding: 1rem;
  border-radius: 6px;
  background: white;
  border: 1px solid #e0e0e0;
}

/* Query the container for the variant value */
@container style(--card-variant: featured) {
  .card {
    background: oklch(97% 0.04 50);
    border-color: oklch(70% 0.15 50);
    border-width: 2px;
  }

  .card-title {
    font-size: 1.4rem;
    color: oklch(40% 0.15 50);
  }
}
```

When the card is inside a container where `--card-variant: featured` is set, it renders differently. When it's in a normal context, it renders normally. The card component has zero knowledge of where it lives — the context provides the information through CSS custom properties.

## The prop drilling problem in practice

In React, context-aware component theming usually looks like this:

```jsx
// Provider at the top of the feature section
<FeaturedContext.Provider value={{ variant: 'featured' }}>
  <Card />
</FeaturedContext.Provider>

// Inside the Card component
const { variant } = useContext(FeaturedContext);
return (
  <div className={`card ${variant === 'featured' ? 'card--featured' : ''}`}>
    ...
  </div>
);
```

Or, if you're not using Context, you're passing `variant` as a prop through every intermediate component in the tree. The technical term for this is "prop drilling." The accurate term is "paperwork."

CSS Container Style Queries remove the JavaScript layer from this pattern:

```css
/* No React Context. No prop. Just a custom property on an ancestor. */
.hero-section {
  --card-variant: featured;
}
```

The `container-type: style` declaration on `.card` makes it a style query container, which means any child can query its custom properties. The card responds to context without being told directly.

## Design system tokens as query values

```css
/* Token-driven variant system */
@container style(--density: compact) {
  .card { padding: 0.5rem; }
  .card-title { font-size: 1rem; }
}

@container style(--density: comfortable) {
  .card { padding: 1.5rem; }
  .card-title { font-size: 1.25rem; }
}

@container style(--density: spacious) {
  .card { padding: 2rem; }
  .card-title { font-size: 1.5rem; }
}
```

```html
<!-- Dashboard in compact mode -->
<div class="dashboard" style="--density: compact">
  <!-- All cards inside are compact -->
</div>
```

One token on the ancestor. Every descendant card adapts. No prop, no class, no context provider.

## The caveats

**Performance**: Style queries cause style recalculation on any descendant when the queried property changes. Keep the query scope narrow and avoid changing style query values in high-frequency events (scroll, mouse move).

**No boolean logic**: You can query for a specific value, but there's no `@container style(--variant != default)` or `@container style(--variant: featured or promoted)`. One condition per query block.

**Computed vs. inherited values**: The query reads the computed value of the property on the container. If the property isn't set at all, it reads as its initial value (or empty). Make sure your defaults are explicit.

This isn't a replacement for component APIs — it's an alternative to one specific kind of them. The kind that exists purely to communicate visual context through the JavaScript layer when CSS always had the ability to communicate context through inheritance. That particular kind of prop can retire.
