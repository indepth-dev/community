---
name: mergeAll
title: mergeAll - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/merge-all
tags: rxjs, javascript, reactive programming
---

# mergeAll
`mergeAll` combines a number of inner observable streams and concurrently emits all values from every input stream. It’s similar to [merge](https://indepth.dev/reference/rxjs/operators/merge), but instead of taking a set of streams directly as input, it takes an observable source that produces other streams (observables). Those streams are often referred to as **inner** stream (observable) and the stream that emits them is known as **higher-order observable**.

As values from any combined sequence are produced, those values are emitted as part of the resulting sequence. Such process is often referred to as **flattening** in documentation.

Use this operator if you’re not concerned with the order of emissions and is simply interested in all values coming out from multiple combined streams as if they were produced by one stream.

The operator works in the following way:
1. Subscribe to the higher-order source observable
2. When a new inner observable is emitted from the source observable, subscribe to it
3. When a new value arrives from an inner souce observable, pass it down to an observer
4. Only after all inner source observables complete, send the complete notification to the observer.
5. If any of the inner source observables throws an error, send the error notification to the observer.

In the diagram below you can see the `H` higher-order stream that produces two inner streams `A` and `B`. The `mergeAll` operator combines values from these two streams and then passes them through to the resulting sequence as they occur:

<video>
    <source src="https://images.indepth.dev/references/rxjs/operators/merge-all.mp4" type="video/mp4">
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

<iframe src="https://stackblitz.com/edit/indepth-rxjs-merge-all?embed=1&file=index.ts"></iframe>

## Additional resources

- [Official documentation](https://rxjs.dev/api/operators/mergeAll)

## See also

- [merge](https://indepth.dev/reference/rxjs/operators/merge)
- [mergeMap](https://indepth.dev/reference/rxjs/operators/merge-map)
- [concat](https://indepth.dev/reference/rxjs/operators/concat)
- [zip](https://indepth.dev/reference/rxjs/operators/zip)
- [withLatestFrom](https://indepth.dev/reference/rxjs/operators/with-latest-from)
