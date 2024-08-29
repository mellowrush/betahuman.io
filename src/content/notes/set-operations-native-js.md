---
title: "Set.prototype.intersection() and the six years we spent on lodash"
date: "2024-08-29"
category: JavaScript
description: Native Set operations — intersection, union, difference, symmetricDifference — became baseline in 2024. Here's what you can delete from your dependencies.
tags:
  - javascript
  - sets
  - data-structures
  - baseline
draft: false
---

In the interest of full disclosure: I'm going to show you a feature that shipped in every major browser by mid-2024 and then I'm going to ask you to spend a moment thinking about how many times you installed lodash to do this.

I'll try to be kind about it.

```javascript
// The old way
import { intersection, union, difference } from 'lodash';

const userPermissions  = new Set(['read', 'write', 'comment']);
const requiredPerms    = new Set(['read', 'write', 'delete']);
const commonPerms      = intersection([...userPermissions], [...requiredPerms]);
// → ['read', 'write']
```

Note the spread operators. lodash works on arrays. You had to convert, operate, and decide whether to convert back. Not complicated — just a ceremony that existed because there was no native alternative.

```javascript
// Now
const userPermissions = new Set(['read', 'write', 'comment']);
const requiredPerms   = new Set(['read', 'write', 'delete']);

userPermissions.intersection(requiredPerms);      // Set {'read', 'write'}
userPermissions.union(requiredPerms);             // Set {'read', 'write', 'comment', 'delete'}
userPermissions.difference(requiredPerms);        // Set {'comment'}
userPermissions.symmetricDifference(requiredPerms); // Set {'comment', 'delete'}
userPermissions.isSubsetOf(requiredPerms);        // false
userPermissions.isSupersetOf(requiredPerms);      // false
userPermissions.isDisjointFrom(new Set(['admin'])); // true
```

All return new Sets. The originals are not mutated. The methods accept any iterable — you can pass an array and it works.

## Why this took until 2024

JavaScript Sets were added in ES6 (2015). The operations — the things that make Sets genuinely useful as a data structure — were conspicuously absent. The reason is boring: specification process. Someone has to write the proposal, it goes through stages, implementations follow, it reaches baseline.

Chrome 122 shipped in February 2024. Firefox 127 in June 2024. Safari 17 in September 2023. Baseline: June 2024.

Nine years to add methods that were in the original mathematical definition of a set.

I won't say it's fine. It's also not worth being angry about. The timeline is what it is.

## Real use cases that feel genuinely nicer

**Tag filtering system:**

```javascript
const activeFilters = new Set(['css', 'performance']);
const articleTags   = new Set(['css', 'animation', 'performance', 'browser-api']);

// Does this article match any active filter?
const hasMatch = !activeFilters.isDisjointFrom(articleTags); // true
const matches  = activeFilters.intersection(articleTags);    // Set {'css', 'performance'}
```

**User permission checking:**

```javascript
function canAccess(userRoles, requiredRoles) {
  return !userRoles.isDisjointFrom(new Set(requiredRoles));
}

canAccess(new Set(['editor', 'commenter']), ['admin', 'editor']); // true
```

**Finding what changed between two data snapshots:**

```javascript
const prevIds = new Set([1, 2, 3, 4, 5]);
const nextIds = new Set([3, 4, 5, 6, 7]);

const added   = nextIds.difference(prevIds);     // Set {6, 7}
const removed = prevIds.difference(nextIds);     // Set {1, 2}
const kept    = prevIds.intersection(nextIds);   // Set {3, 4, 5}
```

This is the "what changed" pattern. Previously: convert both to arrays, filter with `.includes()`, `O(n²)` unless you're careful. Now: two Set method calls, `O(n)` by the spec.

## The lodash eulogist

lodash is not deprecated by this. lodash has `_.chunk`, `_.debounce`, `_.throttle`, `_.cloneDeep`, `_.get`, and approximately four hundred other utilities that remain useful. It's a good library. It's been a good library for a decade.

What's changed is that the category of lodash utilities that exist because JavaScript doesn't have native equivalents keeps shrinking. Array methods ate a significant chunk in ES6. `Object.entries`, `Object.fromEntries`, optional chaining, nullish coalescing ate more. Now Set operations.

The library isn't dying. Its necessity for certain operations is. Both things can be true.

If your lodash import is specifically for Set intersection/union/difference: you can delete that dependency today. The browser has it. It's been there since June. You just didn't get the memo.

Now you have.
