---
title: switchAll - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/switch-all
tags:
    -rxjs 
    -javascript 
    -reactive programming
---

# switchAll

## Diagram

<video>
    <source src="https://images.indepth.dev/references/rxjs/operators/switch-all.mp4" type="video/mp4">
</video>

## Code

```javascript
const a = stream('a', 200, 3);
const b = stream('b', 200, 3);
const h = stream('h', 100, [a, b]);

h.pipe(switchAll()).subscribe(fullObserver('switchAll'));
```
