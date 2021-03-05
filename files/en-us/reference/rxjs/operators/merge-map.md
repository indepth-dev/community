---
title: mergeMap - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/merge-map
tags:
    -rxjs 
    -javascript 
    -reactive programming
---

# MergeMap

## Diagram

<video>
    <source src="https://images.indepth.dev/references/rxjs/merge-map.mp4" type="video/mp4">
</video>

## Code

```javascript
const h = stream('h', 100, ['a', 'b']);

h.pipe(
  mergeMap((name) => {
    return stream(name, 200, 3);
  })
).subscribe(fullObserver(operator));
```
