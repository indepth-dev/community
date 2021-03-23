---
title: sampleTime - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/sample-time
tags: rxjs, javascript, reactive programming
---

# sampleTime

`sampleTime` delays the values emitted by a source for the given due time. Similarly to [debounceTime](https://indepth.dev/reference/rxjs/operators/debounce-time), this operator can be used to control the rate with which the values are emitted to an observer. Unlike `debounceTime` though, `sampleTime` guarantees the values will be emitted regurarly, but not more often than the configured interval.

`sampleTime` is similar to [throttleTime](https://indepth.dev/reference/rxjs/operators/throttle-time) with the configuration `{leading: false, trailing: true}`. There are two significant differences:
- `sampleTime` keeps scheduling new intervals regardless if If the source observable emits new values or not, while `throttleTime` only schedules a new interval when new value comes from the source observable. This might have big effect on performance and hence `auditTime` is a preferred operator to be used for rate-limiting.
- If the source observable completes before the interval ends, `throttleTime` still sends the value to an observer, while `sampleTime` discards it.

The operator works in the following way:
1. schedule a new interval
2. when a new value arrives keep it and override the existing one
3. once the interval ends, emit the value to the observer
4. schedule a new interval
5. if the source observable completes before the interval ends, discard the current value

By default the operator uses `setInterval` through AsyncScheduler under the hood for scheduling.

The following diagram demonstrates this sequence of steps:

<video>
    <source src="https://images.indepth.dev/references/rxjs/operators/sample-time.mp4" type="video/mp4">
</video>

## Usage
When you are interested in ignoring values from a source observable for a given amount of time, you can use `sampleTime`. It is mostly used for events that can be triggered tens or even hundreds of times per second. The most common examples are DOM events such as scrolling, resizing, mouse movements, and keypress. 

For example, if you attach a scroll handler to an element, and scroll that element down to say 5000px, you’re likely to see 100+ events being fired. If your event handler performs multiple tasks (like heavy calculations and other DOM manipulation), you may see performance issues (jank). Reducing the number of times this handler is executed without disrupting user experience will have a significant effect on performance.

In addition, you should consider wrapping any interaction that triggers excessive calculations or API calls with a throttle.

Here’s an example of using `sampleTime` on an input:

```javascript
const inputElement = document.createElement('input');
document.body.appendChild(inputElement);

fromEvent(inputElement, 'input')
   .pipe(
       sampleTime(500),
       map((event: any) => event.target.value)
   ).subscribe(val => console.log(val));
```

## Playground

<iframe src="https://stackblitz.com/edit/indepth-rxjs-sampletime?embed=1&file=index.ts"></iframe>

## Additional resources

- [Official documentation](https://rxjs.dev/api/operators/sampleTime)

## See also

- [debounceTime](https://indepth.dev/reference/rxjs/operators/debounce-time)
- [auditTime](https://indepth.dev/reference/rxjs/operators/audit-time)
- [throttleTime](https://indepth.dev/reference/rxjs/operators/throttle-time)
- [delay](https://indepth.dev/reference/rxjs/operators/delay)
