---
name: delay
title: delay - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/delay
tags: rxjs, javascript, reactive programming
---

# delay
`delay` time shifts each emitted value from the source observable by a defined time span or until a given date.  If the delay argument is a Number, this operator time shifts the source observable by that amount of time expressed in milliseconds. If the delay argument is a Date, `delay` starts emitting all incoming values until that point at the date specified preserving the intervals between the values.

Note that `delay` preserves the relative time intervals between the values. It means that if the source observable emits 3 items synchronously, all 3 of them will be delayed by a certain amount of time but delivered synchronously with a minimum time span between them. The time span will be equal or very close to the time span between the values emitted by the source obsevable. This is in contrast to how `interval` operator works, which inserts delays in between the values.

`delay` operator works in the following way:

1. Subscribe to a source observable
2. When a new value arrives from a source observable, add the value to internal cache and start a delay interval for that value
3. Once the interval elapses, send the corresponding value to the observer
4. Once the source observable completes, send the complete notification to the observer
5. If the source observable throws an error, send the error notification to the observer

The following diagram demonstrates this sequence of steps:

<video>
    <source src="https://images.indepth.dev/references/rxjs/operators/delay.mp4" type="video/mp4">
</video>

By default the operator uses [setInterval through AsyncScheduler](https://github.com/ReactiveX/rxjs/blob/9b708613cb7687647dc43c5e15b821e17ccc23ef/src/internal/operators/debounceTime.ts#L64) under the hood for scheduling.

## Usage
`delay` is commonly used to mock some asynchronous activity, such as network requests. 

Here’s an example that demonstrates how to simulate network latency with 3 requests:

```javascript
const api = 'https://reqres.in/api/users/';
const urls = [1, 2, 3].map(id => api + id);

from(urls).pipe(
   mergeMap(url => mockHTTPRequest(url))
).subscribe(val => console.log(val));

function mockHTTPRequest(url) {
   return of(`Response from ${url}`).pipe(
       // responses come in a random order
       delay(Math.random() * 1000)
   );
```

## Playground

<iframe src="https://stackblitz.com/edit/indepth-rxjs-delay?embed=1&file=index.ts"></iframe>

## Additional resources

- [Official documentation](https://rxjs.dev/api/operators/delay)

## See also

- [delayWhen](https://indepth.dev/reference/rxjs/operators/delay-when)
- [throttle](https://indepth.dev/reference/rxjs/operators/throttle)
- [throttleTime](https://indepth.dev/reference/rxjs/operators/throttle-time)
- [debounce](https://indepth.dev/reference/rxjs/operators/debounce)
- [debounceTime](https://indepth.dev/reference/rxjs/operators/debounce-time)
- [sampleTime](https://indepth.dev/reference/rxjs/operators/sample-time)
- [audit](https://indepth.dev/reference/rxjs/operators/audit)
- [auditTime](https://indepth.dev/reference/rxjs/operators/audit-time)
