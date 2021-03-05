---
title: delayWhen - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/delay-when
tags:
    -rxjs 
    -javascript 
    -reactive programming
---

# DelayWhen

## Diagram

<video>
    <source src="https://images.indepth.dev/references/rxjs/delay-when.mp4" type="video/mp4">
</video>

## Code

```javascript
const a = stream('a', 200, 3);
a.pipe(map(transform)).subscribe(fullObserver(operator));

function transform(v) {
  return v.split('-').shift();
}
```
