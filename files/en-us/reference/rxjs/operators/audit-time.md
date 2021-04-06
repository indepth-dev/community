---
name: auditTime
title: auditTime - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/audit-time
tags: rxjs, javascript, reactive programming
---

# auditTime

`auditTime` delays the values emitted by a source observable for the given due time. Similarly to [debounceTime](https://indepth.dev/reference/rxjs/operators/debounce-time), this operator can be used to control the rate with which the values are emitted to an observer. Unlike `debounceTime` though, `throttleTime` guarantees the values will be emitted regurarly, but not more often than the configured interval.

`auditTime` is very similar to throttleTime with the configuration `{leading: false, trailing: true}`. There’s one difference regarding the handling of the trailing value. If the source observable completes before the interval ends, `throttleTime` still sends the value to an observer, while `auditTime` discards it.

The operator works in the following way:
- when a new value arrives schedule a new interval
- keep the value
- if a new value comes before the interval ends, keep it and override the existing one
- once the interval ends, emit the value to the observer
- if the source observable completes before the interval ends, discard the value

By default the operator uses `setInterval` [through AsyncScheduler under the hood](https://github.com/ReactiveX/rxjs/blob/9b708613cb7687647dc43c5e15b821e17ccc23ef/src/internal/operators/debounceTime.ts#L64) for scheduling.

The following diagram demonstrates this sequence of steps:

<video>
    <source src="https://images.indepth.dev/references/rxjs/operators/audit-time.mp4" type="video/mp4">
</video>

## Usage
When you are interested in ignoring values from a source observable for a given amount of time, you can use auditTime. It is mostly used for events that can be triggered tens or even hundreds of times per second. The most common examples are DOM events such as scrolling, resizing, mouse movements, and keypress. 

For example, if you attach a scroll handler to an element, and scroll that element down to say 5000px, you’re likely to see 100+ events being fired. If your event handler performs multiple tasks (like heavy calculations and other DOM manipulation), you may see performance issues (jank). Reducing the number of times this handler is executed without disrupting user experience will have a significant effect on performance.

In addition, you should consider wrapping any interaction that triggers excessive calculations or API calls with a throttle.

Here’s an example of using `auditTime` on an input:

```javascript
const inputElement = document.createElement('input');
document.body.appendChild(inputElement);

fromEvent(inputElement, 'input')
   .pipe(
       auditTime(500),
       map((event: any) => event.target.value)
   ).subscribe(val => console.log(val));
```

## Playground

<iframe src="https://stackblitz.com/edit/indepth-rxjs-audittime?embed=1&file=index.ts"></iframe>

## Additional resources

- [Official documentation](https://rxjs.dev/api/operators/auditTime)

## See also

- [debounceTime](https://indepth.dev/reference/rxjs/operators/debounce-time)
- [sampleTime](https://indepth.dev/reference/rxjs/operators/sample-time)
- [throttleTime](https://indepth.dev/reference/rxjs/operators/throttle-time)
- [delay](https://indepth.dev/reference/rxjs/operators/delay)

