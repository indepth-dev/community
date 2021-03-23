---
title: concat - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/concat
tags: rxjs, javascript, reactive programming
---

# concat
`concat` combines a number of observables streams and **sequentially** emits all values from every given input stream. It only has one active subscription at a time from which the values are passed down to an observer. Once the current active stream completes it subscribes to next observable in a sequence. As values from any combined sequence are produced, those values are emitted as part of the resulting sequence. Such process is often referred to as **flattening** in documentation.

The operator works in the following way:
1. Subscribe to the first source observable in a sequence
2. When a new value arrives from a source observable, pass it down to an observer
3. Once this source observable completes, subscribe to the next source observable in the sequence
4. Only after all source observables complete, send the complete notification to the observer.
5. If any of the source observables throws an error, send the error notification to the observer.

Please note that `concat` will never complete if some of the input streams don’t complete. This also means that some streams will never be subscribed to.

In the diagram below you can see the concat operator combining two streams `A` and `B` each producing 3 items and the values falling through to the resulting sequence first from the `A` and then from the `B`:

<video>
    <source src="https://images.indepth.dev/references/rxjs/operators/concat.mp4" type="video/mp4">
</video>

## Usage
Use this operator if the order of emissions is important and you want to first see values emitted by streams that you pass first to the operator. For example, you may have an observable sequences that delivers values from a cache and another sequence that delivers values from a remote server. Use `concat` if you want to combine them and ensure that the value from cache is delivered first.

Here’s an example of using `concat` on two streams:

```javascript
const a = interval(500).pipe(map((v) => 'a' + v), take(3));
const b = interval(500).pipe(map((v) => 'b' + v), take(3));

concat(a, b).subscribe((value) => console.log(value));
```

## Playground

<iframe src="https://stackblitz.com/edit/indepth-rxjs-concat?embed=1&file=index.ts"></iframe>

## Additional resources

- [Official documentation](https://rxjs.dev/api/operators/concat)

## See also

- [concatAll](https://indepth.dev/reference/rxjs/operators/concat-all)
- [concatMap](https://indepth.dev/reference/rxjs/operators/concat-map)
- [merge](https://indepth.dev/reference/rxjs/operators/merge)
- [zip](https://indepth.dev/reference/rxjs/operators/zip)
- [withLatestFrom](https://indepth.dev/reference/rxjs/operators/with-latest-from)
