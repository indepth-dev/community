---
title: debounceTime - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/debounce-time
tags:
    -rxjs 
    -javascript 
    -reactive programming
---

# debounceTime

## About

`debounceTime` delays the values emitted by a source for the given due time. If within this time a new value arrives, the previous pending value is dropped and the timer is reset. In this way `debounceTime` keeps track of most recent value and emits that most recent value when the given due time is passed.

The operator works in the following way:
1. when new value arrives schedule a new interval
2. remember the value and discard old ones if exists
3. when the interval ends, emit the value
4. if new value comes before the interval ends, run step 1 again

By default it uses `setInterval` [through AsyncScheduler](https://github.com/ReactiveX/rxjs/blob/9b708613cb7687647dc43c5e15b821e17ccc23ef/src/internal/operators/debounceTime.ts#L64) under the hood for scheduling.

The following diagram demonstrates this sequence of steps:

<video>
    <source src="https://images.indepth.dev/references/rxjs/debounce-time.mp4" type="video/mp4">
</video>

## Usage

This operator is mostly used for events that can be triggered tens or even hundreds of times per second. The most common examples are DOM events such as scrolling, mouse movement, and keypress. When using `debouceTime` you only care about the final state. For example, current scroll position when a user stops scrolling, or a final text in a searchbox after a user stops typing in characters. In effect, using the operator allows grouping multiple sequential events into a single one and hence execute the callback just once. This can greatly improve performance.

Common scenarios for a debounce are resize, scroll, and keyup/keydown events. In addition, you should consider wrapping any interaction that triggers excessive calculations or API calls with a debounce.

Here’s an example of using `debounceTime` on an input::

```javascript
const inputElement = document.createElement('input');
document.body.appendChild(inputElement);

fromEvent(inputElement, 'input')
   .pipe(
       debounceTime(500),
       map((event: any) => event.target.value)
   ).subscribe(val => console.log(val));
```

## Playground

<iframe src="https://stackblitz.com/edit/indepth-rxjs-debouncetime?embed=1&file=index.ts"></iframe>

## Additional resources

[Official documentation](https://rxjs-dev.firebaseapp.com/api/operators/debounceTime)

[How to debounce an input while skipping the first entry](https://indepth.dev/posts/1444/how-to-debounce-an-input-while-skipping-the-first-entry)

[Rx.js Operators, Part II](https://indepth.dev/posts/1445/rx-js-operators-part-ii)

## See alsoo

[debounce](https://indepth.dev/reference/rxjs/operators/debounce)

[auditTime](https://indepth.dev/reference/rxjs/operators/audit-time)

[sampleTime](https://indepth.dev/reference/rxjs/operators/sample-time)

[throttleTime](https://indepth.dev/reference/rxjs/operators/throttle-time)

[delay](https://indepth.dev/reference/rxjs/operators/delay)
