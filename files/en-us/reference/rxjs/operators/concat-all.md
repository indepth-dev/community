---
title: concatAll - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/concat-all
tags: rxjs, javascript, reactive programming
---

# concatAll

`concatAll` combines a number of inner observable streams and **sequentially** emits all values from every given input stream. It is similar to [concat](https://indepth.dev/reference/rxjs/operators/concat), but instead of taking a set of streams directly as input, it takes an observable source that produces other streams (observables). Those streams are often referred to as **inner** streams.

The operator only has one active subscription at a time from which the values are passed down to an observer. Once the current active stream completes it subscribes to next observable in a sequence. As values from any combined sequence are produced, those values are emitted as part of the resulting sequence. Such process is often referred to as **flattening** in documentation.

The operator works in the following way:

1. Subscribe to a higher-order source observable
2. When a new inner observable is emitted from the source observable, subscribe to it
3. When a new value arrives from this inner source observable, pass it down to an observer
4. When a new inner observable is emitted from the source observable, put it in a queue
5. Once the current source observable completes, subscribe to the next source observable in the queue
6. Only after all source observables complete, send the complete notification to the observer.
7. If any of the inner source observables throws an error, send the error notification to the observer.

Please note that `concatAll` will never complete if some of the input streams don’t complete. This also means that some streams will never be subscribed to.

In the diagram below you can see the `H` higher-order stream that produces two inner streams `A` and `B`. The `concatAll` operator combines values from these two streams and then passes them through to the resulting sequence as they occur. As opposed to [mergeAll](https://indepth.dev/reference/rxjs/operators/merge-all), it subscribes to those inner observables sequentially. First it subscribes to `A`, waits until it’s completed, and only then subscribes to `B`.

The following diagram demonstrates this behavior:

<video>
    <source src="https://images.indepth.dev/references/rxjs/operators/concat-all.mp4" type="video/mp4">
</video>

## Usage
Use this operator if the order of emissions is important and you want to first see values emitted by streams that come through the operator first. For example, you may have an observable sequences that delivers values from a cache and another sequence that delivers values from a remote server. Use `concatAll` if you want to combine them and ensure that the value from cache is delivered first.

Here’s an example of using `concatAll` to combine two streams:

```javascript
const a = interval(500).pipe(map((v) => 'a' + v), take(3));
const b = interval(500).pipe(map((v) => 'b' + v), take(3));
const higherOrderObservable = of(a, b);

higherOrderObservable.pipe(concatAll()).subscribe((value) => console.log(value));
```

## Playground

<iframe src="https://stackblitz.com/edit/indepth-rxjs-concat-all?embed=1&file=index.ts"></iframe>

## Additional resources

- [Official documentation](https://rxjs.dev/api/operators/concatAll)

## See also

- [concat](https://indepth.dev/reference/rxjs/operators/concat)
- [concatMap](https://indepth.dev/reference/rxjs/operators/concat-map)
- [merge](https://indepth.dev/reference/rxjs/operators/merge)
- [mergeMap](https://indepth.dev/reference/rxjs/operators/merge-map)
- [zip](https://indepth.dev/reference/rxjs/operators/zip)
- [withLatestFrom](https://indepth.dev/reference/rxjs/operators/with-latest-from)
