---
title: mergeAll - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/merge-all
tags:
    -rxjs 
    -javascript 
    -reactive programming
---

# MergeAll

## Description

This operator combines all emitted inner streams and just as with plain merge concurrently produces values from each
stream. In the diagram below you can see the H higher-order stream that produces two inner streams A and B. The mergeAll
operator combines values from these two streams and then passes them through to the resulting sequence as they occur.

<a href="/">Test Link</a>

**[indetpth.dev](https://indepth.dev)**

## Code

```javascript
const a = stream('a', 200, 3);
const b = stream('b', 200, 3);
const h = stream('h', 100, [a, b]);

h.pipe(mergeAll()).subscribe(fullObserver(operator));
```

## Diagram

<video>
    <source src="https://images.indepth.dev/references/rxjs/throttle-time-leading-false.mp4" type="video/mp4">
</video>






