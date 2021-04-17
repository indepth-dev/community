---
name: behaviorSubject
title: BehaviorSubject - RxJS Reference | indepth.dev
slug: reference/rxjs/subjects/behaviour-subject
tags: rxjs, javascript, reactive programming
---

# BehaviorSubject

`BehaviorSubject` is a variant of a [Subject](https://indepth.dev/reference/rxjs/subjects) which has a notion of the current value that it stores and emits to all new subscriptions. This current value is either the item most recently emitted by the source observable or a seed/default value if none has yet been emitted. Since there must always be a current value, `BehaviorSubject` requires an initial value during initialization. If you want the last emitted value(s) on subscription, but do not want to supply a seed value, check out [ReplaySubject](https://indepth.dev/reference/rxjs/subjects/replay-subject) instead.

This behavior means that you can always directly get the last emitted value from the `BehaviorSubject` even if the subscriber subscribes much later than the value was stored. When an observer subscribes to a `BehaviorSubject`, it begins by first emitting the current value and then continues to emit any other items emitted by the source Observable(s) after the subscription. However, as opposed to the [ReplaySubject], if the `BehaviorSubject` is in the stopped state, it will only emit `COMPLETE` or `ERROR` notification, whereas [ReplaySubject] in the stopped state still replays the values before sending those notifications. Subjects typically switch to the stopped state when the observable they are subscribed to completes or produces an error.

In case of an error on the source observable, `BehaviorSubject` will not emit any items to subsequent subscriptions. Instead, it will simply pass along the error notification from the source Observable to new subscriptions.

`BehaviorSubject` works in the following way:

1. Create an internal subscriptions container
2. Set the current value kept by the subject to the initial value passed as an argument during instantiation
3. When a new subscription occurs, add it to the container and emit the current value to the corresponding observer
4. When a source observable emits a new value or the method `next` is called on the subject, update the current value with the emitted value and send it to observers for all subscriptions
5. If the source observable completes or the method `complete` is called on the subject, set the state of the subject to `stopped` and current value to the complete notification; send the complete notification to all existing subscriptions, remove them from the container
6. If the source observable throws an error or the method `error` is called on the subject, set the state of the subject to `stopped` and current value to the `error` notification; send the `error` notification to all existing subscriptions, remove them from the container
7. If the subject is `stopped`, do not add any new subscriptions to the container and instead send the current value (complete or error notification) immediately on subscription to the corresponding observer.
8. If the `stopped` subject is subscribed to a new source observable, ignore the values from this source

The following diagram demonstrates this sequence of steps:

<video>
    <source src="https://images.indepth.dev/references/rxjs/subjects/behavior-subject.mp4">
</video>


## Usage
A most common use case for `BehaviorSubject` is to act as a store or a cache that subscribers can read the latest value when they need it. Here’s an example that demonstrates a basic implementation of a store:

```javascript
const state1 = {name: 'James', age: 33};
const state2 = {name: 'Anna', age: 27};

const store = new BehaviorSubject(state1);

const v1 = getValueFromStore();
console.log(v1.name); // 'James'

updateStore(state2);

const v2 = getValueFromStore();
console.log(v2.name); // 'Anna'

selectFromStore((state) => state.age).subscribe((v) => console.log(v));

function updateStore(v) {
   store.next(v);
}

function getValueFromStore() {
   return store.value;
}

function selectFromStore(selector) {
   return store.asObservable().pipe(
       map(selector)
   );
}
```

## Playground

<iframe src="https://stackblitz.com/edit/indepth-rxjs-behavior-subject?embed=1&file=index.ts"></iframe>

## Additional resources

- [Official documentation](https://rxjs-dev.firebaseapp.com/api/index/class/BehaviorSubject)

## See also

- [ReplaySubject](https://indepth.dev/reference/rxjs/subjects/replay-subject)
- [AsyncSubject](https://indepth.dev/reference/rxjs/subjects/async-subject)
