---
name: buffer
title: buffer - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/buffer
tags: rxjs, javascript, reactive programming
---

# buffer

`buffer` collects values emitted from the source observable into cache without passing them down to an observer until the `notifier` observable emits (buffering). The buffer then sends the cached values as a group, resets and starts buffering again until the provided observable emits once more.

One interesting behavior of the buffer operator is that it can send empty set of values to the observer if the `notifier` observable emits a value and the cache is empty.

The operator works in the following way:

1. Subscribe to a source observable
2. Subscribe to a notifier observable
3. When a new value arrives from a source observable, put it in a cache
4. When a notifier observable emits a value, send all values from the cache to the observer, even if the cache is empty
5. Once the source or the notifier observable completes, send the complete notification to the observer
6. If any of the source observables throws an error, send the error notification to the observer.

`buffer` completes if either source observable or notifier observable completes.

The following diagram demonstrates this sequence of steps:

<video>
    <source src="https://images.indepth.dev/references/rxjs/operators/buffer.mp4" type="video/mp4">
</video>

## Usage
`buffer` is often used when the batching technique is required. For example, sometimes you need to repeatedly perform a very expensive operation within a very small time frame, such as re-rendering a DOM tree on updates from a stream, batching can help collect updates and render them at once.

Here’s an example of using `buffer` to emit array of most recent interval events on every click:

```javascript
const clicks = fromEvent(document, 'click');
const intervalEvents = interval(1000);

const buffered = intervalEvents.pipe(buffer(clicks));

buffered.subscribe(x => console.log(x));
```

## Playground

<iframe src="https://stackblitz.com/edit/indepth-rxjs-buffer?embed=1&file=index.ts"></iframe>

## Additional resources

- [Official documentation](https://rxjs.dev/api/operators/buffer)

## See also

- [bufferTime](https://indepth.dev/reference/rxjs/operators/buffer-time)
- [bufferWhen](https://indepth.dev/reference/rxjs/operators/buffer-when)
