---
title: map - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/map
tags:
    -rxjs 
    -javascript 
    -reactive programming
---

# Map

## Diagram

<video>
    <source src="https://images.indepth.dev/references/rxjs/map.mp4" type="video/mp4">
</video>

## Code

```javascript
const a = stream('a', 200, 3);
a.pipe(map(transform)).subscribe(fullObserver(operator));

function transform(v) {
  return v.split('-').shift();
}
```
