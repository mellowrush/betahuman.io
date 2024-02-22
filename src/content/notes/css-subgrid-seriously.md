---
title: Taking subgrid seriously
date: "2024-02-22"
category: CSS
description: Subgrid became baseline in 2023. Everyone shows the card grid demo. Here's the form layout case that actually rewrites how you think about it.
tags:
  - css
  - grid
  - layout
  - subgrid
draft: false
---

The card grid demo has been done. Beautifully aligned card titles across unrelated containers, subgrid threading columns down through nested elements. You've seen it. I've seen it. We've all admired it in a CodePen and moved on.

What's less documented is the form layout case, which is subtler and significantly more useful in daily work.

The problem: you have a form where labels and inputs should align across components that don't share a parent. Previously this required JavaScript measurement — read label widths, apply as a uniform left offset. Or you made every label a fixed width, which worked until the content changed.

```css
/* The form itself defines the track structure */
.form {
  display: grid;
  grid-template-columns: max-content 1fr;
  gap: 0.75rem 1.5rem;
}

/* Each field group inherits those tracks */
.field {
  display: contents; /* vanishes structurally, children participate in parent grid */
}

.field label {
  grid-column: 1;
  align-self: center;
  font-size: 0.875rem;
  color: #666;
}

.field input,
.field select,
.field textarea {
  grid-column: 2;
}
```

The `display: contents` approach works, but it removes the field container from layout, which means you can't put a border or background on the `.field` element itself without breaking the grid flow.

Subgrid solves this cleanly:

```css
.form {
  display: grid;
  grid-template-columns: max-content 1fr;
  gap: 0.75rem 1.5rem;
}

.field {
  display: grid;
  grid-column: 1 / -1;         /* span full width */
  grid-template-columns: subgrid; /* inherit parent column tracks */
  align-items: center;
}

.field label  { grid-column: 1; }
.field input  { grid-column: 2; }
```

Now `.field` is a real grid item — you can style it, border it, animate it — and its children *inherit* the parent's column tracks. The label in every `.field` lives on the same track. The input in every `.field` lives on the same track. The widths are computed once and shared.

## What rows look like (spoiler: different)

Rows in subgrid have constraints worth knowing. `grid-template-rows: subgrid` requires that the parent explicitly define row tracks too — you can't inherit auto rows. That limits the row inheritance story:

```css
/* This doesn't work the way you'd hope */
.card-grid {
  display: grid;
  grid-template-rows: auto; /* 'auto' can't be subgridded */
}

/* This does work */
.card-grid {
  display: grid;
  grid-template-rows: auto 1fr auto; /* named track count must be known */
}

.card {
  display: grid;
  grid-row: span 3;
  grid-template-rows: subgrid; /* inherits the 3-row definition */
}
```

The "aligned card titles across a row" demo works because the parent explicitly defines how many rows each card occupies. Genuinely implicit row count? Not yet subgrid-able. JavaScript still wins there.

## The deeper thing subgrid changes

CSS Grid forced a mental model shift in 2017: layout happens on the parent, not the child. Grid items *participate* in the parent's coordinate system.

Subgrid extends that shift one level deeper. Grandchildren can now participate in the grandparent's coordinate system. The grid becomes a shared spatial language across the DOM hierarchy, not just a local layout tool.

That sounds like marketing copy. It's actually the most useful thing that shipped to CSS in the last two years.

The JavaScript measurement pattern for aligned labels isn't dead because subgrid isn't powerful enough. It's dead because there's no longer a reason to do math the browser already does.

Baseline November 2023. Not a reason to wait.
