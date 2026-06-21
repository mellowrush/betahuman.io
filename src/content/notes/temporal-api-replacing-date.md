---
title: "Date was a mistake. Temporal is the correction."
date: "2024-10-10"
category: JavaScript
description: "The Temporal API is Stage 3. The polyfill works. Here's what Date got wrong, what Temporal does differently, and whether you should replace date-fns today."
tags:
  - javascript
  - temporal
  - dates
  - api
draft: false
---

`Date` was ported from Java in 1995 by a developer who, by his own account, had ten days to do it. The resulting API is month-zero-indexed (January is `0`, which nobody expected in 1995 and nobody accepts gracefully now), uses local time by default while internally storing UTC (producing timezone bugs that are technically correct and experientially inexplicable), and cannot represent a date without a time or a time without a date.

For thirty years, we shipped this.

The Temporal proposal is Stage 3. The polyfill from `@js-temporal/polyfill` works in production. Here's what's different.

## Concepts that Date didn't have

`Date` has one concept: a millisecond offset from the Unix epoch, disguised as a date-time object. Temporal has six:

```javascript
import { Temporal } from '@js-temporal/polyfill';

// A calendar date with no time component
const today = Temporal.Now.plainDateISO();
// → PlainDate { 2024-10-10 }

// A clock time with no date component
const time = Temporal.Now.plainTimeISO();
// → PlainTime { 14:35:22.481 }

// A date-time with no timezone (e.g., a meeting time in a local context)
const meeting = Temporal.PlainDateTime.from('2024-11-15T09:30:00');

// A moment in time, with timezone
const event = Temporal.ZonedDateTime.from('2024-11-15T09:30:00[Europe/Bucharest]');

// A span of time (duration)
const oneWeek = Temporal.Duration.from({ weeks: 1 });

// The actual current moment (replaces new Date())
const now = Temporal.Now.instant();
```

This is not a richer API bolted onto the same broken model. These are genuinely different types for genuinely different concepts.

## Date arithmetic that actually works

```javascript
// With Date
const today = new Date();
const nextMonth = new Date(today);
nextMonth.setMonth(nextMonth.getMonth() + 1);
// This is wrong when today is January 31st (produces March 2nd or 3rd)
// You need special handling for month-overflow. Almost nobody writes it.

// With Temporal
const today = Temporal.Now.plainDateISO();
const nextMonth = today.add({ months: 1 });
// If today is 2024-01-31, this produces 2024-02-29 (or 2024-02-28 if not a leap year)
// The overflow is handled correctly by default, with overflow option available.
```

```javascript
// Duration arithmetic
const departure = Temporal.ZonedDateTime.from('2024-10-10T08:00:00[Europe/Bucharest]');
const arrival   = Temporal.ZonedDateTime.from('2024-10-10T16:30:00[America/New_York]');

const flightDuration = departure.until(arrival);
// Duration correctly accounts for timezone offset.
// New York is UTC-4, Bucharest is UTC+3 — 7 hours difference.
// 8:00 Bucharest = 01:00 New York. Flight from 01:00 to 16:30 = 15.5 hours.
console.log(flightDuration.hours, flightDuration.minutes); // 15, 30
```

With `Date`, this calculation requires converting both to UTC milliseconds, subtracting, and parsing the result. Correct, tedious, easy to get wrong.

## Replacing date-fns

I ran this experiment on a small project with twelve `date-fns` usages:

```javascript
// Before: date-fns
import { format, addDays, differenceInDays, isAfter } from 'date-fns';

const due      = new Date(task.dueDate);
const today    = new Date();
const overdue  = isAfter(today, due);
const daysLeft = differenceInDays(due, today);
const reminder = format(addDays(due, -3), 'MMM d');

// After: Temporal polyfill
const due      = Temporal.PlainDate.from(task.dueDate);
const today    = Temporal.Now.plainDateISO();
const overdue  = Temporal.PlainDate.compare(today, due) > 0;
const daysLeft = today.until(due).days;
const reminder = due.subtract({ days: 3 })
                    .toLocaleString('en-US', { month: 'short', day: 'numeric' });
```

Results: removed `date-fns` (12.5KB gzipped). Added `@js-temporal/polyfill` (45KB gzipped). Net: larger bundle, because the polyfill includes the full Temporal implementation.

The honest assessment: **don't replace date-fns today for bundle-size reasons**. The polyfill is larger. The native Temporal API, when it ships in browsers without a polyfill, will cost nothing. The appropriate strategy:

1. Use the polyfill now in projects where Temporal's correctness model solves real bugs you've had.
2. Adopt it in new projects with the expectation that the polyfill cost disappears when native support arrives.
3. Keep date-fns in existing stable projects until native support makes migration worth the effort.

## The timezone trap

```javascript
// This is the bug you've shipped:
const date = new Date('2024-10-10');
console.log(date.getDate()); // might log 9 — wrong!

// New Date() with a date-only string parses as UTC midnight.
// getDate() returns local time. If you're UTC-5, midnight UTC is 7pm yesterday.
// getDate() returns the day before the one you typed.

// Temporal doesn't have this trap:
const date = Temporal.PlainDate.from('2024-10-10');
date.day; // 10. Always.
```

This bug has existed since 1995. It's in production code right now. It's the reason `date-fns/parseISO` exists.

Temporal's types are explicit about what they represent. A `PlainDate` is a date with no timezone. A `ZonedDateTime` has an explicit timezone. You cannot accidentally mix them.

`Date` was built in ten days. Given the constraints, it was fine. Given the thirty years since, it has not aged well.

Temporal is not a library. It's a correction. It has taken longer than it should have. The result is worth the wait.
