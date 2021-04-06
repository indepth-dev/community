---
name: bufferWhen
title: bufferWhen - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/buffer-when
tags: rxjs, javascript, reactive programming
---

# bufferWhen

`bufferWhen` collects values emitted from the source observable into cache without passing them down to an observer until the `notifier` observable emits (buffering). The buffer then sends the cached values as a group, resets and starts buffering again until the provided observable emits once more. 

This operator is very similar to [buffer](https://indepth.dev/reference/rxjs/operators/buffer), **but once the buffer is flushed, it unsubscribes from the notifier observable and will re-subscribe once a new value arrives.**

One interesting behavior of the `bufferWhen` operator is that it can send empty set of values to the observer if the `notifier` observable emits a value and the cache is empty.

The operator works in the following way:

1. Subscribe to a source observable
2. When a new value arrives from a source observable, put it in a cache
3. If there’s no active subscription to a notifier observable, subscribe to it
4. When a notifier observable emits a value, send all values from the cache to the observer, even if the cache is empty
5. Unsubscribe from the notifier observable
6. Once the source completes, send the complete notification to the observer
7. If any of the source observables throws an error, send the error notification to the observer.

The following diagram demonstrates this sequence of steps:

<video>
    <source src="https://images.indepth.dev/references/rxjs/operators/buffer-when.mp4" type="video/mp4">
</video>

## Usage
`bufferWhen` is often used when the batching technique is required. For example, sometimes you need to repeatedly perform a very expensive operation within a very small time frame, such as re-rendering a DOM tree on updates from a stream, batching can help collect updates and render them at once.

Here’s an example of using `bufferWhen` to emit array of most recent interval events on every click:

```javascript
const clicks = fromEvent(document, 'click');
const intervalEvents = interval(1000);

const buffered = intervalEvents.pipe(bufferWhen(() => clicks));

buffered.subscribe(x => console.log(x));
```

## Playground

<iframe src="https://stackblitz.com/edit/indepth-rxjs-buffer-when?embed=1&file=index.ts"></iframe>

## Additional resources

- [Official documentation](https://rxjs.dev/api/operators/bufferWhen)

## See also

- [buffer](https://indepth.dev/reference/rxjs/operators/buffer)
- [bufferTime](https://indepth.dev/reference/rxjs/operators/buffer-time)
