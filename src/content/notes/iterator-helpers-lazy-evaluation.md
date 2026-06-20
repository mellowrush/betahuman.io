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

The reason a good campaign never shows the whole product at once isn't mystery for its own sake. It's economics. You reveal what the audience needs to make the next decision. Not more. Not the entire catalog.

Iterator helpers work on a similar principle: they don't build the full result. They process one element at a time, on demand, and stop when you have enough.

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

Chrome 122 (February 2024), Firefox 131 (October 2024). The methods:

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

## Processing a ReadableStream

The most compelling real-world case: data that arrives as a stream and shouldn't be fully buffered.

```javascript
async function processStreamLines(response) {
  const decoder = new TextDecoder();

  // Create an iterator from the response body
  const lines = response.body
    .values()
    .map(chunk => decoder.decode(chunk))
    .flatMap(text => text.split('\n').values())
    .filter(line => line.trim().length > 0);

  // Process first 100 non-empty lines without reading the rest of the stream
  const first100 = await lines.take(100).toArray();

  return first100.map(line => JSON.parse(line));
}
```

The stream never fully buffers. After 100 valid lines, the iterator stops consuming. If the stream has 10,000 lines, 9,900 of them are never decoded, never split, never filtered.

## An infinite sequence

```javascript
function* naturals() {
  let n = 0;
  while (true) yield n++;
}

// Find the first 5 prime numbers without building an intermediate array
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

You can't do this with array methods because you can't have an infinite array. The lazy evaluation model makes infinite sequences viable.

## Comparing to array methods honestly

Iterator helpers are not replacements for array methods. They're additive:

| Scenario | Array methods | Iterator helpers |
|---|---|---|
| Small arrays (<1000) | Correct. Simpler. | Correct. Slightly more verbose. |
| Large arrays, need all results | Same performance. | No advantage. |
| Large arrays, need first N | Build-then-slice (wasteful). | Take N (efficient). |
| Infinite sequences | Impossible. | Native use case. |
| Streaming data | Must buffer first. | Process as it arrives. |

The sweet spots are real: large data sets with early termination, streaming responses, and infinite generators. Outside those, array methods remain the readable choice.

## The protocol underneath

Everything in JavaScript that is iterable — arrays, sets, maps, strings, generator functions, ReadableStreams — implements the iterator protocol. Iterator helpers work on any of them. You're not limited to arrays.

```javascript
const activeUsers = new Set(['alice', 'bob', 'carol']);

// Sets don't have .filter() on them directly.
// Values() returns an iterator, which does.
const shortNames = activeUsers.values()
  .filter(name => name.length <= 4)
  .toArray();
// → ['bob', 'carol'] — wait, 'carol' is 5 chars, so → ['bob']
```

The protocol also composes: if you build a generator that yields from another generator, the lazy evaluation composes across the chain. No materialization until you explicitly request it.

Data is not free. Memory is not free. The practice of building complete intermediate structures when you only needed a fraction of them has always been a cost we normalized because there was no alternative.

Now there is.
