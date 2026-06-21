---
title: "Abusing @keyframes as a state machine (it works)"
date: "2024-03-21"
category: CSS
description: "animation-play-state + animation-delay with a negative value lets you scrub to any frame of an animation. Which means you can use CSS animations as state machines."
tags:
  - css
  - animation
  - patterns
  - weird
draft: false
---

Something technically inadvisable that works perfectly in production.

CSS `animation-play-state: paused` + a negative `animation-delay` lets you scrub to any keyframe of an animation. The browser renders the frame at that offset and doesn't advance. Move between states by updating the delay value — via a custom property, a checkbox, or a `data-` attribute.

This is a state machine. With CSS.

```css
@keyframes toggle-switch {
  0%   { transform: translateX(0);    background: #999; }
  100% { transform: translateX(24px); background: #22c55e; }
}

.toggle-thumb {
  width: 20px;
  height: 20px;
  border-radius: 50%;
  animation: toggle-switch 200ms ease both;
  animation-play-state: paused;
  animation-delay: calc(var(--state, 0) * -200ms);
}

/* When the hidden checkbox is checked, jump to the end frame */
input:checked ~ .toggle .toggle-thumb {
  --state: 1;
}
```

When `--state` is `0`, delay is `0ms` — the animation is at its start frame. When `--state` is `1`, delay is `-200ms` — the animation is "200ms into" a 200ms animation, which is the end frame. Nothing plays. You're statically positioned at a keyframe.

Change `--state` to `0.5` and you're at the exact midpoint. The browser interpolates.

> **Browser compatibility:** This technique uses only longstanding CSS features — no new APIs. Tested in Chrome 100+, Firefox 100+, Safari 15.4+. The `transition: animation-delay` approach (see below) works in Chrome and Firefox; Safari may not transition `animation-delay` as a standalone property. The custom property approach with `@property` registration is more reliable cross-browser.

## A working toggle with transition between states

The state *change* can still be animated — not by playing the animation forward, but by animating the delay value:

```css
.toggle-thumb {
  animation: toggle-switch 200ms ease both;
  animation-play-state: paused;
  animation-delay: calc(var(--state, 0) * -200ms);
  transition: animation-delay 200ms ease;
}
```

When `--state` switches from `0` to `1`, the delay transitions from `0` to `-200ms`. Because the delay change is animated, the browser scrubs through the keyframes on the way. The toggle slides.

For more reliable cross-browser behavior, register the custom property and transition it directly:

```css
@property --state {
  syntax: '<number>';
  inherits: true;
  initial-value: 0;
}

.toggle-thumb {
  animation: toggle-switch 200ms ease both;
  animation-play-state: paused;
  animation-delay: calc(var(--state) * -200ms);
  transition: --state 200ms ease;
}
```

This is the kind of thing that makes you suspicious the CSS spec authors knew exactly what they were doing.

<div class="ksm-demo" role="region" aria-label="Live demo: keyframe state machine toggle">
  <style>
    .ksm-demo {
      border: 1px solid var(--bh-border-strong, #d4c8c0);
      border-radius: 8px;
      padding: 1rem 1.25rem;
      margin: 1.75rem 0;
      background: var(--bh-bg, #faf7f4);
    }
    .ksm-demo-label {
      font-family: monospace;
      font-size: 0.7rem;
      letter-spacing: 0.08em;
      text-transform: uppercase;
      color: var(--bh-text-faint, #9a8f87);
      margin-bottom: 1rem;
      display: block;
    }
    .ksm-toggle-wrap {
      display: flex;
      align-items: center;
      gap: 0.875rem;
    }
    .ksm-checkbox {
      position: absolute;
      opacity: 0;
      width: 0;
      height: 0;
    }
    .ksm-toggle {
      width: 48px;
      height: 28px;
      border-radius: 14px;
      background: #d1cac5;
      position: relative;
      cursor: pointer;
      flex-shrink: 0;
    }
    @keyframes ksm-thumb-slide {
      0%   { transform: translateX(2px);  background: #fff; }
      100% { transform: translateX(22px); background: #fff; }
    }
    @keyframes ksm-track-color {
      0%   { background: #c9c2bc; }
      100% { background: #c4522a; }
    }
    .ksm-thumb {
      width: 24px;
      height: 24px;
      border-radius: 50%;
      position: absolute;
      top: 2px;
      box-shadow: 0 1px 3px rgba(0,0,0,0.3);
      animation: ksm-thumb-slide 200ms ease both;
      animation-play-state: paused;
      animation-delay: calc(var(--ksm-state, 0) * -200ms);
      transition: animation-delay 200ms ease;
    }
    .ksm-toggle {
      animation: ksm-track-color 200ms ease both;
      animation-play-state: paused;
      animation-delay: calc(var(--ksm-state, 0) * -200ms);
      transition: animation-delay 200ms ease;
    }
    .ksm-checkbox:checked ~ .ksm-toggle-wrap .ksm-toggle,
    .ksm-checkbox:checked ~ .ksm-toggle-wrap .ksm-thumb {
      --ksm-state: 1;
    }
    .ksm-label-text {
      font-size: 0.875rem;
      color: var(--bh-text, #2a1f1b);
      cursor: pointer;
    }
    @media (prefers-reduced-motion: reduce) {
      .ksm-thumb, .ksm-toggle { transition: none; }
    }
  </style>
  <span class="ksm-demo-label">↓ Click the toggle — pure CSS state machine</span>
  <input type="checkbox" class="ksm-checkbox" id="ksm-cb">
  <div class="ksm-toggle-wrap">
    <label class="ksm-toggle" for="ksm-cb">
      <div class="ksm-thumb"></div>
    </label>
    <label class="ksm-label-text" for="ksm-cb">
      <code>--ksm-state: 0 → 1</code>
    </label>
  </div>
</div>

## A stepper component (three states)

```css
@keyframes step-progress {
  0%   { width: 0%; }
  33%  { width: 33.3%; }
  66%  { width: 66.6%; }
  100% { width: 100%; }
}

.step-bar {
  animation: step-progress 300ms ease both;
  animation-play-state: paused;
  animation-delay: calc(var(--step, 0) * -100ms);
  transition: animation-delay 300ms ease;
}
```

```html
<div class="stepper" style="--step: 0">
  <div class="step-bar"></div>
</div>
```

Increment `--step` with a single `style.setProperty` call and the progress bar smoothly advances to the corresponding keyframe. Three states. One animation. No per-state class switching.

## When to actually use this

This technique shines for:

- **Binary toggles** that need smooth transitions (switches, expand/collapse)
- **Progress indicators** with fixed step counts
- **Icon morphs** where start and end states are defined but intermediate is decorative

It's a bad fit for:

- **Many states** — more than ~5 keyframe jumps gets hard to reason about
- **Conditional transitions** — different animation between state A→B vs B→A requires two animation definitions
- **Anything that needs accurate progress reporting** — you're scrubbing CSS, not reading a clock

The real reason to know this technique isn't to use it everywhere. It's to understand that the animation system you think of as "play something over time" is actually a "map time to state" system — and time is just one of several possible inputs.

Negative delays are a scrubber. That's always been true. We just rarely needed to exploit it.
