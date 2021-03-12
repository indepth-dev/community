---
title: debounceTime - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/debounce-time
tags:
    -rxjs 
    -javascript 
    -reactive programming
---

# debounceTime

## Diagram

<video>
    <source src="https://images.indepth.dev/references/rxjs/debounce-time.mp4" type="video/mp4">
</video>

## Code

```javascript
// 0 1 2 3 last value is always emitted
const a = stream('a', [0, 600, 600, 300, 800], 5); // 0, 1, 3, 4
const b = stream('b', [0, 300, 300, 600, 300], 5); // 2, 4

a.pipe(debounceTime(500)).subscribe(fullObserver(operator));
/*const debouncedA = a.pipe(debounceTime(500));
const debouncedB = b.pipe(debounceTime(500));
from([debouncedA, debouncedB]).pipe(concatAll()).subscribe(fullObserver(operator));*/
```
