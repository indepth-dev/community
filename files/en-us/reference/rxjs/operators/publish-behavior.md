---
name: publishBehavior
title: publishBehavior - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/publish-behavior
tags: rxjs, javascript, reactive programming
---

# publishBehavior

`publishBehavior` operator is a variation of the [publish](https://indepth.dev/reference/rxjs/operators/publish) operator which internally uses [BehaviorSubject](https://indepth.dev/reference/rxjs/subjects/behavior-subject) instead of the plain [Subject](https://indepth.dev/reference/rxjs/subjects) in the case of the `publish` operator. `publishBehavior` makes it possible to share a **single** subscription to the underlying stream between multiple subscribers and replay the last cached value emitted **before** the underlying stream completes.

Here’s an example of using `publishBehavior` to share a subscription to the underlying `obs` stream between 3 subscriptions. To demonstrates the sharing behavior, we’ve implemented our custom `Observable` that logs emitted values:

```javascript
const log = (index) => (v) => console.log(`subscription ${index}\t: ` + v);
const logCompleted = (index) => () => console.log(`subscription ${index}\t: ` + 'completed');

const obs = new Observable(producer).pipe(
   take(3),
   tap({ complete() { console.log('underlying stream completed') }})
);

const shared = obs.pipe(
   publishBehavior(null)
) as ConnectableObservable<any>;

shared.subscribe({next: log(1), complete: logCompleted(1)});
shared.subscribe({next: log(2), complete: logCompleted(2)});

shared.connect();

setTimeout(() => shared.subscribe({next: log(3), complete: logCompleted(3)}), 500);
```

Similarly to `multicast`, `publishBehavior` operator returns a `ConnectableObservable` type that needs to be connected to the underlying stream.

As you can see, first two observers subscribe before the `shared` observable is connected and hence receive all 4 values including the initial `null` value required by `BehaviorSubject`. The 3rd subscription is delayed with a timeout and occurs after the first 2 values have been emitted, so it gets the cached value `1` and then receives the last remaining value `2` before the underlying stream completes.

Here’s the output:

```
subscription 1	: null
subscription 2	: null
producer		: 0
subscription 1	: 0
subscription 2	: 0
producer		: 1
subscription 1	: 1
subscription 2	: 1
subscription 3	: 1
producer		: 2
subscription 1	: 2
subscription 2	: 2
subscription 3	: 2
underlying stream completed
subscription 1	: completed
subscription 2	: completed
subscription 3	: completed
```

`publishBehavior` can be used as a shortcut for the [multicast](https://indepth.dev/reference/rxjs/operators/multicast) operator with the `BehaviorSubject`, so to understand `publishBehavior` you need to understand [how `multicast` works](https://indepth.dev/reference/rxjs/operators/multicast). It means that if we simply replace the `publishBehavior` with `multicast` like this we’ll get the same behavior:

```javascript
const shared = obs.pipe(
   multicast(new BehaviorSubject(null))
);
```

Note that, as opposed to the [publishReplay](https://indepth.dev/reference/rxjs/operators/publish-replay), **if the subscription occurs after the underlying steam completes, `publishBehavior` doesn’t send the cached value** like `publishReplay` does. It only sends the `COMPLETE` or `ERROR` notification.

Here’s an example of `publishBehavior` where just like in previous examples one subscription to the underlying stream `obs` is shared between 3 subscriptions. This time though, the 3rd subscription is delayed with a timeout and **occurs after the underlying stream completes**, so the only thing it receives is the `COMPLETE` notification:

```javascript
const obs = interval(200).pipe(
   take(3),
   tap({ complete() { console.log('underlying stream completed') }})
);

const shared = obs.pipe(
   publishBehavior(null)
);

shared.subscribe({next: log(1), complete: logCompleted(1)});
shared.subscribe({next: log(2), complete: logCompleted(2)});

shared.connect();

setTimeout(() => shared.subscribe({next: log(3), complete: logCompleted(3)}), 1000);
```

And the output is:

```
subscription 1	: null
subscription 2	: null
subscription 1	: 0
subscription 2	: 0
subscription 1	: 1
subscription 2	: 1
subscription 1	: 2
subscription 2	: 2
underlying stream completed
subscription 1	: completed
subscription 2	: completed
subscription 3	: completed
```

In the output you can see that the 3rd subscription only received the `COMPLETE` notification and didn’t receive any of the values cached by the `BehaviorSubject`.

Similarly to `multicast`, `publishBehavior` simply allowed us to introduce the `BehaviorSubject` into the observable chain. It’s the `BehaviorSubject` that handles sharing, not the `publishBehavior` or `multicast` operator. Both of them just simplify the workflow with the Subject.

In the case of the `multicast` though, you can pass any instance of a Subject yourself. With `publishBehavior`, the underlying `BehaviorSubject` is created internally and is never shared.

Note that `mulicast` operator is more powerful because it can take a factory function that can create a `BehaviorSubject` instance on demand multiple times instead of an internal instance created just once in the case of `publish` operator. You can learn more about this behavior [here](https://indepth.dev/reference/rxjs/operators/multicast).

In the following example using `multicast` with a factory results in the different output because the underlying subscription is reactivated:

```javascript
const obs = interval(200).pipe(
   take(3),
   tap({ complete() { console.log('underlying stream completed') }})
);

const shared = obs.pipe(
   multicast(() => new BehaviorSubject(null))
);

shared.subscribe({next: log(1), complete: logCompleted(1)});
shared.subscribe({next: log(2), complete: logCompleted(2)});

shared.connect();

setTimeout(() => { shared.subscribe({next: log(3), complete: logCompleted(3)}); shared.connect() }, 1000);
```

If you run this code snippet you’ll see the following output:

```
subscription 1	: null
subscription 2	: null
subscription 1	: 0
subscription 2	: 0
subscription 1	: 1
subscription 2	: 1
subscription 1	: 2
subscription 2	: 2
underlying stream completed
subscription 1	: completed
subscription 2	: completed
subscription 3	: null
subscription 3	: 0
subscription 3	: 1
subscription 3	: 2
underlying stream completed
subscription 3	: completed
```

Note how the 3rd subscription gets all 4 values (including the initial `null`) and the underlying stream completes two times:

```
subscription 3	: null
subscription 3	: 0
subscription 3	: 1
subscription 3	: 2
underlying stream completed
subscription 3	: completed
```

That's the effect of the `multicast` operator with the factory function.
