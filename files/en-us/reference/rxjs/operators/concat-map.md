---
name: concatMap
title: concatMap - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/concat-map
tags: rxjs, javascript, reactive programming
---

# concatMap

`concatMap` operator is basically a combination of two operators - concat and map. The map part lets you map a value from a source observable to an observable stream. Those streams are often referred to as inner streams. The concat part works like concatAll - it combines all inner observable streams returned from the map and sequentially emits all values from every input stream.

As values from any combined sequence are produced, those values are emitted as part of the resulting sequence. Such process is often referred to as flattening in documentation.

Use this operator if the order of emissions is important and you want to first see values emitted by streams that come through the operator first. 

The operator works in the following way:

1. Subscribe to a source observable.
2. When a new value arrives from a source observable, execute a `map` function that returns an inner observable.
3. Subscribe to this inner observable.
4. When this inner observable emits a value, pass it down to an observer.
5. When a new value arrives from a source observable, execute a `map` function that returns an inner observable.
6. Put this inner observable in a queue.
7. Once the current inner observable (refer step 3) completes, subscribe to the next inner observable in the queue (refer step 6).
8. Only after all inner observables complete, send the complete notification to the observer.
9. If any of the source observables throws an error, send the error notification to the observer.

The following diagram demonstrates this sequence of steps:

<video>
    <source src="https://images.indepth.dev/references/rxjs/operators/concat-map.mp4" type="video/mp4">
</video>

## Usage
This operator is mostly used when you need to map a plain value to an observable. A common use case is mapping URL strings into HTTP requests. As opposed to mergeMap, `concatMap` allows to execute all requests in a sequence - only once the previous request completes, a new request is initiated via subscription.

Here’s an example of using `concatMap` to do exactly this:

```javascript
const urls = [
   'https://api.mocki.io/v1/0350b5d5',
   'https://api.mocki.io/v1/ce5f60e2'
];

from(urls).pipe(
   concatMap((url) => {
       return fromFetch(url);
   })
).subscribe((response) => console.log(response.status));
```

## Playground

<iframe src="https://stackblitz.com/edit/indepth-rxjs-concat-map?embed=1&file=index.ts"></iframe>

## Additional resources

- [Official documentation](https://rxjs.dev/api/operators/concatMap)

## See also

- [concat](https://indepth.dev/reference/rxjs/operators/concat)
- [concatAll](https://indepth.dev/reference/rxjs/operators/concat-all)
- [merge](https://indepth.dev/reference/rxjs/operators/merge)
- [mergeMap](https://indepth.dev/reference/rxjs/operators/merge-map)
- [zip](https://indepth.dev/reference/rxjs/operators/zip)
- [withLatestFrom](https://indepth.dev/reference/rxjs/operators/with-latest-from)
