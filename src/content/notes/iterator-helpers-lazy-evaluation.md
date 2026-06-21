---
title: "Iterator helpers: processing without materializing"
date: "2024-09-19"
category: JavaScript
description: "Iterator helpers — .map(), .filter(), .take() on iterators — landed in Chrome 122 and Firefox 131. The difference from array methods: they don't build the result until you ask for it. Here's when that matters."
tags:
  - javascript
  - iterators
  - performance
  - lazy-evaluation
draft: false
---

Iterator helpers process one element at a time, on demand and stop when you have enough. They don't build the full result in advance.

The difference from array methods:

```javascript
// Array methods: materialize everything first
const result = hugeArray
  .filter(x => x.active)           // new array: n elements
  .map(x => x.value * 2)           // new array: n elements
  .slice(0, 10);                    // new array: 10 elements
// Built three full arrays. Used 10 values.

// Iterator methods: process one at a time, stop at 10
const result = hugeArray.values()
  .filter(x => x.active)           // lazy: no new array
  .map(x => x.value * 2)           // lazy: no new array
  .take(10)                         // lazy: stop at 10
  .toArray();                       // realize: 10 elements
// Processed n elements from filter perspective. Built one array of 10.
```

For small arrays, the difference is invisible. For large data sets or infinite sequences, it's meaningful.

> **Browser support:** Chrome 122 (February 2024), Firefox 131 (October 2024). Check [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Iterator) for Safari status. Try any of these in the browser console right now — Chrome and Firefox both have full support.

The methods:

```javascript
iterator
  .map(fn)           // transform each element
  .filter(predicate) // skip elements that don't match
  .take(n)           // stop after n elements
  .drop(n)           // skip first n elements
  .flatMap(fn)       // map + flatten one level
  .forEach(fn)       // iterate for side effects (terminal)
  .find(predicate)   // find first match (terminal, returns element or undefined)
  .some(predicate)   // short-circuits true on first match (terminal)
  .every(predicate)  // short-circuits false on first non-match (terminal)
  .reduce(fn, init)  // fold (terminal)
  .toArray()         // realize to array (terminal)
```

Terminal methods consume the iterator. Lazy methods chain without consuming.

## An infinite sequence

```javascript
function* naturals() {
  let n = 0;
  while (true) yield n++;
}

function isPrime(n) {
  if (n < 2) return false;
  for (let i = 2; i <= Math.sqrt(n); i++) {
    if (n % i === 0) return false;
  }
  return true;
}

const firstFivePrimes = naturals()
  .filter(isPrime)
  .take(5)
  .toArray();

// → [2, 3, 5, 7, 11]
```

You can't do this with array methods because you can't have an infinite array. Lazy evaluation makes infinite sequences viable. Paste this into a Chrome or Firefox console — it runs.

## Processing a ReadableStream

> **Note on async iterators:** The iterator helpers in this spec (`Iterator.prototype.*`) work on **sync** iterators. `ReadableStream` is async iterable. Async iterator helpers are a separate, later TC39 proposal and don't yet have browser support. The pattern below works for sync iterators over buffered data — not for streaming responses in real time.

```javascript
// Works for sync iterables (arrays, sets, generators)
const lines = largeTextContent.split('\n').values()
  .filter(line => line.trim().length > 0)
  .map(line => JSON.parse(line))
  .take(100)
  .toArray();
// Stops processing after 100 valid JSON lines
```

## Comparing to array methods honestly

Iterator helpers are not replacements for array methods. They're additive:

| Scenario | Array methods | Iterator helpers |
|---|---|---|
| Small arrays (<1000) | Correct. Simpler. | Correct. Slightly more verbose. |
| Large arrays, need all results | Same performance. | No advantage. |
| Large arrays, need first N | Build-then-slice (wasteful). | Take N (efficient). |
| Infinite sequences | Impossible. | Native use case. |
| Streaming data (sync) | Must buffer first. | Process on demand. |

The sweet spots are real: large data sets with early termination, infinite generators and chaining across Sets and Maps without intermediate arrays. Outside those, array methods remain the readable choice.

## The protocol underneath

Everything in JavaScript that is iterable — arrays, sets, maps, strings, generator functions — implements the iterator protocol. Iterator helpers work on any of them:

```javascript
const activeUsers = new Set(['alice', 'bob', 'carol']);

// Sets don't have .filter() directly.
// .values() returns an iterator, which does.
const shortNames = activeUsers.values()
  .filter(name => name.length <= 4)
  .toArray();
// → ['bob', 'carol'] — wait, 'carol' is 5 chars, so → ['bob']
```

The protocol also composes: if you build a generator that yields from another generator, the lazy evaluation composes across the chain. No materialization until you explicitly request it.

Data is not free. Memory is not free. The practice of building complete intermediate structures when you only needed a fraction of them has always been a cost we normalized because there was no alternative.

Now there is.
