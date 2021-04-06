---
name: exhaust
title: exhaust - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/exhaust
tags: rxjs, javascript, reactive programming
---

# exhaust
`exhaust` takes a source observable that produces other observable streams. Such observable is called **higher-order observable** and the streams emitted as values are called **inner observables**. As values from inner sequences are produced, those values are emitted as part of the resulting sequence to an observer. Such process is often referred to as **flattening** in documentation.

`exhaust` only has one active subscription at a time from which the values are passed down to an observer. When the higher-order observable emits a new inner observable stream, if the current stream hasn’t completed, this new inner observable is dropped. Once the current active stream completes, the operator waits for another inner observable to subscribe ignoring previous inner observables.

The operator works in the following way:

1. Subscribe to a higher-order source observable
2. When a new inner observable is emitted from the higher-order observable, subscribe to it
3. When a new value arrives from this inner observable, pass it down to an observer
4. When a new inner observable is emitted from the higher-order observable, check if there’s any active subscription
5. If there’s an active subscription, ignore the observable, otherwise subscribe to it
6. Only after a higher-order source observable and current active subscriptions complete, send the complete notification to the observer.
7. If any of the inner source observables throws an error, send the error notification to the observer.

Please note that `exhaust` will never complete if some of the input streams don’t complete. This also means that some streams will never be subscribed to.

In the diagram below you can see the `H` higher-order stream that produces two inner streams `A` and `B`. The `exhaust` operator subscribes to the inner stream `A` and ignores `B` while `A` is active. Once `A` completes and `C` arrives, it subscribes to `C` and waits for it to complete:

<video>
    <source src="https://images.indepth.dev/references/rxjs/operators/exhaust.mp4" type="video/mp4">
</video>

## Usage
`exhaust` is useful when your system can produce multiple actions that trigger a long lasting task, like a login HTTP request, and all other similar actions should be ignored until the current task is complete.

Here’s an example of using `exhaust` to do exactly this:

```javascript
const request = (index) => timer(500).pipe(take(1), mapTo({index}));

const a = request(1);
const b = request(2);
const actions = interval(100).pipe(take(2), map((v, i) => [a, b][i]));

// logs 1
actions.pipe(exhaust()).subscribe((res: any) => console.log(res.index));
```

## Playground

<iframe src="https://stackblitz.com/edit/indepth-rxjs-exhaust?embed=1&file=index.ts"></iframe>

## Additional resources

- [Official documentation](https://rxjs-dev.firebaseapp.com/api/operators/exhaust)

## See also

- [exhaustMap](https://indepth.dev/reference/rxjs/operators/exhaust-map)
- [mergeAll](https://indepth.dev/reference/rxjs/operators/merge-all)
- [concatAll](https://indepth.dev/reference/rxjs/operators/concat-all)
- [zip](https://indepth.dev/reference/rxjs/operators/zip)
- [withLatestFrom](https://indepth.dev/reference/rxjs/operators/with-latest-from)
