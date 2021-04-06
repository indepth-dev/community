---
name: takeUntil
title: takeUntil - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/take-until
tags: rxjs, javascript, reactive programming
---

# takeUntil

`takeUntil` passes values from the source observable to the observer (mirroring) until a provided observable known as the notifier emits its first value. The operator subscribes to the source observable and begins mirroring the source Observable. It also subscribes to the notifier observable. If the notifier emits a value, `takeUntil` unsubscribes from the source and notifier observables and completes. If the notifier doesn't emit any value and completes then `takeUntil` will pass all values.

Note, that if you only need to take a specific number of values from the source observable, you should use [take](https://indepth.dev/reference/rxjs/operators/take) operator.

The operator works in the following way:

1. Subscribe to both a source and notifier observables
2. When a new value arrives from a source observable, send it to the observer
3. If the notifier observable emits a value, unsubscribe from both observables, send the complete notification to the observer
4. Once the source observable completes, send the complete notification to the observer
5. If the source observable throws an error, send the error notification to the observer

The following diagram demonstrates this sequence of steps:

<video>
    <source src="https://images.indepth.dev/references/rxjs/operators/take-until.mp4" type="video/mp4">
</video>

## Usage
`takeUntil` is mostly used to avoid memory leaks and clear resources once a certain even happens. In a component based application, `takeUntil` can be triggered by the event of a component destruction through a notifier observable. This notifier observable is commonly implemented as a [Subject](https://indepth.dev/reference/rxjs/subjects/subject).

Here’s an example that demonstrates this logic:

```javascript
const stream = interval(1000);

function component(source) {
   const notifier = new Subject();

   source.pipe(
       takeUntil(notifier)
   ).subscribe(render);

   return () => {
       notifier.next(null);
   };
}

const destroy = component(stream);

setTimeout(destroy, 3000);

function render(v) {
   const text = document.createTextNode(v);
   document.body.appendChild(text);
}
```

## Playground

<iframe src="https://stackblitz.com/edit/indepth-rxjs-take-until?embed=1&file=index.ts"></iframe>

## Additional resources

- [Official documentation](https://rxjs.dev/api/operators/takeUntil)

## See also

- [take](https://indepth.dev/reference/rxjs/operators/take)
- [takeWhile](https://indepth.dev/reference/rxjs/operators/take-while)
