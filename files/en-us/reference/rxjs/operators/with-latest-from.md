---
name: withLatestFrom
title: withLatestFrom - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/with-latest-from
tags: rxjs, javascript, reactive programming
---

# withLatestFrom

`withLatestFrom` combines the source observable with other streams and emits values calculated from the latest values of each, only when the source emits. While the similar combineLatest operator emits a new value whenever there’s a new emission from any of the input streams, `withLatestFrom` emits a new value only if there’s a new emission from the guiding stream.

Just as with `combineLatest` it still waits for at least one emitted value from each stream and may complete without a single emission when the guiding stream completes. It will never complete if the guiding stream doesn’t complete and will throw an error if any of the inner streams errors out.

The operator works in the following way:

1. Subscribe to the guiding stream
2. Subscribe to all other input observables
3. When a source observable emits a value, override the value in the cache corresponding to this observable
4. When a guiding stream emits a value, emit the values from each source observable as a group
5. When a guiding stream completes, send the complete notification to the observer
6. If any of the source observables throws an error, send the error notification to the observer.

In the diagram below you can see the `withLatestFrom` operator combining two streams `A` and `B` with the stream `B` being the guiding stream. Every time the stream `B` emits a new value the resulting sequence produces a combined value using latest value from the stream `A`:

<video>
    <source src="https://images.indepth.dev/references/rxjs/operators/with-latest-from.mp4" type="video/mp4">
</video>

Here is the code example that demonstrates the setup shown by the above diagram:

```javascript
const a = stream('a', 500, 3);
const b = stream('b', 2000, 3);

b.pipe(withLatestFrom(a)).subscribe(fullObserver(operator));
```

And the editable demo:

<iframe src="https://stackblitz.com/edit/indepth-rxjs-with-latest-from?embed=1&file=index.ts"></iframe>

## Usage

This operator cab ne used when you have one stream that drives emissions but also need to combine its values with latest values from other streams.

## Additional resources

- [Official documentation](https://rxjs-dev.firebaseapp.com/api/operators/withLatestFrom)

## See also

- [zip](https://indepth.dev/reference/rxjs/operators/zip)
- [withLatestFrom](https://indepth.dev/reference/rxjs/operators/with-latest-from)
- [concat](https://indepth.dev/reference/rxjs/operators/concat)
- [concatAll](https://indepth.dev/reference/rxjs/operators/concat-all)
- [merge](https://indepth.dev/reference/rxjs/operators/merge)
- [mergeAll](https://indepth.dev/reference/rxjs/operators/merge-all)
