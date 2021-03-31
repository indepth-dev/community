---
title: delayWhen - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/delay-when
tags: rxjs, javascript, reactive programming
---

# delayWhen

`delayWhen` delays each emitted value from the source observable by a time span determined by another observable referred to as duration observable. When the source observable emits a value, the operator executes the callback with the value as an argument and expects the function to return a duration observable. `delayWhen` keeps the value emitted from the source observable in a cache until the delay observable emits a value. When that happens, it sends the delayed value to the observer.

It’s important to note that `delayWhen` executes the callback function to get a duration observable **on every new value that comes from the source observable**. It means that it keeps a registry of all duration observables for all values emitted from the source observable. Once a duration observable emits a value and the corresponding value from the source observable is sent to the observer the operator unsubscribes from this duration observable.

Optionally, `delayWhen` takes a second argument known as subscriptionDelay, which is an observable that can be used to delay the subscription the source observable. When subscriptionDelay emits its first value or completes, the operator subscribes to the source and starts re-emitting values with a delay. If subscriptionDelay is not provided, delayWhen will subscribe to the source observable as soon as the output observable is subscribed to.

`delayWhen` operator works in the following way:

1. Subscribe to a source observable
2. When a new value arrives from a source observable, execute the callback function to get the duration observable corresponding to this value
3. Add the value and the corresponding duration observable to internal cache
4. When a duration observable emits a value, send the corresponding value to the observer, unsubscribe from this duration observable and remove it from cache
5. Once the source observable completes, send the complete notification to the observer
6. If the source observable throws an error, send the error notification to the observer

The following diagram demonstrates this sequence of steps:

<video>
    <source src="https://images.indepth.dev/references/rxjs/operators/delay-when.mp4" type="video/mp4">
</video>

## Usage
`delayWhen` can be used when you need to delay a value until some other action takes place. For example, when you login a user and return the token, you may need to perform an action on the token before returning it to the calling method. 

Here’s an example that demonstrates how wait until the token is added to the local storage before sending it to the observer:

```javascript
login().pipe(
   delayWhen((user) => setToLocalStorage(user))
).subscribe((user) => console.log(user));

function login() {
   return fromFetch('https://reqres.in/api/users/2').pipe(
       switchMap((response) => response.json())
   );
}

function setToLocalStorage(value) {
   return of(value).pipe(
       tap((value) => {
           localStorage.setItem('token', JSON.stringify(value));
       })
   );
}
```

Adding a token to the local storage is just an example. You can have any synchronous or an asynchronous action instead.

## Playground

<iframe src="https://stackblitz.com/edit/indepth-rxjs-delay-when?embed=1&file=index.ts"></iframe>

## Additional resources

- [Official documentation](https://rxjs.dev/api/operators/delayWhen)

## See also

- [delay](https://indepth.dev/reference/rxjs/operators/delay)
- [throttle](https://indepth.dev/reference/rxjs/operators/throttle)
- [throttleTime](https://indepth.dev/reference/rxjs/operators/throttle-time)
- [debounce](https://indepth.dev/reference/rxjs/operators/debounce)
- [debounceTime](https://indepth.dev/reference/rxjs/operators/debounce-time)
- [sampleTime](https://indepth.dev/reference/rxjs/operators/sample-time)
- [audit](https://indepth.dev/reference/rxjs/operators/audit)
- [auditTime](https://indepth.dev/reference/rxjs/operators/audit-time)
