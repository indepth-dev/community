---
title: merge - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/merge
tags:
    -rxjs 
    -javascript 
    -reactive programming
---

# Merge

## Diagram

<video>
    <source src="https://images.indepth.dev/references/rxjs/merge.mp4" type="video/mp4">
</video>

## Code

```javascript
const a = stream('a', 200, 3);
const b = stream('b', 200, 3);

merge(a, b).subscribe(fullObserver(operator));
```
