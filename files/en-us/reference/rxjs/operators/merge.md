---
name: merge
title: merge - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/merge
tags: rxjs, javascript, reactive programming
---

# merge

`merge` combines a number of observables streams and concurrently emits all values from every given input stream. As values from any combined sequence are produced, those values are emitted as part of the resulting sequence. Such process is often referred to as flattening in documentation.

Use this operator if you’re not concerned with the order of emissions and is simply interested in all values coming out from multiple combined streams as if they were produced by one stream.

The operator works in the following way:
1. Subscribe to all source observables
2. When a new value arrives from a source observable, pass it down to an observer
3. Only after all source observables complete, send the complete notification to the observer.
4. If any of the source observables throws an error, send the error notification to the observer.

In the diagram below you can see the merge operator combining two streams `A` and `B` each producing 3 items and the values falling through to the resulting sequence as they occur:

<video>
    <source src="https://images.indepth.dev/references/rxjs/operators/merge.mp4" type="video/mp4">
</video>

## Usage
Use this operator if you’re not concerned with the order of emissions and is simply interested in all values coming out from multiple combined streams as if they were produced by one stream.

Here’s an example of using `merge` on multiple streams:

```javascript
const a = interval(500).pipe(map((v) => 'a' + v), take(3));
const b = interval(500).pipe(map((v) => 'b' + v), take(3));
merge(a, b).subscribe((value) => console.log(value));
```

## Playground

<iframe src="https://stackblitz.com/edit/indepth-rxjs-merge?embed=1&file=index.ts"></iframe>

## Additional resources

- [Official documentation](https://rxjs.dev/api/operators/merge)

## See also

- [mergeAll](https://indepth.dev/reference/rxjs/operators/merge-all)
- [mergeMap](https://indepth.dev/reference/rxjs/operators/merge-map)
- [concat](https://indepth.dev/reference/rxjs/operators/concat)
- [zip](https://indepth.dev/reference/rxjs/operators/zip)
- [withLatestFrom](https://indepth.dev/reference/rxjs/operators/with-latest-from)
