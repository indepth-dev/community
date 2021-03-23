---
title: mergeMap - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/merge-map
tags: rxjs, javascript, reactive programming
---

# mergeMap

`mergeMap` operator is basically a combination of two operators - [merge](https://indepth.dev/reference/rxjs/operators/merge) and [map](https://indepth.dev/reference/rxjs/operators/map). The map part lets you map a value from a source observable to an observable stream. Those streams are often referred to as **inner** streams. The merge part than works like [mergeAll](https://indepth.dev/reference/rxjs/operators/merge-all) - it combines all inner observable streams returned from the map and concurrently emits all values from every input stream.

As values from any combined sequence are produced, those values are emitted as part of the resulting sequence. Such process is often referred to as **flattening** in documentation.

Use this operator if you’re not concerned with the order of emissions and is simply interested in all values coming out from multiple combined streams as if they were produced by one stream.

The operator works in the following way:
1. Subscribe to a source observable
2. When a new value arrives from a source observable, execute a `map` function that returns an inner observable
3. Subscribe to this inner observable
4. When this inner observable emits a value, pass it down to an observer
5. Only after all inner observables complete, send the complete notification to the observer.
6. If any of the source observables throws an error, send the error notification to the observer.

The following diagram demonstrates this sequence of steps:

<video>
    <source src="https://images.indepth.dev/references/rxjs/operators/merge-map.mp4" type="video/mp4">
</video>

## Usage
This operator is mostly used when you need to map a plain value to an observable. A common use case is mapping URL strings into HTTP requests.

Here’s an example of using `mergeMap` to do exactly this:

```javascript
const urls = [
   'https://api.mocki.io/v1/0350b5d5',
   'https://api.mocki.io/v1/ce5f60e2'
];

from(urls).pipe(
   mergeMap((url) => {
       return fromFetch(url);
   })
).subscribe((response) => console.log(response.status));
```

## Playground

<iframe src="https://stackblitz.com/edit/indepth-rxjs-merge-map?embed=1&file=index.ts"></iframe>

## Additional resources

- [Official documentation](https://rxjs.dev/api/operators/mergeMap)

## See also

- [merge](https://indepth.dev/reference/rxjs/operators/merge)
- [mergeAll](https://indepth.dev/reference/rxjs/operators/merge-all)
- [concat](https://indepth.dev/reference/rxjs/operators/concat)
- [zip](https://indepth.dev/reference/rxjs/operators/zip)
- [withLatestFrom](https://indepth.dev/reference/rxjs/operators/with-latest-from)
