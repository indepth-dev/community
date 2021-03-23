---
title: switchMap - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/switch-map
tags: rxjs, javascript, reactive programming
---

# switchMap

`switchMap` operator is basically a combination of two operators - switchAll and map. The map part lets you map a value from a higher-order source observable to an inner observable stream. The switch part than works like `switchAll` - it subscribes to the most recently provided inner observable emitted by higher-order observable, unsubscribing from any previously subscribed inner observable.

`switchMap` has only one active subscription at a time from which the values are passed down to an observer. Once the higher-order observable emits a new value, `switchMap` executes the function to get a new inner observable stream and switches the streams. It unsubscribes from the current stream and subscribes to the new inner observable.

The operator works in the following way:

1. Subscribe to a higher-order source observable
2. When a new value arrives from a source observable, execute a map function that returns an inner observable
3. Subscribe to this inner observable
4. When a new value arrives from this inner observable, pass it down to an observer
5. When a new value arrives from a source observable, execute a map function that returns an inner observable
6. Unsubscribe from the current inner observable, subscribe to the new inner observable
7. Only after all source observables complete, send the complete notification to the observer.
8. If any of the inner source observables throws an error, send the error notification to the observer.

Please note that `switchAll` will never complete if some of the input streams don’t complete.

The following diagram demonstrates this sequence of steps:

<video>
    <source src="https://images.indepth.dev/references/rxjs/operators/switch-map.mp4" type="video/mp4">
</video>

## Usage

Sometimes receiving values from all inner observable sequences is not what we need. In some scenarios, we may only be interested in the the values from the most recent inner sequence. A good example of such functionality is search. As a user types in some text, the request is sent to a server and since it’s asynchronous the result is returned as an observable. What if the user updates the text in the search-box before the result is returned? The second request is sent and so by now two searches have been sent to the server. However, the first search contains the results that we are no longer interested in. Furthermore, if the result for the first search was merged together with result for the second search, the user would be very surprised. We don’t want that and so this is where `switchAll` operator comes in. It subscribes and produces values only from the most recent inner sequence ignoring previous streams.

Here’s an example of using `switchMap` to switch between two streams:

```javascript
const urls = [
   'https://api.mocki.io/v1/0350b5d5',
   'https://api.mocki.io/v1/ce5f60e2'
];

from(urls).pipe(
   switchMap((url) => {
       return fromFetch(url);
   })
).subscribe((response) => console.log(response.status));
```

## Playground

<iframe src="https://stackblitz.com/edit/indepth-rxjs-switch-map?embed=1&file=index.ts"></iframe>

## Additional resources

- [Official documentation](https://rxjs.dev/api/operators/switchMap)

## See also

- [concat](https://indepth.dev/reference/rxjs/operators/concat)
- [concatMap](https://indepth.dev/reference/rxjs/operators/concat-map)
- [merge](https://indepth.dev/reference/rxjs/operators/merge)
- [mergeMap](https://indepth.dev/reference/rxjs/operators/merge-map)
- [zip](https://indepth.dev/reference/rxjs/operators/zip)
- [withLatestFrom](https://indepth.dev/reference/rxjs/operators/with-latest-from)
