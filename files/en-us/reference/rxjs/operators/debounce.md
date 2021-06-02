---
name: debounce
title: debounce - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/debounce
tags: rxjs, javascript, reactive programming  
---

# debounce

`debounce` delays the values emitted by a source until the duration Observable emits a value or completes. If within this time a new value arrives, the previous pending value is dropped and the duration Observable is re-subscribed. In this way `debounce` keeps track of the most recent value and emits that most recent value using the duration observable as an indicator of where to do that.

The operator works in the following way:
1. when new value arrives, execute a function to get the duration Observable
2. subscribe to the duration Observable
3. keep the value and discard old ones if exists
4. when the duration Observable emits a value or completes, pass the stored value to the observer
5. if new value comes before the duration Observable emits or complets, run step 1 again

The following diagram demonstrates this sequence of steps:

<video>
    <source src="https://images.indepth.dev/references/rxjs/operators/debounce.mp4" type="video/mp4">
</video>

## Usage

This operator is mostly used for events that can be triggered tens or even hundreds of times per second. The most common examples are DOM events such as scrolling, mouse movement, and keypress. When using `debouce` you only care about the final state. For example, current scroll position when a user stops scrolling, or a final text in a searchbox after a user stops typing in characters. In effect, using the operator allows grouping multiple sequential events into a single one and hence execute the callback just once. This can greatly improve performance.

Common scenarios for a debounce are resize, scroll, and keyup/keydown events. In addition, you should consider wrapping any interaction that triggers excessive calculations or API calls with a debounce.

Here’s an example of using `debounce` on an input:

```javascript
const inputElement = document.createElement('input');
document.body.appendChild(inputElement);

fromEvent(inputElement, 'input')
   .pipe(
       debounce(() => interval(500)),
       map((event: any) => event.target.value)
   ).subscribe(val => console.log(val));
```

## Playground

<iframe src="https://stackblitz.com/edit/indepth-rxjs-debounce?embed=1&file=index.ts"></iframe>

## Additional resources

- [Official documentation](https://rxjs-dev.firebaseapp.com/api/operators/debounce)
- [How to debounce an input while skipping the first entry](https://indepth.dev/posts/1444/how-to-debounce-an-input-while-skipping-the-first-entry)
- [Rx.js Operators, Part II](https://indepth.dev/posts/1445/rx-js-operators-part-ii)

## See also

- [debounceTime](https://indepth.dev/reference/rxjs/operators/debounce-time)
- [auditTime](https://indepth.dev/reference/rxjs/operators/audit-time)
- [sampleTime](https://indepth.dev/reference/rxjs/operators/sample-time)
- [throttleTime](https://indepth.dev/reference/rxjs/operators/throttle-time)
- [delay](https://indepth.dev/reference/rxjs/operators/delay)
