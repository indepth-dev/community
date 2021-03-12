---
title: concat - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/concat
tags:
    -rxjs 
    -javascript 
    -reactive programming
---

# concat

## Diagram

<video>
    <source src="https://images.indepth.dev/references/rxjs/concat.mp4" type="video/mp4">
</video>

## Code

```javascript
const a = stream('a', 200, 3);
const b = stream('b', 200, 3);

concat(a, b).subscribe(fullObserver(operator));
```
