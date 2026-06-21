---
title: "Promise.withResolvers(): the deferred pattern, finally legible"
date: "2024-07-30"
category: JavaScript
description: "For ten years we've been storing Promise resolve/reject functions in outer-scope variables to call them later. Promise.withResolvers() is the native version of that pattern, and it's baseline now."
tags:
  - javascript
  - async
  - promises
  - patterns
draft: false
---

The deferred promise pattern: you need to resolve a promise from outside its executor. The old ceremony:

```javascript
let resolve, reject;

const promise = new Promise((res, rej) => {
  resolve = res;
  reject = rej;
});

// resolve/reject are accessible outside the constructor.
// Works. Slightly uncomfortable every time.

socket.on('message', (data) => {
  if (data.type === 'ready') resolve(data);
  if (data.type === 'error') reject(data.error);
});

await promise;
```

The discomfort: `let resolve, reject` is technically uninitialized — TypeScript flags it. The assignment happens inside the executor, which runs synchronously, so by the time you reach the outer code both are defined. But the pattern relies on the reader knowing that Promise executors run synchronously — a detail new developers correctly don't know and experienced developers incorrectly assume everyone knows.

`Promise.withResolvers()` became Baseline in July 2024:

```javascript
const { promise, resolve, reject } = Promise.withResolvers();

socket.on('message', (data) => {
  if (data.type === 'ready') resolve(data);
  if (data.type === 'error') reject(data.error);
});

await promise;
```

Destructured from a single call. Types are correct without assertion. No implicit knowledge of executor timing required.

> **Browser support:** Baseline July 2024. Chrome 119+, Firefox 121+, Safari 17.4+. [MDN reference](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/withResolvers). Try it now in any modern browser console.

## The animationend / transitionend case

Animation events are the most common real-world use:

```javascript
async function waitForAnimation(element, animationName) {
  const { promise, resolve } = Promise.withResolvers();

  element.addEventListener('animationend', (event) => {
    if (event.animationName === animationName) {
      resolve(event);
    }
  }, { once: true });

  return promise;
}

// Usage
await waitForAnimation(dialog, 'dialog-exit');
dialog.remove();
```

Without `withResolvers`, you'd either nest a callback (losing the await ergonomics) or store the resolve function in an outer variable. With it, the deferred resolve lives cleanly inside the function.

## The WebSocket ready state

```javascript
function waitForSocketReady(ws) {
  if (ws.readyState === WebSocket.OPEN) {
    return Promise.resolve(ws);
  }

  const { promise, resolve, reject } = Promise.withResolvers();

  ws.addEventListener('open', () => resolve(ws), { once: true });
  ws.addEventListener('error', reject, { once: true });

  return promise;
}

const socket = new WebSocket(url);
await waitForSocketReady(socket);
socket.send(JSON.stringify({ type: 'subscribe', channel: 'updates' }));
```

## Building a manual gate

```javascript
// A promise that can be resolved externally — useful for testing and coordination
class Gate {
  constructor() {
    const { promise, resolve, reject } = Promise.withResolvers();
    this.wait  = () => promise;
    this.open  = resolve;
    this.close = reject;
  }
}

const gate = new Gate();

async function worker() {
  await gate.wait();
  doTheActualWork();
}

setTimeout(() => gate.open(), 2000);
```

Five lines. Before `withResolvers()`, those five lines had a `let` declaration and a timing assumption in the constructor.

## What it doesn't change

`Promise.withResolvers()` is a quality-of-life improvement for a specific pattern. It doesn't:

- Make deferred promises more common or advisable (they're already common enough)
- Replace `async/await` for sequential flows
- Solve the "forgotten reject" problem — you still need to call `reject` on error paths

It removes the ceremony from a pattern that was always correct but unnecessarily awkward to write. Most great language improvements are this quiet. They don't introduce new capabilities — they reduce the distance between intent and expression.

The language has been doing this more often lately, which suggests someone on the committee is paying attention.
