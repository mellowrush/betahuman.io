# Content Ideas
Experiments, POCs and notes worth writing — biased toward browser capabilities
that have landed (or are landing) since June 2024 but haven't made it into
the mainstream tutorial circuit yet. The question is never "what is this" but
"what can you actually do with it that you couldn't before and is it interesting?"

---

## CSS Experiments

- [ ] **Anchor Positioning without a single line of JS**
  Chrome 125+. `anchor()` function + `position-anchor` let you float an element
  relative to any other element in the DOM — through scroll, resize, overflow.
  POC: build a tooltip that correctly follows its trigger through a scrollable container.
  The interesting question: does this actually replace the dozen JS position-tracking
  libraries we've been reaching for, or just replace the simple cases?
  _Category: experiment · Estimated: 3–4h_

- [ ] **Scroll-driven animations: the CSS-only version of what we've been doing wrong**
  `animation-timeline: scroll()` and `animation-timeline: view()` land in Chrome 115
  but most people still reach for Intersection Observer + JS. Build a reading progress
  bar, a parallax header and a sticky section reveal — all in CSS, no JS.
  Then measure: what breaks, what doesn't, what the spec can't do yet.
  _Category: experiment · Estimated: 2h_

- [ ] **`@starting-style`: finally, enter animations without the DOM hack**
  For years: set `opacity: 0`, render, then flip to `opacity: 1` in a rAF.
  `@starting-style` makes the browser handle this natively. POC: animated toast
  notifications using only CSS — appear and disappear with `display: none` transitions.
  Compare bundle size vs. a headless UI animation library.
  _Category: experiment · Estimated: 2h_

- [ ] **Relative color syntax: one seed color, a whole system**
  `oklch(from var(--rust) calc(l * 0.8) c h)` — derive tints, shades and
  complementaries from a single design token. Build a live palette generator
  that produces a full design system from a single hex input. No preprocessor,
  no JS color library. Just CSS.
  _Category: build / POC · Estimated: 4–6h_

- [ ] **`calc-size()`: animating height to `auto` was genuinely impossible until now**
  Chrome 129+. Every accordion, every expandable section has had this hack:
  measure height with JS, animate to the pixel value. `calc-size(auto, size)` fixes it.
  Write the two-line CSS version, then write the note about what took us this long.
  _Category: short note + demo · Estimated: 1h_

- [ ] **CSS `@scope`: component-scoped styles without Shadow DOM or BEM**
  `@scope (.card) to (.card-body) { ... }` — style a component in isolation
  without the specificity arms race. The interesting experiment: take a component
  that currently relies on BEM, rewrite it with `@scope`, measure what's actually
  simpler and what the tradeoffs are.
  _Category: experiment · Estimated: 2–3h_

- [ ] **`@property`: the animatable custom property nobody talks about enough**
  Declare a custom property with a type (`<angle>`, `<color>`, `<length>`) and
  it becomes smoothly animatable. Build: a gradient that rotates on hover with
  CSS transitions only. Build: a progress ring using a `<number>` custom property.
  The point: the browser can't interpolate `--angle: 0deg` to `--angle: 360deg`
  without knowing the type. Now it can.
  _Category: experiment · Estimated: 2h_

- [ ] **`:has()` use cases that aren't obvious**
  Parent selector coverage is everywhere now but the interesting cases aren't.
  Collection: style a `<form>` that contains an invalid `<input>`, style a card
  differently when it contains an image vs. when it doesn't, build a tab system
  purely with `:has(:checked)`. No JS.
  _Category: note + demos · Estimated: 2h_

- [ ] **CSS Container Style Queries: component theming without JS prop drilling**
  `@container style(--variant: featured) { ... }` — query the value of a custom
  property, not just the container size. This means you can pass "variant" information
  through CSS alone. POC: a card component that renders differently in a "featured"
  context without any class changes in the markup.
  _Category: experiment · Estimated: 3h_

- [ ] **`initial-letter`: drop caps natively, no hacks**
  `p::first-letter { initial-letter: 3; }` — proper typographic drop caps, baseline-aligned,
  occupying N lines. Baseline now. Build a longform article layout that uses it and
  measure what breaks (RTL, edge cases with punctuation, inline images nearby).
  _Category: short note + demo · Estimated: 1h_

