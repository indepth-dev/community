---
name: share
title: share - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/share
tags: rxjs, javascript, reactive programming
---

# share

`share` operator is a mechanism to share (multicast) a **single** subscription to the underlying source observable between multiple subscribers and automate the process of re-subscription to this underlying stream. Similarly to the `publish` and `multicast` operators, it uses a [Subject](https://indepth.dev/reference/rxjs/subjects) internally to handle the sharing (multicasting) logic.

`share` is an alias for the `multicast` operator with the factory function and `refCount` operator applied on top of it. So, this code snippet:

```javascript
const shared = obs.pipe(
   share()
);
```

Is equivalent to this one:

```javascript
const shared = obs.pipe(
   multicast(() => new Subject()),
   refCount()
);
```

It means that to understand `share` you first need to understand [how multicast with a factory function works](https://indepth.dev/reference/rxjs/operators/multicast). That’s the foundation. Similarly to `multicast`, `share` simply allows us to introduce the `Subject` into the observable chain. It’s the `Subject` that handles sharing, not the `share` or `multicast` operator. Both of them are there just to simplify the workflow with the `Subject`.

The logic behind `refCount` is pretty straightforward. As long as there is at least one subscriber to a shared observable the `refCount` operator keeps an active subscription to the underlying source observable. When all subscribers have unsubscribed it will unsubscribe from the underlying stream. Internally it counts the subscriptions to the observable and subscribes to the source if the number of subscriptions is larger than 0. If the number of subscriptions is smaller than 1, it unsubscribes from the source. This way you can make sure that everything before the published `refCount` has only a single subscription independently of the number of subscribers to the target observable.

The `refCount` operator basically automates the process of connecting shared observable to the source stream:

- If there’s no active subscription to the source observable when a new subscription happens, it calls the `connect` method on the `ConnectableObservable` returned by the `multicast` operator
- When the shared observable is unsubscribed and there are no more active subscriptions to the shared observable, it unsubscribes from the underlying source stream
- If the underlying stream completes, it will send the `COMPLETE` notification to all subscriptions to the shared observable. If later a new subscription is added to the shared observable, it will re-subscribe to the underlying stream by calling the `connect` method on the `ConnectableObservable` returned by the `multicast` operator

The following example demonstrates how `multicast` with **a factory function** and the `connect` method can be used to re-subscribe to the source observable after this observable completes:

```javascript
const log = (index) => (v) => console.log(`subscription ${index}\t: ` + v);

const obs = new Observable(producer).pipe(
   take(3),
   tap({ complete() { console.log('underlying stream completed') }})
);

const shared = obs.pipe(
   multicast(() => new Subject())
) as ConnectableObservable<any>;

shared.subscribe(log(1));
shared.subscribe(log(2));

shared.connect();

setTimeout(() => { shared.subscribe(log(3)); shared.connect() }, 1000);
```

If you run this code snippet you’ll see the following output:

```
subscription 1	: 0
subscription 2	: 0
subscription 1	: 1
subscription 2	: 1
subscription 1	: 2
subscription 2	: 2
underlying stream completed
subscription 3	: 0
subscription 3	: 1
subscription 3	: 2
underlying stream completed
```

Note that although subscriptions 1 and 2 both got the values, the underlying `obs` stream was activated (subscribed) only once and shared between `subsctiption 1` and `subsctipion 2`, hence we got only one `COMPLETE` notification before the 3rd subscription occurred.

However, because the 3rd subscription was delayed with a timeout and occurred after the source observable completed, calling `connect` on the shared observable re-subscribed to the underlying source observable and so this 3rd subscription received all 3 values and completed again. That’s why you see two `COMPLETE` notifications in the output.

That's the effect of the `multicast` operator with the factory function.

You can now automate this process of re-subscription using the `refCount` operator:

```javascript
const shared = obs.pipe(
   multicast(() => new Subject()),
   refCount()
);

shared.subscribe(log(1));
shared.subscribe(log(2));
```

When `refCount` is used, we no longer need to manually call `connect` to establish a subscription to the underlying source observable. The operator does it for us once the first active subscription to the shared observable occurs.

As mentioned in the beginning, we can replace the pair of `multicast` and `refCount` operators with `share` to get exactly the same behavior and the output:

```javascript
const shared = obs.pipe(
    share()
);

shared.subscribe(log(1));
shared.subscribe(log(2));
```

There is a variation of the `share` operator, `shareReplay`, which instead of the plain `Subject` used by `share` uses [ReplaySubject](https://indepth.dev/reference/rxjs/subjects/replay-subject).
