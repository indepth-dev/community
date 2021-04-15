---
name: publish
title: publish - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/multicast
tags: rxjs, javascript, reactive programming
---

# publish

`publish` operator is a mechanism to introduce a [Subject](https://indepth.dev/reference/rxjs/subjects) into the observable stream. This makes it possible to share a **single** subscription to the underlying stream between multiple subscribers.
`publish` is mostly used as a shortcut for the [multicast](https://indepth.dev/reference/rxjs/operators/multicast) operator, so to understand `publish` **you need to understand how multicast works**. 

Here’s how `publish` operator can be used to share a subscription to the `obs` stream between `subscription 1` and `subscription 2` :

```javascript
const obs = interval(500).pipe(take(3));

const shared = obs.pipe(
   publish()
);

shared.subscribe((v) => console.log('subscription 1\t: ' + v));
shared.subscribe((v) => console.log('subscription 2\t: ' + v));

shared.connect();
```

You can simply replace the `publish` with ``multicast(new Subject()`` and the behavior will be the same:

```javascript
const shared = obs.pipe(
   multicast(new Subject())
);
```

Similarly to `multicast`, `publish` simply allowed us to introduce the Subject into the observable chain. It’s the Subject that handles sharing, not the `publish` or `multicast` operator. Both of them are used to simplify the workflow with a Subject.

In the case of the `multicast` though, you can pass any instance of a Subject yourself. With `publish`, the underlying Subject is created internally and is never shared.

If you run the code, you’ll see that the output either for `mulicast` or for `publish` is the same:

```
producer		: 0
subscription 1	: 0
subscription 2	: 0
producer		: 1
subscription 1	: 1
subscription 2	: 1
```

As you can see, we are getting 3 values logged - 2 for each subscription handler and **only one** for the producer function. This means that the producer function is activated only once and the underlying stream is shared.

Note that `mulicast` operator is more powerful because it can take a factory function that **can create a Subject instance on demand multiple times** instead of an internal instance created once in the case of `publish` operator. You can learn more about this behavior [here](https://indepth.dev/reference/rxjs/operators/multicast).

There are a couple of variations of the `publish` operator that instead of the plain Subject use [ReplaySubject](https://indepth.dev/reference/rxjs/subjects/replay-subject) or [BehaviorSubject](https://indepth.dev/reference/rxjs/subjects/behavior-subject). Those operators are `publishReplay` and `publishBehavior`.