---

## Browser API Experiments

- [ ] **Popover API: rebuild a component library feature with zero JS**
  `[popover]` + `popovertarget` — native floating elements with focus management,
  light-dismiss and `::backdrop`. POC: a tooltip, a modal, a dropdown menu —
  each using only HTML attributes. Then the honest verdict: where does it actually
  cover the use case and where does it break down (nested popovers, animation)?
  _Category: experiment · Estimated: 4h_

- [ ] **Speculation Rules API: prerender the next page before the click**
  `<script type="speculationrules">` — declarative prefetch/prerender based on
  rules you define. Build: a site where every internal link feels instant because
  the next page is already rendered in the background. Measure: how much memory
  does aggressive speculation use? What's the right threshold?
  _Category: experiment + note · Estimated: 3h_

- [ ] **`scrollend` event: the reliable scroll-end detection we've been debouncing forever**
  Replaces `scroll` + `setTimeout(fn, 150)` with a real event that fires when
  inertia scroll has actually stopped. POC: a sticky element that repositions
  only after scroll has genuinely ended. Compare the old pattern vs. the new.
  _Category: short note + demo · Estimated: 1h_

- [ ] **`content-visibility: auto` on a genuinely long page**
  The browser skips rendering offscreen content until it's near the viewport.
  The experiment: take a page with 500 items, measure paint time with and without,
  understand the gotchas (coordinate system shifts, find-in-page issues, off-screen
  image loading behavior).
  _Category: performance experiment · Estimated: 3h_

- [ ] **Navigation API: real SPA transitions without a router library**
  `window.navigation.addEventListener('navigate', ...)` intercepts navigations
  at the browser level — back/forward, link clicks, form submissions. POC: custom
  view transitions on a multi-page site without any client-side router.
  _Category: experiment · Estimated: 4–5h_

- [ ] **Declarative Shadow DOM + Astro: server-rendered web components**
  `<template shadowrootmode="open">` streams shadow roots from the server.
  No JS required to hydrate. POC: build an Astro component that is a real
  web component, style-encapsulated, SSR'd, zero client JS.
  _Category: build · Estimated: 4h_

---

## JavaScript Language Experiments

- [ ] **`Promise.withResolvers()`: the deferred pattern done right**
  `const { promise, resolve, reject } = Promise.withResolvers()` — clean up
  every pattern where you store `resolve` in an outer variable to call it later.
  Write a real use case: a loading state that resolves when a WebSocket message
  arrives, an animation that resolves when `transitionend` fires.
  _Category: short note + example · Estimated: 1h_

- [ ] **Set operations: `intersection`, `union`, `difference` are just there now**
  Chrome 122+, Firefox 127+. Build something that would have required lodash
  before: a tag filter UI, a permission system, a "what changed" diff.
  The note: why did this take until 2024?
  _Category: short note · Estimated: 1h_

- [ ] **Iterator helpers: lazy evaluation over arrays you haven't built yet**
  `.map()`, `.filter()`, `.take()`, `.drop()` on iterators. The difference from
  array methods: they don't materialize the result. POC: process a stream of
  events from a `ReadableStream` with iterator helpers, compare to the array
  approach. Worth it for large data; pointless for small arrays — write the honest verdict.
  _Category: experiment · Estimated: 2h_

- [ ] **`Temporal` API: treat it as if it already exists**
  Stage 3, polyfill available. `Temporal.PlainDate`, `Temporal.ZonedDateTime`,
  `Temporal.Duration` — what `Date` should have been. Experiment: replace every
  `date-fns` call in a small project with `Temporal`. What's better? What's
  missing from the polyfill? What's just different?
  _Category: experiment · Estimated: 3–4h_

---

## Weird / Out-of-the-box Angles

- [ ] **A single-CSS-property dark mode with `light-dark()`**
  `color: light-dark(#1a1814, #f5f2eb)` — one token, two values, browser picks
  based on `prefers-color-scheme`. Experiment: convert the entire betahuman token
  system to `light-dark()` and remove the `[data-theme]` JavaScript entirely.
  What breaks? Is the JS toggle still needed? What's the real tradeoff?
  _Category: experiment · build · Estimated: 3h_

