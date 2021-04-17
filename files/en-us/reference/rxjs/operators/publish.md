---
name: publish
title: publish - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/multicast
tags: rxjs, javascript, reactive programming
---

# publish

`publish` operator is a mechanism to introduce a [Subject](https://indepth.dev/reference/rxjs/subjects) into the observable stream. This makes it possible to share a **single** subscription to the underlying stream between multiple subscribers.

`publish` is mostly used as a shortcut for the `multicast` operator, so to understand `publish` you need to understand [how multicast works](https://indepth.dev/reference/rxjs/operators/multicast).

Here’s how `multicast` is used to share a subscription to the underlying `obs` stream between `subscription 1` and `subscription 2`:

```javascript
const log = (index) => (v) => console.log(`subscription ${index}\t: ` + v);

const obs = interval(200).pipe(
   take(3),
   tap({ complete() { console.log('underlying stream completed') }})
);

const shared = obs.pipe(
   publish()
);

shared.subscribe(log(1));
shared.subscribe(log(2));

shared.connect();
```

Similarly to `multicast`, `publish` operator returns a `ConnectableObservable` type that needs to be connected to the underlying stream.

Here’s the output of the code snippet above:

```javascript
subscription 1	: 0
subscription 2	: 0
subscription 1	: 1
subscription 2	: 1
subscription 1	: 2
subscription 2	: 2
underlying stream completed
```

Note that although both subscriptions got the values, the underlying `obs` stream was activated (subscribed) only once and shared between `subsctiption 1` and `subsctipion 2`, hence we get only one `COMPLETE` notification.

To see it more clearly, we could create our own `Observable` that logs the emitted values:

```javascript
function producer(observer) {
   let counter = 0;

   const id = setInterval(() => {
       console.log('producer\t\t: ' + counter);
       observer.next(counter);
       counter++;
   }, 200);

   return () => clearInterval(id);
}

const obs = new Observable(producer).pipe(
   take(3),
   tap({ complete() { console.log('underlying stream completed') }})
);

const shared = obs.pipe(
   publish()
);

shared.subscribe(log(1));
shared.subscribe(log(2));

shared.connect();
```

And the output will be the following:

```
producer		: 0
subscription 1	: 0
subscription 2	: 0
producer		: 1
subscription 1	: 1
subscription 2	: 1
producer		: 2
subscription 1	: 2
subscription 2	: 2
underlying stream completed
```

As you can see, we are getting 3 values logged - 2 for each subscription handler and **only one** for the producer function. This means that the producer function is activated only once and the underlying stream is shared.

As mentioned earlier, `publish` can be used as a shortcut for the [multicast](https://indepth.dev/reference/rxjs/operators/multicast) operator with the `Subject, so we can do the replacement and still observe the same output:

```javascript
const shared = obs.pipe(
   multicast(new Subject())
);
```

Similarly to `multicast`, `publish` simply allowed us to introduce the `Subject` into the observable chain. It’s the `Subject` that handles sharing, not the `publish` or `multicast` operator. Both of them are there just to simplify the workflow with the `Subject`.

In the case of the `multicast` though, you can pass any instance of a Subject yourself. With `publish`, the underlying `Subject` is created internally and is never shared.

Note that `mulicast` operator is more powerful because it can take a factory function that can create a Subject instance on demand multiple times instead of an internal instance created once in the case of `publish` operator. You can learn more about this behavior [here](https://indepth.dev/reference/rxjs/operators/multicast).

In the following example using `multicast` with **a factory function** that returns a `Subject` results in the different output because the underlying subscription is reactivated:

```javascript
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
producer		: 0
subscription 1	: 0
subscription 2	: 0
producer		: 1
subscription 1	: 1
subscription 2	: 1
producer		: 2
subscription 1	: 2
subscription 2	: 2
underlying stream completed
producer		: 0
subscription 3	: 0
producer		: 1
subscription 3	: 1
producer		: 2
subscription 3	: 2
underlying stream completed
```

Note how the 3rd subscription gets all 3 values and the underlying stream completes two times.

```
producer		: 0
subscription 3	: 0
producer		: 1
subscription 3	: 1
producer		: 2
subscription 3	: 2
underlying stream completed
```

That's the effect of the `multicast` operator with the factory function.

There are a couple of variations of the `publish` operator that instead of the plain `Subject` use [ReplaySubject](https://indepth.dev/reference/rxjs/subjects/replay-subject) or [BehaviorSubject](https://indepth.dev/reference/rxjs/subjects/behavior-subject). Those operators are [publishReplay](https://indepth.dev/reference/rxjs/operators/publish-replay) and publishBehavior.

