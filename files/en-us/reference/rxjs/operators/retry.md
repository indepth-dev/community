---
title: retry - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/retry
tags: rxjs, javascript, reactive programming
---

# retry

Just as a plain observable `retry` passes values along to the observer (mirroring) until an error in the source stream occurs. When that happens, the operator will resubscribe to the source observable instead of propagating the error to the observer. The number of times this resubscription will occur on an error is defined by the `count` parameter passed to the operator.

Please note, that the observer will receive all items emitted by the source observable, even those emitted before the error occurred. For example, if the source observable emits values `{1,2}`, then fails, then resubscription happens and the stream successfully emits values `{1,2,3,4}` before completing, the observer receives the following sequence of values `{1,2,1,2,3,4}`.

`retry` operator works in the following way:

1. Subscribe to a source observable
2. When a new value arrives from a source observable, send the value to the observer
3. If the source observable throws an error, compare the number of defined retries with the number of occurred resubscriptions
4. If the number of subscriptions is less, resubscribe, otherwise send the error notification to the observer
5. Once the source observable completes, send the complete notification to the observer

The following diagram demonstrates this sequence of steps:

<video>
    <source src="https://images.indepth.dev/references/rxjs/operators/retry.mp4" type="video/mp4">
</video>

## Usage
Errors in programs happen all the time and RxJS is no exception. For example, a network request can fail because of numerous reasons. It’s always useful to have a mechanism to retry an action. 

`retry` can be used to retry a failed network request. But when you need to retry based on some criteria, for example, an error description, you need to use [retryWhen](https://indepth.dev/reference/rxjs/operators/retry-when).

Here’s an example of using `retry` to retry the request twice in case of an error:

```javascript
const url = 'https://i.imgur.com/fHyEMsl.jpg';

fromFetch(url).pipe(
   switchMap((response) => response.json()),
   // triggers 3 network requests:
   // 1 initial and 2 retries
   retry(2)
).subscribe();
```

## Playground

<iframe src="https://stackblitz.com/edit/indepth-rxjs-retry?embed=1&file=index.ts"></iframe>

## Additional resources

- [Official documentation](https://rxjs.dev/api/operators/retry)

## See also

- [retryWhen](https://indepth.dev/reference/rxjs/operators/retry-when)
- [catchError](https://indepth.dev/reference/rxjs/operators/catch-error)
