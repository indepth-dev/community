---
title: concat-all - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/concat-all
tags:
    -rxjs 
    -javascript 
    -reactive programming
---

# ConcatAll

## Diagram

<video>
    <source src="https://images.indepth.dev/references/rxjs/concat-all.mp4" type="video/mp4">
</video>

## Code

```javascript
const a = stream('a', 200, 3);
const b = stream('b', 200, 3);
const h = stream('h', 100, [a, b]);

h.pipe(concatAll()).subscribe(fullObserver(operator));
```
