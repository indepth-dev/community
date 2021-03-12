---
title: concatMap - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/concat-map
tags:
    -rxjs 
    -javascript 
    -reactive programming
---

# concatMap

## Diagram

<video>
    <source src="https://images.indepth.dev/references/rxjs/concat-map.mp4" type="video/mp4">
</video>

## Code

```javascript
const h = stream('h', 100, ['a', 'b']);

h.pipe(
  concatMap((name) => {
    return stream(name, 200, 3);
  })
).subscribe(fullObserver(operator));
```
