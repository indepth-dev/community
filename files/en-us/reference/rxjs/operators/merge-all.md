---
title: mergeAll - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/merge-all
tags:
    -rxjs 
    -javascript 
    -reactive programming
---

# mergeAll

## Diagram

<video>
    <source src="https://images.indepth.dev/references/rxjs/merge-all.mp4" type="video/mp4">
</video>

## Code

```javascript
const a = stream('a', 200, 3);
const b = stream('b', 200, 3);
const h = stream('h', 100, [a, b]);

h.pipe(mergeAll()).subscribe(fullObserver(operator));
```
