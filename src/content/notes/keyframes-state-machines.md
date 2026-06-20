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

Let me show you something technically inadvisable that works perfectly in production.

The technique: CSS `animation-play-state: paused` + a negative `animation-delay` lets you "scrub" to any keyframe of an animation. The browser renders the frame at that offset but doesn't advance. You move between states by updating the delay value — which you can do via a custom property, a checkbox, or a `data-` attribute.

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
  animation: toggle-switch 200ms var(--ease) both;
  animation-play-state: paused;
  animation-delay: calc(var(--state, 0) * -200ms);
}

/* When the hidden checkbox is checked, jump to the end frame */
input:checked ~ .toggle .toggle-thumb {
  --state: 1;
}
```

When `--state` is `0`, `animation-delay` is `0ms` — the animation is at its start frame. When `--state` is `1`, `animation-delay` is `-200ms` — the animation is "200ms into" a 200ms animation, which is the end frame. The browser renders the final state. Nothing plays. You're statically positioned at a keyframe.

Change `--state` to `0.5` and you're at the exact midpoint. The browser interpolates.

## A working toggle with transition between states

The beautiful part is that the state *change* can still be animated — just not by playing the animation forward. You change the delay with a transition:

```css
.toggle-thumb {
  animation: toggle-switch 200ms ease both;
  animation-play-state: paused;
  animation-delay: calc(var(--state, 0) * -200ms);
  transition: animation-delay 200ms ease;
}
```

When `--state` switches from `0` to `1`, the `animation-delay` value transitions from `0` to `-200ms`. Because the delay changes are animated, the browser interpolates through the keyframes on the way. The toggle slides.

This is the kind of thing that makes you suspicious the CSS spec authors knew exactly what they were doing.

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
  <!-- content -->
</div>
```

Increment `--step` with JavaScript (a single `style.setProperty` call) and the progress bar smoothly advances to the corresponding keyframe. Three states. One animation. No per-state class switching.

## When to actually use this

The honest answer: not everywhere. This technique shines for:

- **Binary toggles** that need smooth transitions (checkboxes, switches, expand/collapse)
- **Progress indicators** with fixed step counts
- **Icon morphs** where the start and end states are defined but intermediate is decorative

It's a bad fit for:

- **Many states** (more than ~5 keyframe jumps gets hard to reason about)
- **Conditional transitions** (different animation between state A→B vs B→A — you'd need two animation definitions)
- **Anything that needs accurate progress reporting** (you're scrubbing CSS, not reading a clock)

The real reason to know this technique isn't to use it everywhere. It's to understand that the animation system you think of as "play something over time" is actually a "map time to state" system, and time is just one of several possible inputs.

Negative delays are a scrubber. That's always been true. We just rarely needed to exploit it.
