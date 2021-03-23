---
title: switchAll - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/switch-all
tags: rxjs, javascript, reactive programming
---

# switchAll

`switchAll` takes a source observable that produces other observable streams. Such observable is called **higher-order observable** and the streams emitted as values are called **inner observables**. The operator subscribes to the most recently provided inner observable emitted by higher-order observable, unsubscribing from any previously subscribed inner observable.

`switchAll` has only one active subscription at a time from which the values are passed down to an observer. Once the higher-order observable emits a new inner observable stream, the operator switches the stream by unsubscribing from the current stream and subscribing to the newly emitted observable.

As values from any inner observable are produced, those values are emitted as part of the resulting sequence. Such process is often referred to as **flattening** in documentation.

The operator works in the following way:

1. Subscribe to a higher-order source observable
2. When a new inner observable is emitted from the higher-order observable, subscribe to it
3. When a new value arrives from this inner observable, pass it down to an observer
4. When a new inner observable is emitted from the higher-order observable, unsubscribe from the current inner observable
5. Subscribe to the new inner observable
6. Only after all source observables complete, send the complete notification to the observer.
7. If any of the inner source observables throws an error, send the error notification to the observer.

Please note that `switchAll` will never complete if some of the input streams don’t complete.

In the diagram below you can see the higher-order stream `H` that produces two inner streams `A` and `B`. The `switchAll` operator subscribes to the `A` stream first and passes values down to the observer. But as soon as the stream B is emitted from the source observable, `switchAll` switches to this stream `B` by unsubscribing from `A` and subscribing to `B`.

<video>
    <source src="https://images.indepth.dev/references/rxjs/operators/switch-all.mp4" type="video/mp4">
</video>

## Usage
Sometimes receiving values from all inner observable sequences is not what we need. In some scenarios, we may only be interested in the the values from the most recent inner sequence. A good example of such functionality is search. As a user types in some text, the request is sent to a server and since it’s asynchronous the result is returned as an observable. What if the user updates the text in the search-box before the result is returned? The second request is sent and so by now two searches have been sent to the server. However, the first search contains the results that we are no longer interested in. Furthermore, if the result for the first search was merged together with result for the second search, the user would be very surprised. We don’t want that and so this is where `switchAll` operator comes in. It subscribes and produces values only from the most recent inner sequence ignoring previous streams.

Here’s an example of using `switchAll` to switch between two streams:

```javascript
const a = interval(500).pipe(map((v) => 'a' + v), take(3));
const b = interval(500).pipe(map((v) => 'b' + v), take(3));
const higherOrderObservable = interval(1000).pipe(take(2), map(i => [a, b][i]));

higherOrderObservable.pipe(switchAll()).subscribe((value) => console.log(value));
```

## Playground

<iframe src="https://stackblitz.com/edit/indepth-rxjs-switch-all?embed=1&file=index.ts"></iframe>

## Additional resources

- [Official documentation](https://rxjs.dev/api/operators/switchAll)

## See also

- [merge](https://indepth.dev/reference/rxjs/operators/merge)
- [mergeAll](https://indepth.dev/reference/rxjs/operators/merge-all)
- [concat](https://indepth.dev/reference/rxjs/operators/concat)
- [concatAll](https://indepth.dev/reference/rxjs/operators/concat-all)
- [zip](https://indepth.dev/reference/rxjs/operators/zip)
- [withLatestFrom](https://indepth.dev/reference/rxjs/operators/with-latest-from)
