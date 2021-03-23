---
title: exhaust - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/exhaust
tags:
    -rxjs 
    -javascript 
    -reactive programming
---

# exhaust

## Diagram

<video>
    <source src="https://images.indepth.dev/references/rxjs/operators/exhaust.mp4" type="video/mp4">
</video>

## Code

```javascript
const a = stream('a', 200, 3);
const b = stream('b', 200, 3);
const h = stream('h', 1000, [a, b]);

h.pipe(exhaust()).subscribe(fullObserver(operator));
```
