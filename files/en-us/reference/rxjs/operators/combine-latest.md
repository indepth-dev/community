---
name: combineLatest
title: combineLatest - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/combine-latest
tags: rxjs, javascript, reactive programming
---

# combineLatest

`combineLatest` allows to merge several streams by taking the most recent value from each input observable and emitting those values to the observer as a combined output (usually as an array). The operators caches the last value for each input observable and **only** once all input observables produced at least one value it emits the combined cached values to the observer.

The resulting stream completes when all inner streams complete and will throw an error if any of the inner streams throws an error. It will never complete if any of the inner streams doesn’t complete. On the other hand, if any stream does not emit value but completes, resulting stream will complete at the same moment without emitting anything, since it will be now impossible to include value from completed input stream in resulting sequence. Also, if some input stream does not emit any value and never completes, `combineLatest` will also never emit and never complete, since, again, it will wait for all streams to emit some value.

The operator works in the following way:

1. Subscribe to all input observables
2. When a source observable emits a value, override any existing value in the cache for this observable
3. If here’s a cached value for each obsevable in the cache, emit the combined values to the observer 
4. Only after all source observables complete, send the complete notification to the observer.
5. If any of the source observables throws an error, send the error notification to the observer.

In the diagram below you can see the `combineLatest` operator combining two streams `A` and `B`. As soon as all streams have emitted at least one value each new emission produces a combined value through the result stream:

<video>
    <source src="https://images.indepth.dev/references/rxjs/operators/combine-latest.mp4" type="video/mp4">
</video>

Here is the code example that demonstrates the setup shown by the above diagram:

```javascript
const a = stream('a', 200, 3);
const b = stream('b', 500, 3);

combineLatest(a, b).subscribe(fullObserver(operator));
```

And the editable demo:

<iframe src="https://stackblitz.com/edit/indepth-rxjs-combine-latest?embed=1&file=index.ts"></iframe>

## Usage

This operator can be useful if you need to evaluate some combination of state which needs to be kept up-to-date when part of the state changes. A simple example would be a monitoring system. Each service is represented by a sequence that returns a `Boolean` indicating the availability of a said service. The monitoring status is green if all services are available so the projection function should simply perform a logical `AND`.

## Additional resources

- [Official documentation](https://rxjs.dev/api/operators/combineLatest)

## See also

- [concat](https://indepth.dev/reference/rxjs/operators/concat)
- [concatAll](https://indepth.dev/reference/rxjs/operators/concat-all)
- [merge](https://indepth.dev/reference/rxjs/operators/merge)
- [mergeAll](https://indepth.dev/reference/rxjs/operators/merge-all)
- [zip](https://indepth.dev/reference/rxjs/operators/zip)
- [withLatestFrom](https://indepth.dev/reference/rxjs/operators/with-latest-from)

