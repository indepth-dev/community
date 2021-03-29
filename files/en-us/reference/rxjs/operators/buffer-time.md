---
title: bufferTime - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/buffer-time
tags: rxjs, javascript, reactive programming
---

# bufferTime

`bufferTime` collects values emitted from the source observable into cache without passing them down to an observer until the specified time interval elapses (buffering). The buffer then sends the cached values as a group, resets and starts buffering again until time interval elapses once again.

This operator is similar to [buffer](https://indepth.dev/reference/rxjs/operators/buffer), but instead of using an observable for indication when to flush the buffer, it uses a time interval.

One interesting behavior of the `bufferTime` operator is that it can send an empty set of values to the observer if the time interval elapses while the cache is empty.

The operator works in the following way:

1. Subscribe to a source observable
2. Start a time span interval
3. When a new value arrives from a source observable, put it in a cache
4. When an interval ends, send all values from the cache to the observer, even if the cache is empty
5. Continue from step 2
6. Once the source completes, send the complete notification to the observer
7. If the source observable throws an error, send the error notification to the observer.

The following diagram demonstrates this sequence of steps:

<video>
    <source src="https://images.indepth.dev/references/rxjs/operators/buffer-time.mp4">
</video>

**There’s also a special case of this operator that allows ignoring certain values.** To do that, the operator must be used with **the creation interval parameter** like this:

```javascript
const bufferCreationInterval = 1000;
const bufferTimeSpan = 500;
interval(200).pipe(
   take(10),
   // this setup will ignore values 2, 3, 7, 8
   bufferTime(bufferTimeSpan, bufferCreationInterval)
).subscribe((v) => console.log(v));
```

With the creation interval, the operator works in the following way:

1. Subscribe to a source observable
2. Start both a time span and a creation intervals
3. When a new value arrives from a source observable, put it in a cache if the time span interval is running, ignore otherwise 
4. When a time span interval ends, send all values from the cache to the observer, even if the cache is empty
5. When a creation interval ends, continue from step 2
6. Once the source completes, send the complete notification to the observer
7. If the source observable throws an error, send the error notification to the observer.

The following diagram demonstrates this sequence of steps:

<video>
    <source src="https://images.indepth.dev/references/rxjs/operators/buffer-time-plus-creation-interval.mp4">
</video>

## Usage
`bufferTime` is often used when the batching technique is required. For example, sometimes you need to repeatedly perform a very expensive operation within a very small time frame, such as re-rendering a DOM tree on updates from a stream, batching can help collect updates and render them at once.

Here’s an example of using `bufferTime` to emit array of the most recent clicks:

```javascript
const clicks = fromEvent(document, 'click');
const buffered = clicks.pipe(bufferTime(1000));
buffered.subscribe(x => console.log(x));
```

## Playground

<iframe src="https://stackblitz.com/edit/indepth-rxjs-buffer-time?embed=1&file=index.ts"></iframe>

## Additional resources

- [Official documentation](https://rxjs.dev/api/operators/bufferTime)

## See also

- [buffer](https://indepth.dev/reference/rxjs/operators/buffer)
- [bufferWhen](https://indepth.dev/reference/rxjs/operators/buffer-when)
