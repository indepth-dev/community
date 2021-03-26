---
title: exhaustMap - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/exhaust
tags: rxjs, javascript, reactive programming
---

# exhaustMap
`exhaustMap` operator is basically a combination of two operators - exhaust and map. The map part lets you map a value from a higher-order source observable to an inner observable stream. The exhaust part than subscribes to an inner observable and passes values down to an observer if there’s no active subscription already, otherwise it just ignores new inner observables.

`exhaustMap` has only one active subscription at a time from which the values are passed down to an observer. When the higher-order observable emits a new inner observable stream, if the current stream hasn’t completed, this new inner observable is dropped. Once the current active stream completes, the operator waits for another inner observable to subscribe ignoring previous inner observables.

The operator works in the following way:

1. Subscribe to a higher-order source observable
2. When a new inner observable arrives from a source observable, check if there’s no active subscription
3. If there’s an active subscription, ignore the observable
4. If there’s no active subscription, execute a map function that returns an inner observable and subscribe to it
5. When a new value arrives from this inner observable, pass it down to an observer
6. Only after a higher-order source observable and current active subscriptions complete, send the complete notification to the observer.
7. If any of the inner source observables throws an error, send the error notification to the observer.

Please note that `exhaustMap` will never complete if some input streams don't complete. This also means that some streams will never be subscribed to.

The following diagram demonstrates this sequence of steps:

<video>
    <source src="https://images.indepth.dev/references/rxjs/operators/exhaust-map.mp4" type="video/mp4">
</video>

## Usage
`exhaustMap` is useful when your system can produce multiple actions that trigger a long lasting task, like a login HTTP request, and all other similar actions should be ignored until the current task is complete.

Here’s an example of using `exhaustMap` to do exactly this:

```javascript
const request = (index) => timer(500).pipe(take(1), mapTo({index}));

const a = request(1);
const b = request(2);
const actions = interval(100).pipe(take(2), map((v, i) => [a, b][i]));

// logs 1
actions.pipe(exhaustMap((request) => request)).subscribe((res: any) => console.log(res.index));
```

## Playground

<iframe src="https://stackblitz.com/edit/indepth-rxjs-switch-exhaust-map?embed=1&file=index.ts"></iframe>

## Additional resources

- [Official documentation](https://rxjs-dev.firebaseapp.com/api/operators/exhaustMap)

## See also

- [exhaust](https://indepth.dev/reference/rxjs/operators/exhaust)
- [mergeAll](https://indepth.dev/reference/rxjs/operators/merge-all)
- [concatAll](https://indepth.dev/reference/rxjs/operators/concat-all)
- [zip](https://indepth.dev/reference/rxjs/operators/zip)
- [withLatestFrom](https://indepth.dev/reference/rxjs/operators/with-latest-from)
