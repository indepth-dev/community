---
name: take
title: take - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/take
tags: rxjs, javascript, reactive programming
---

# take

`take` passes values from the source observable to the observer (mirroring) until it reaches the threshold of values set for the operator.

On each value `take` compares the number of values it has mirrored with the defined threshold. If the threshold is reached, it completes the stream by unsubscribing from the source and passing the complete notification to the observer.

`take` operator works in the following way:

1. Subscribe to a source observable
2. When a new value arrives from a source observable, send the value to the observer
3. Increase the counter and compare it with the defined threshold
4. If these numbers are equal, unsubscribe from the source observable and send the complete notification to the observer
5. Once the source observable completes, send the complete notification to the observer
6. If the source observable throws an error, send the error notification to the observer

The following diagram demonstrates this sequence of steps:

<video>
    <source src="https://images.indepth.dev/references/rxjs/operators/take.mp4" type="video/mp4">
</video>

## Usage
A common use case for the `take` operator is to query a storage that is an observable. For example, when you have a global store implemented with observables, like NgRx or Redux-observable, you will need to get part of that state in an imperative way. 

Here’s how you can do it with `take` operator:

```javascript
const store = new BehaviorSubject(1);

// since store here is implemented as a Subject and doesn't complete,
// and if we don't use `take(1)`, the subscription is kept forever
store.pipe(take(1)).subscribe((v) => console.log(v));
```

## Playground

<iframe src="https://stackblitz.com/edit/indepth-rxjs-take-1?embed=1&file=index.ts"></iframe>

## Additional resources

- [Official documentation](https://rxjs.dev/api/operators/take)

## See also

- [takeUntil](https://indepth.dev/reference/rxjs/operators/take-until)
- [takeWhile](https://indepth.dev/reference/rxjs/operators/take-while)
