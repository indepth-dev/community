---
name: zip
title: zip - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/zip
tags: rxjs, javascript, reactive programming
---

# zip

`zip` operator combines observable streams in a way that resembles the mechanics of a zipper on clothing or a bag. It brings together two or more sequences of corresponding values as a tuple (a pair in case of two input streams). It waits for the corresponding value to be emitted from all input streams, then transforms them into a single value using a projection function and emits the result. It will only emit the value once it has a pair of fresh values from each source sequence. So if one of the source observables emits values faster than the other sequence, the rate of publishing will be dictated by the slower of the two sequences.

The resulting stream completes when any of the inner streams complete and the corresponding matched pairs are emitted from other streams. It will never complete if any of the inner streams doesn’t complete and will throw an error if any of the inner streams errors out.

The operator works in the following way:

1. Subscribe to all input observables
2. When a source observable emits a value, add to the cache for this source observable under certain index
3. If there’s a cached value for each obsevable in the cache for this index, emit the combined values to the observer 
4. Only after all source observables complete, send the complete notification to the observer.
5. If any of the source observables throws an error, send the error notification to the observer.

In the diagram below you can see the `zip` operator combining two streams `A` and `B`. As soon as a corresponding pair is matched the resulting sequence produces a combined value:

<video>
    <source src="https://images.indepth.dev/references/rxjs/operators/zip.mp4" type="video/mp4">
</video>

Here is the code example that demonstrates the setup shown by the above diagram:

```javascript
const a = stream('a', 200, 3);
const b = stream('b', 500, 3);

combineLatest(a, b).subscribe(fullObserver(operator));
```

And the editable demo:

<iframe src="https://stackblitz.com/edit/indepth-rxjs-zip?embed=1&file=index.ts"></iframe>

## Additional resources

- [Official documentation](https://rxjs.dev/api/operators/zip)

## See also

- [concat](https://indepth.dev/reference/rxjs/operators/concat)
- [concatAll](https://indepth.dev/reference/rxjs/operators/concat-all)
- [merge](https://indepth.dev/reference/rxjs/operators/merge)
- [mergeAll](https://indepth.dev/reference/rxjs/operators/merge-all)
- [combineLatest](https://indepth.dev/reference/rxjs/operators/combine-latest)
- [withLatestFrom](https://indepth.dev/reference/rxjs/operators/with-latest-from)