- [ ] **QR codes as deliberate visual artifacts: a proper investigation**
  QR codes have error correction baked in (up to 30% of codewords can be wrong
  and it still scans). That means you can cover a chunk of the pattern with art.
  The interesting question: can you do this client-side, purely in canvas or SVG,
  with a clean API? POC for QR Artifier.
  _Category: build / research · Estimated: 6–8h_

- [ ] **Resume as SVG: typographically correct, client-side, zero server**
  Most resume generators produce either a pixel-perfect PDF (from HTML) or a
  generic Word doc. An SVG resume is: scalable, portable, editable in any vector
  tool and can embed animation. The interesting constraint: all text must remain
  selectable and copy-pastable. Dig into `<text>` + `<textPath>` + SVG font metrics.
  _Category: build / research · Estimated: 8–12h_

- [ ] **Font Pairing in context, not in a specimen**
  Every font pairing tool shows you "Heading / Body" in a vacuum. The problem:
  typefaces read very differently at UI scale, in dark mode, at 13px. Build a live
  pairing lab where you see the combo in a real component: a card, a form, a nav.
  The opinionated add: a "bad pairing" detector based on contrast ratios + x-height matching.
  _Category: build · Estimated: 6h_

- [ ] **Abusing `@keyframes` for state machines**
  CSS `animation-play-state: paused` + `animation-delay: -Ns` lets you jump to
  any point in an animation by setting a negative delay. This is functionally a
  state machine with no JS. POC: a toggle switch, a stepper component, a loading
  state — all driven by CSS custom properties and checked inputs, zero JS.
  _Category: experiment · Estimated: 3h_

- [ ] **`fetchpriority` and what it actually does to LCP**
  `<img fetchpriority="high">` on the hero image. Sounds obvious but the
  interaction with lazy loading, connection ramp-up and preload scanner is
  non-trivial. Measure with and without on a real page, WebPageTest waterfall
  included. The honest verdict on whether it matters below a certain image size.
  _Category: performance experiment · Estimated: 2h_

- [ ] **What happens when you take CSS subgrid seriously**
  `grid-template-columns: subgrid` is baseline now. The use case everyone shows:
  aligned card grids. The more interesting case: a form where label and input
  columns align across unrelated parent elements. Build a form layout that would
  have required JS measurement before. Document what subgrid can't do (hint: rows
  don't behave the same way).
  _Category: experiment · Estimated: 2–3h_

- [ ] **The browser has a proper color picker. We've been ignoring it.**
  `<input type="color">` supports `list` attribute for swatches since Chrome 99.
  The `EyeDropper` API (Chrome 95+) lets you pick any pixel on screen. POC:
  build a color palette tool that lets you sample colors from anywhere on your
  screen, then generates a harmony using CSS relative colors. No color library.
  _Category: build · Estimated: 4h_

---

## Promised / Not-Yet-Released (worth watching)

- [ ] **Invoker Commands (`invoketarget` / `invokeaction`) — write the note when it ships**
  `<button invoketarget="dialog-id" invokeaction="showModal">` removes the need
  for JavaScript event listeners to connect interactive elements. Currently
  Chrome behind a flag. When it's baseline: write the "we didn't need JS for this"
  article and build the component library version vs. the HTML-only version.
  _Status: flag in Chrome · watch: https://chromestatus.com/feature/5712623289819136_

- [ ] **CSS `@function` (Stage 1 proposal) — track and experiment early**
  Define reusable CSS functions with parameters: `@function --space($n) { result: calc($n * 8px); }`.
  If it ships, rewrites how design tokens work. Set up an experiment when
  the first implementation lands.
  _Status: proposal, no implementation yet_

- [ ] **Scoped custom element registries — for when web components actually compose**
  Define custom elements scoped to a shadow root, not globally. Means two
  components can both register `<x-button>` without conflict. The thing that would
  make web component libraries actually composable. Experiment when shipped.
  _Status: Chrome in progress_

- [ ] **`interpolate-size: allow-keywords` — the other way to animate to `auto`**
  An opt-in property that lets the browser interpolate between fixed values
  and CSS keywords (`auto`, `min-content`, `fit-content`). Works differently
  from `calc-size()`. Worth comparing both approaches when it's stable.
  _Status: Chrome 129+ behind flag_
