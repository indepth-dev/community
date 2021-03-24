---
title: race - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/race
tags: rxjs, javascript, reactive programming
---

# race
`race` is used to select an observable sequences that is the first to produce values. As soon as one of the sequences starts emitting values the other sequences are unsubscribed and completely ignored.

The resulting stream completes when the selected input stream completes and will throw an error if this one stream errors out. It will also never complete if this inner stream doesn’t complete.

The operator works in the following way:

1. Subscribe to all source observables
2. When a new value arrives from a source observable, pass it down to an observer
3. Unsubscribe from all other source observables
4. After the main source observables completes, send the complete notification to the observer.
5. If the source observables throws an error, send the error notification to the observer.

In the diagram below you can see the race operator combining two streams `A` and `B` each producing 3 items but only the values from the stream `A` are emitted since this stream starts emitting values first.

<video>
    <source src="https://images.indepth.dev/references/rxjs/operators/race.mp4" type="video/mp4">
</video>

## Usage
This operator can be useful if you have multiple resources that can provide values, for example, servers around the world, but due to network conditions the latency is not predictable and varies significantly. Using this operator you can send the same request out to multiple data sources and consume the result of the first that responds.

Here’s an example of using `race` to run multiple HTTP requests and cancel one of them:

```javascript
const a = fromFetch('https://api.mocki.io/v1/0350b5d5');
const b = fromFetch('https://api.mocki.io/v1/ce5f60e2');

race(a, b).subscribe((response) => console.log(response));
```

## Playground

<iframe src="https://stackblitz.com/edit/indepth-rxjs-race?embed=1&file=index.ts"></iframe>

## Additional resources

- [Official documentation](https://rxjs.dev/api/operators/race)

## See also

- [merge](https://indepth.dev/reference/rxjs/operators/merge)
- [concat](https://indepth.dev/reference/rxjs/operators/concat)
- [zip](https://indepth.dev/reference/rxjs/operators/zip)
- [withLatestFrom](https://indepth.dev/reference/rxjs/operators/with-latest-from)
