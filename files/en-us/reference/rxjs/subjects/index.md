---
title: Subjects - RxJS Reference | indepth.dev
slug: reference/rxjs/subjects
---

# What is RxJS Subject

Subjects allow multiple observers to subscribe to them and then broadcast the received value to those observers. The concept is similar to an `EventEmitter` that keeps a registry of multiple listeners. When an event happens, e.g. new value arrives, it notifies those listeners. In the context of Subjects and RxJS, delivering values to multiple observers is called multicasting.

Here’s a simple example:

```javascript
const subject = new Subject();

subject.subscribe((value) => console.log(value));
subject.subscribe((value) => console.log(value));
subject.subscribe((value) => console.log(value));

subject.next('some value');
```

We create a subject that has 3 subscribers. Once we emit a value through the subject, all 3 subscribers receive it.

The code snippet above demonstrates an interesting characteristic of a Subject - it is both an Observable and an Observer. Since it’s an Observable, you can subscribe to it. It has the `subscribe` method. And since it’s an Observer, you can call the `next` method on it. Just like you do when you create an observable and call the `next` method on an observer you get as a parameter:

```javascript
const observable = new Observable((observer) => {
   observer.next('some value');
});
```

To prevent outside code from calling `next` on a subject, the method `asObservable` exists:

```javascript
function listen() {
   const subject = new Subject();
   // some code here that calls `subject.next`
   return subject.asObservable();
}
```

In the code snippet above only the code inside the `listen` function can call `.next` method. If outside code calls the method, the error Property `next` does not exist on type `Observable‹unknown›` is thrown:

```javascript
const subject = listen();
subject.next('d'); // produces the error
```

## Varieties of Subject

There are four varieties of Subject that are designed for particular use cases. Let’s briefly cover them.

### Plain Subject

A standard Subject with no replay behaviour and no initial value. New subscribers will receive emissions from the Subject’s source Observable(s).

Here’s a diagram that demonstrates the behavior:

<video>
    <source src="https://images.indepth.dev/references/rxjs/Subject/subject_1.mp4">
</video>

### Async Subject

A `AsyncSubject` emits the last value (and only the last value) emitted by the source Observable, and only after that source Observable completes. (If the source Observable does not emit any values, the AsyncSubject also completes without emitting any values).

It will also emit this same final value to any subsequent observers. However, if the source Observable terminates with an error, the `AsyncSubject` will not emit any items, but will simply pass along the error notification from the source Observable.

See more [here](https://indepth.dev/reference/rxjs/subjects/async-subject).

### Behaviour Subject

When an observer subscribes to a `BehaviorSubject`, it begins by emitting the item most recently emitted by the source Observable (or a seed/default value if none has yet been emitted) and then continues to emit any other items emitted later by the source Observable(s).

However, if the source Observable terminates with an error, the BehaviorSubject will not emit any items to subsequent observers, but will simply pass along the error notification from the source Observable.

See more [here](https://indepth.dev/reference/rxjs/subjects/behavior-subject).

### Replay Subject

A `ReplaySubject` emits to any observer all of the items that were emitted by the source Observable(s), regardless of when the observer subscribes.

There are also versions of `ReplaySubject` that will throw away old items once the replay buffer threatens to grow beyond a certain size, or when a specified timespan has passed since the items were originally emitted.

See more [here](https://indepth.dev/reference/rxjs/subjects/replay-subject).


