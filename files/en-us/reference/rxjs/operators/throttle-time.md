---
title: throttleTime - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/throttle-time
tags: rxjs, javascript, reactive programming
---

# throttleTime

`throttleTime` delays the values emitted by a source for the given due time. Similarly to [debounceTime](https://indepth.dev/reference/rxjs/operators/debounce-time), this operator can be used to control the rate with which the values are emitted to an observer. Unlike `debounceTime` though, `throttleTime` guarantees the values will be emitted regurarly, but not more often than the configured interval.

This operator takes an optional configuration parameter that can change the behavior of the operator: `{leading: boolean, trailing: boolean}`. The default settings are `{leading: true, trailing: false}`.

For the default configuration `{leading: true, trailing: false}` the operator works in the following way:

1. when new value arrives schedule a new interval
2. emit the value to the observer
3. if a new value comes before the interval ends, ignore it

The following diagram demonstrates this sequence of steps:

<video>
    <source src="https://images.indepth.dev/references/rxjs/operators/throttle-time-leading-true-trailing-false.mp4" type="video/mp4">
</video>

For the configuration `{leading: true, trailing: true}` the operator works in the following way:

1. when new value arrives schedule a new interval
2. emit the value to the observer
3. if a new value comes before the interval ends, keep it and override the existing one
4. once the interval ends, emit the value to the observer

The following diagram demonstrates this sequence of steps:

<video>
    <source src="https://images.indepth.dev/references/rxjs/operators/throttle-time-leading-true-trailing_true.mp4" type="video/mp4">
</video>

For the configuration `{leading: false, trailing: true}` the operator works in the following way:

1. when new value arrives schedule a new interval
2. keep the value
3. if a new value comes before the interval ends, keep it and override the existing one
4. once the interval ends, emit the value to the observer

The following diagram demonstrates this sequence of steps:

<video>
    <source src="https://images.indepth.dev/references/rxjs/operators/throttle-time-leading-false-trailing-true.mp4" type="video/mp4">
</video>

By default the operator uses `setInterval` [through AsyncScheduler under the hood](https://github.com/ReactiveX/rxjs/blob/9b708613cb7687647dc43c5e15b821e17ccc23ef/src/internal/operators/debounceTime.ts#L64) for scheduling.

## Usage
This operator is mostly used for events that can be triggered tens or even hundreds of times per second. The most common examples are DOM events such as scrolling, resizing, mouse movements, and keypress. 

For example, if you attach a scroll handler to an element, and scroll that element down to say 5000px, you’re likely to see 100+ events being fired. If your event handler performs multiple tasks (like heavy calculations and other DOM manipulation), you may see performance issues (jank). Reducing the number of times this handler is executed without disrupting user experience will have a significant effect on performance.

In addition, you should consider wrapping any interaction that triggers excessive calculations or API calls with a throttle.

Here’s an example of using `throttleTime` on an input:

```javascript
const inputElement = document.createElement('input');
document.body.appendChild(inputElement);

fromEvent(inputElement, 'input')
   .pipe(
       throttleTime(500),
       map((event: any) => event.target.value)
   ).subscribe(val => console.log(val));
```

## Playground

#### Leading - true, Trailing - false
<iframe src="https://stackblitz.com/edit/indepth-rxjs-throttletime-ltrue-tfalse?embed=1&file=index.ts"></iframe>

#### Leading - true, Trailing - true
<iframe src="https://stackblitz.com/edit/indepth-rxjs-throttletime-ltrue-ttrue?embed=1&file=index.ts"></iframe>

#### Leading - false, Trailing - true
<iframe src="https://stackblitz.com/edit/indepth-rxjs-throttletime-lfalse-ttrue?embed=1&file=index.ts"></iframe>

## Additional resources

- [Official documentation](https://rxjs.dev/api/operators/throttleTime)

## See also

- [throttle](https://indepth.dev/reference/rxjs/operators/throttle)
- [auditTime](https://indepth.dev/reference/rxjs/operators/audit-time)
- [sampleTime](https://indepth.dev/reference/rxjs/operators/sample-time)
- [debounce](https://indepth.dev/reference/rxjs/operators/debounce)
- [delay](https://indepth.dev/reference/rxjs/operators/delay)
