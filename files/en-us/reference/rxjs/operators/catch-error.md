---
name: catchError
title: catchError - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/catch-error
tags: rxjs, javascript, reactive programming
---

# catchError

Just as a plain observable `catchError` passes values along to the observer until an error in the source stream occurs. When that happens, the operator executes a callback function passing in the error. The function is expected to return an observable that will replace the original source observable. 

This replacement observable is then going to be subscribed to and its values are going to be used in place of the errored out input observable. If an error occurs in the replacement observable, the operator will send the error notification to the observer.

Since the callback function takes the source observable that produced the error as an argument alongside the error, it is possible to **retry** that observable by returning the original observable again.

The operator works in the following way:

1. Subscribe to a source observable
2. When a new value arrives from a source observable, send it to the observer
3. If the source observable throws an error, execute a callback function and subscribe to the returned replacement observable
4. When a new value arrives from the replacement observable, send it to the observer
5. Once the replacement observable completes, send the complete notification to the observer
6. If the replacement observable throws an error, send the error notification to the observer

The following diagram demonstrates this sequence of steps:

<video>
    <source src="https://images.indepth.dev/references/rxjs/operators/catch-error.mp4" type="video/mp4">
</video>

## Usage
Errors in programs happen all the time and RxJS is no exception. For example, a network request can fail because of numerous reasons. Since you can pass an error handler as part of the observer when subscribing to the observable, that’s a one way to handle an error gracefully. And sometimes that’s all we need, but this error handling approach is limited. 

Using error handling inside an observer, we cannot, for example, recover from the error or emit an alternative fallback value that replaces the value that we were expecting from the backend. That’s where `catchError` operator comes in handy.

Here’s an example of using `catchError` to try a different URL if the original fails:

```javascript
const server1 = 'http://url-to-fail.com';
const server2 = 'https://api.mocki.io/v1/b043df5a';

fromFetch(server1).pipe(
   catchError((err) => fromFetch(server2))
).subscribe((res) => res.status);
```

## Playground

<iframe src="https://stackblitz.com/edit/indepth-rxjs-catch-error?embed=1&file=index.ts"></iframe>

## Additional resources

- [Official documentation](https://rxjs.dev/api/operators/catchError)

## See also

- [retry](https://indepth.dev/reference/rxjs/operators/retry)
- [retryWhen](https://indepth.dev/reference/rxjs/operators/retry-when)
