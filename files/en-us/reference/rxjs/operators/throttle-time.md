---
title: throttleTime - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/throttle-time
tags:
    -rxjs 
    -javascript 
    -reactive programming
---

# throttleTime

## Diagram

<video>
    <source src="https://images.indepth.dev/references/rxjs/throttle-time-trailing-false.mp4" type="video/mp4">
</video>

## Code

```javascript
const defaultThrottleConfig = {leading: true, trailing: false};

// a-0 is leading in the first interval,
// a-1 is trailing in the first interval
// a-2 is leading in the second interval
const a = stream('a', 250, 3);

// a-0, a-2; trailing a-1 is skipped
a.pipe(throttleTime(500, undefined, defaultThrottleConfig)).subscribe(fullObserver(operator));
```
