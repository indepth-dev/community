---
name: multicast
title: multicast - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/multicast
tags: rxjs, javascript, reactive programming
---

# multicast

`multicast` operator is a mechanism to introduce a [Subject](https://indepth.dev/reference/rxjs/subjects) into the observable stream. Doing so allows sharing a **single** subscription to the underlying stream between multiple subscribers. Think of multicasting as casting values to multiple observers, hence the name of the operator.

Take a look at this example:

```javascript
function producer(observer) {
   let counter = 0;
   setInterval(() => {
       console.log('producer\t\t: ' + counter);
       observer.next(counter);
       counter++;
   }, 1000);
}

const obs = new Observable(producer);

obs.subscribe((v) => console.log('subscription 1\t: ' + v));
obs.subscribe((v) => console.log('subscription 2\t: ' + v));
```

We have the stream `obs` that emits numbers with the interval of one second and logs the values. 

Once we subscribe to it, it starts emitting values. Each subscription will trigger **its own** producer function, so we’ll have two streams running each for a different subscription. It means we’ll see each number logged 4 times - 2 times from inside of each producer function and 2 times from inside of each subscription handler:

```
producer		: 0
subscription 1	: 0
producer		: 0
subscription 2	: 0
producer		: 1
subscription 1	: 1
producer		: 1
subscription 2	: 1
```

If we want to share the underlying stream between multiple subscriptions so that the producer function is only triggered once, we need to use a Subject. Here’s how we can do it:

```javascript
const shared = new Subject();

shared.subscribe((v) => console.log('subscription 1\t: ' + v));
shared.subscribe((v) => console.log('subscription 2\t: ' + v));

obs.subscribe(shared);
```

Here’s what we’ll see in the console:

```
producer		: 0
subscription 1	: 0
subscription 2	: 0
producer		: 1
subscription 1	: 1
subscription 2	: 1
```

As you can see, we now are getting 3 values logged - 2 for each subscription handler and only one for the producer function. This means that the producer function is activated only once.

It’s a bit elaborate to create a subject and deal with subscriptions in this way. That’s where `multicast` can help. Here’s how we can achieve the same result using `multicast`:

```javascript
const shared = obs.pipe(
   multicast(new Subject())
);

shared.subscribe((v) => console.log('subscription 1\t: ' + v));
shared.subscribe((v) => console.log('subscription 2\t: ' + v));

shared.connect();
```

As you can see, `multicast` simply allowed us to introduce the Subject into the observable chain. The previous example shows us that it’s the Subject that handles sharing, not the `multicast` operator. We used the `multicast` operator to simplify the workflow with the Subject.

One thing to notice is that we’re calling `connect` method on the `shared` which is of `ConnectableObservable` type. This is a way to connect an underlying subject to the original stream. By itself subscriptions to a connectable observable `shared` do not trigger a subscription of the Subject to the source. When we subscribed to `shared` we essentially subscribed to the Subject. 

Calling `connect` on connectable observable is similar in a way to directly subscribing a Subject to the source stream like we did in earlier example:

```javascript
const shared = new Subject();

shared.subscribe((v) => console.log('subscription 1\t: ' + v));
shared.subscribe((v) => console.log('subscription 2\t: ' + v));

obs.subscribe(shared);
```

Instead of a Subject instance, multicast can accept a factory function that creates a Subject instance. There’s a difference between two options. 

The standard behavior of a Subject is once it’s stopped, it can never re-subscribe to the underlying stream. It means that once the underlying stream completes, all future subscriptions will simply receive the COMPLETE notification. Here we’re limiting the stream to 3 values and then complete the stream. The stream completes in 1500ms, and within 2000ms we subscribe again but only receive the COMPLETE notification:

```javascript
const obs = interval(500).pipe(take(3));

const shared = obs.pipe(
   multicast(new Subject())
) as ConnectableObservable<any>;

shared.subscribe({
   next(v) { console.log(v) },
   complete() { console.log('complete') }
});

shared.connect();

setTimeout(() => {
   shared.subscribe({
       next(v) { console.log(v) },
       complete() { console.log('complete') }
   });
}, 2000);
```

Here’s the output:

```
0
1
2
complete
complete
```

However, if you pass the factory function, `multicast` will execute this factory function to get a new Subject if the current subject is stopped and there’s a new subscription. You’ll need to connect it again. Here’s the code that demonstrates this behavior:

```javascript
const obs = interval(500).pipe(take(3));

const shared = obs.pipe(
   multicast(() => new Subject())
) as ConnectableObservable<any>;

shared.subscribe({
   next(v) { console.log(v) },
   complete() { console.log('complete') }
});

shared.connect();

setTimeout(() => {
  
   shared.subscribe({
       next(v) { console.log(v) },
       complete() { console.log('complete') }
   });
  
   shared.connect();
}, 2000);
```

And this time the output is different:

```
0
1
2
complete
0
1
2
complete
```

Notice how we call `connect` again in the `setTimeout` after the subscription. It’s a new Subject we’re connecting. If you put a logger inside the factory function we passed to `multicast`, you’ll see it being executed once the new subscription in the `setTimeout` happens.



