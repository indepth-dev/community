---
name: publishReplay
title: publishReplay - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/publish-replay
tags: rxjs, javascript, reactive programming
---

# publishReplay

`publishReplay` operator is a variation of the [publish](https://indepth.dev/reference/rxjs/operators/publish) operator which internally uses [ReplaySubject](https://indepth.dev/reference/rxjs/subjects/replay-subject) instead of the plain [Subject](https://indepth.dev/reference/rxjs/subjects) in the case of the `publish` operator. `publishReplay` makes it possible to share a **single** subscription to the underlying stream between multiple subscribers and replay a set of values that happened  before the underlying stream completed.

`publishReplay` can be used as a shortcut for the [multicast](https://indepth.dev/reference/rxjs/operators/multicast) operator with the `ReplaySubject`, so to understand `publish` you need to understand [how `multicast` works](https://indepth.dev/reference/rxjs/operators/multicast). 

Here’s an example of `publishReplay` being used to share a subscription to the `obs` stream between 3 subscriptions. The first two subscribe before the observable stream completes and so receive all 3 values. The 3rd subscription is delayed with a timeout and occurs after the underlying stream completes, but still receives the last 2 values alongside the `COMPLETE` notification:

```javascript
const log = (index) => (v) => console.log(`subscription ${index}\t: ` + v);

const obs = interval(200).pipe(
   take(3),
   tap({ complete() { console.log('underlying stream completed') }})
);

const shared = obs.pipe(
   publishReplay(2)
);

shared.subscribe(log(1));
shared.subscribe(log(2));

shared.connect();

setTimeout(() => shared.subscribe(log(3)), 1000);
```

Here’s the output:

```
subscription 1	: 0
subscription 2	: 0
subscription 1	: 1
subscription 2	: 1
subscription 1	: 2
subscription 2	: 2
underlying stream completed
subscription 3	: 1
subscription 3	: 2
```

Notice how we got two more entries for the `subscription 3` compared to the output if we run the example with plain `publish`. Even though the 3rd subscription doesn’t reactive the underlying stream `obs` and it remains completed we still got two last values. That’s because those values are cached in the ReplaySubject and are replayed to the observer on subscription.

As mentioned earlier, `publishReplay` can be used as a shortcut for the [multicast](https://indepth.dev/reference/rxjs/operators/multicast) operator with the `ReplaySubject`, so we can do the replacement and still observe the same output:

```javascript
const shared = obs.pipe(
   multicast(new ReplaySubject(2))
);
```

Similarly to `multicast`, `publishReplay` simply allowed us to introduce the `ReplaySubject` into the observable chain. It’s the `ReplaySubject` that handles sharing, not the `publishReplay` or `multicast` operator. Both of them just simplify the workflow with the Subject.

In the case of the `multicast` though, you can pass any instance of a Subject yourself. With `publishReplay`, the underlying `ReplaySubject` is created internally and is never shared.

Note that `mulicast` operator is more powerful because it can take a factory function that can create a `ReplaySubject` instance on demand multiple times instead of an internal instance created once in the case of `publish` operator. You can learn more about this behavior [here](https://indepth.dev/reference/rxjs/operators/multicast).

In the following example using `multicast` with a factory results in the different output because the underlying subscription is reactivated:

```javascript
const log = (index) => (v) => console.log(`subscription ${index}\t: ` + v);

const obs = interval(200).pipe(
   take(3),
   tap({ complete() { console.log('underlying stream completed') }})
);

const shared = obs.pipe(
   multicast(() => new ReplaySubject(2))
);

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

Note how the 3rd subscription gets all 3 values and the underlying stream completes two times:

```
subscription 3	: 0
subscription 3	: 1
subscription 3	: 2
underlying stream completed
```

That's the effect of the `multicast` operator with the factory function.
