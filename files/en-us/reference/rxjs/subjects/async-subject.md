---
name: AsyncSubject
title: AsyncSubject - RxJS Reference | indepth.dev
slug: reference/rxjs/subjects/async-subject
tags: rxjs, javascript, reactive programming
---

# AsyncSubject

`AsyncSubject` is a variant of a [Subject](https://indepth.dev/reference/rxjs/subjects) which keeps the last value emitted by a source observable before completion and sends it to all new subscriptions. `AsyncSubject` needs to wait until the source observable completes before identifying the current value as the latest and only then emit it to existing or future subscribers.

This behavior means that you can always directly get the last emitted value from the `AsyncSubject` even if the subscriber subscribes much later than the value was stored.

In a way, this `AsyncSubject` is similar to how [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) works. The biggest difference between the two is that a Promise is always eager, meaning it will execute the function you pass to it immediately. With `AsyncSubject`, however, you can control when you want to subscribe to a source observable. And because all observables are lazy, only when you subscribe the producer function in the source observable will be executed.

In case of an error on the source observable, `AsyncSubject` will not emit the latest value to subscriptions. Instead, it will simply pass along the error notification from the source Observable to new subscriptions.

`AsyncSubject` works in the following way:

1. Create an internal subscriptions container
2. When a new subscription occurs, add it to the container
3. When a source observable emits a new value or the method `next` is called on the subject, set the emitted value as the current latest value overriding any existing value if any
4. If the source observable completes or the method `complete` is called on the subject, set the state of the subject to `stopped` and send the current latest value alongside the complete notification to all existing subscriptions; remove the subscriptions from the container
5. If the source observable throws an error or the method `error` is called on the subject, set the state of the subject to `stopped` and set the `error` notification as the current value; send the `error` notification to all existing subscriptions; remove the subscriptions from the container
6. If the subject is `stopped`, do not add any new subscriptions to the container. Instead, if the subject hasn’t been errored, send the current latest alongside complete or notification immediately on subscription to the corresponding observer, otherwise send the error notification without the latest vlue
7. If the `stopped` subject is subscribed to a new source observable, ignore the values from this source

The following diagram demonstrates this sequence of steps:

<video>
    <source src="https://images.indepth.dev/references/rxjs/subjects/async-subject.mp4">
</video>

## Usage
`AsyncSubject` is a great choice if you need to fetch and cache (one-shot) resources. When making a network request we’re generally interested in the final output, which is a response. And to get it we need to wait until the request is fully loaded at which point an observable stream completes.

Here’s an example that demonstrates that scenario:

```javascript
const cache = {};

function getResource(url) {
   if (!cache[url]) {
       cache[url] = new AsyncSubject();
       fetch(url)
           .then((response) => response.json())
           .then((data) => {
               cache[url].next(data);
               cache[url].complete();
           });
   }

   return cache[url].asObservable();
}

const url = 'https://api.mocki.io/v1/ce5f60e2';

getResource(url).subscribe((data) => console.log(data));

setTimeout(() => {
   // no request is made, data is served from the AsyncSubject's cache
   getResource(url).subscribe((data) => console.log(data));
}, 3000);
```

## Playground

<iframe src="https://stackblitz.com/edit/indepth-rxjs-async-subject?embed=1&file=index.ts"></iframe>

## Additional resources

- [Official documentation](https://rxjs-dev.firebaseapp.com/api/index/class/AsyncSubject)

## See also

- [ReplaySubject](https://indepth.dev/reference/rxjs/subjects/replay-subject)
- [BehaviorSubject](https://indepth.dev/reference/rxjs/subjects/behavior-subject)
