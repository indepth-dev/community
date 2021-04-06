---
name: retryWhen
title: retryWhen - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/retry-when
tags: rxjs, javascript, reactive programming
---

# retryWhen

Just as a plain observable `retryWhen` passes values along to the observer (mirroring) until an error in the source stream occurs. When that happens, the operator executes a callback function passing in the stream that will be emitting the errors starting with the current one that has just occurred. Regardless of the number of errors and retries, the callback function is only executed once.

The function is expected to return an observable that will be used as a guide to notify the operator when to retry the subscription to the original source observable. The operator subscribes to the returned guiding observable only once and doesn’t unsubscribe on future errors or until the original source observable completes. If that guiding observable calls complete or error then the `retryWhen` will send complete or error notification to the observer. Any other value emitted from the guiding observable will cause the operator to resubscribe to the original source observer.

`retryWhen` operator works in the following way:

1. Subscribe to a source observable
2. When a new value arrives from a source observable, send the value to the observer
3. If the source observable throws an error, execute a callback function and subscribe to the returned guiding observable
4. When a new value arrives from the guiding observable, re-subscribe to the original source
5. If  the guiding observable completes or throws an error, send the complete or error notification to the observer
6. Once the source observable completes, send the complete notification to the observer

The following diagram demonstrates this sequence of steps:

<video>
    <source src="https://images.indepth.dev/references/rxjs/operators/retry-when.mp4" type="video/mp4">
</video>

## Usage
Errors in programs happen all the time and RxJS is no exception. For example, a network request can fail because of numerous reasons. It’s always useful to have a mechanism to retry an action. 

For simple retries of network requests the [retry](https://indepth.dev/reference/rxjs/operators/retry) operator is preferred. But when you need to retry based on some criteria, for example, an error description, you need to use `retryWhen`.

Here’s an example of using `retryWhen` to retry the request using information about the error:

```javascript
const url = 'https://i.imgur.com/fHyEMsl.jpg';

fromFetch(url).pipe(
   switchMap((response) => response.json()),
   retryWhen((errors) => {
           return errors.pipe(
               takeWhile((error) => {
                   if (error instanceof SyntaxError) {
                       throw new Error(error.message);
                   }

                   return true;
               })
           );
       }
   )
).subscribe();
```

## Playground

<iframe src="https://stackblitz.com/edit/indepth-rxjs-retry-when?embed=1&file=index.ts"></iframe>

## Additional resources

- [Official documentation](https://rxjs.dev/api/operators/retryWhen)

## See also

- [retry](https://indepth.dev/reference/rxjs/operators/retry)
- [catchError](https://indepth.dev/reference/rxjs/operators/catch-error)
