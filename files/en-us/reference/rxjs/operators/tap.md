---
name: tap
title: tap - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/tap
tags: rxjs, javascript, reactive programming
---

# tap

`tap` passes values along to the observer (mirroring) without modification while also executing a callback corresponding to the type of the value emitted by a source observable- next, error or complete notifications. It means that for normal emissions from the source will execute the `next` callback passed to `tap` as a parameter, if the source observable throws an error or sends a complete notification it will execute the corresponding error or complete callbacks passed to `tap`.

Because `tap` returns an observable that is identical to the source observable higher in the pipeline, nothing you do in the callback executed by `tap` will have any effect on the value that gets passed to an observer. **Because of this characteristic `tap` is most often used for running side effects or debugging purposes.**

Please note, that although logic inside the `tap` operator doesn’t effect the value, if an error occurs inside the `tap` operator handler, it breaks an observable chain and the error is sent down to the observer. Also, because `tap` is an operator, it requires a subscription to the source just like any other observable. Simply ending an observable chain with `tap` doesn’t trigger subscription to the source and hence no side effects inside `tap` will run.

`tap` operator works in the following way:

1. Subscribe to a source observable
2. When a new value arrives from a source observable, execute the `next` callback for the current value, disregard the returned value
3. Send the original value to the observer
4. If an error occurs inside the `next` callback, unsubscribe from the source, and send the error notification to the observer
5. Once the source observable completes, execute the `complete` callback, and send the complete notification to the observer
6. If the source observable throws an error, execute the `error` callback, and send the error notification to the observer

The following diagram demonstrates this sequence of steps:

<video>
    <source src="https://images.indepth.dev/references/rxjs/operators/tap.mp4" type="video/mp4">
</video>

## Usage
`tap` is mostly used for side-effects. Imagine you want to record all events happening inside an input and then later replay them. Here’s how you’d do it using `tap` operator:

```javascript
const input = document.createElement('input');
document.body.appendChild(input);

const events = new ReplaySubject();

fromEvent(input, 'keydown').pipe(
   tap((event: any) => {
       events.next(event);
   })
).subscribe();
```

## Playground

<iframe src="https://stackblitz.com/edit/indepth-rxjs-tap?embed=1&file=index.ts"></iframe>

## Additional resources

- [Official documentation](https://rxjs.dev/api/operators/tap)
