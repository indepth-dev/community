---
name: ReplaySubject
title: ReplaySubject - RxJS Reference | indepth.dev
slug: reference/rxjs/subjects/replay-subject
tags: rxjs, javascript, reactive programming
---

# ReplaySubject

`ReplaySubject` is a variant of a [Subject](https://indepth.dev/reference/rxjs/subjects) which keeps a cache of previous values emitted by a source observable and sends them to all new observers immediately on subscription. This behavior of replaying a sequence of old values to new subscribes is where the name for this type of a subject comes from. When an observer subscribes to a ReplaySubject, the subject begins by first emitting all values from the cache and then continues to emit any other items emitted by the source observable after the subscription. `ReplaySubject` will replay the cached sequence of values even if the observer subscribes much later than the values were cached.

`ReplaySubject` is similar to the [BehaviorSubject](https://indepth.dev/reference/rxjs/subjects/behavior-subject) in the way that it can send cached values to new subscribers, but instead of just one current value, it can record and replay a whole series of values. 

There’s another more crucial difference. Once `BehaviorSubject` receives the complete or the error notification and transitions into a stopped state, all subsequent subscriptions will only receive the complete or the error notification and **will not** receive the current value. 

In contrast, even in the stopped state in case of a completion or an error on the source observable,  `ReplaySubject` **still replays the cached values** before sending the complete or the error error notification to new subsequent subscriptions.

When creating a `ReplaySubject` you can specify how many values you want to store in the buffer (bufferSize) and the amount of time to hold a value in the buffer before removing it from it (windowTime). Both configurations may exist simultaneously. 

So if you would like to buffer a maximum of 3 values, as long as the values are less than 2 seconds old, you could do so with a `new ReplaySubject(3, 2000)`. Another way to think about `windowTime` is to define it as the amount of time that has passed prior to a new subscription. In other words, the configuration above means “I want to store the last 3 values, that have been executed in the last 2 seconds prior to a new subscription”.

`ReplaySubject` works in the following way:

1. Create an internal subscriptions container
2. When a new subscription occurs, add it to the container and replay the values from the cache if any to the corresponding observer
3. When a source observable emits a new value or the method `next` is called on the subject, add the emitted value to the cache overriding earliest values if needed and send the new value to observers for all subscriptions
4. If the source observable completes or the method `complete` is called on the subject, set the state of the subject to `stopped` and add the complete notification to the cache; send the complete notification to all existing subscriptions, remove them from the container
5. If the source observable throws an error or the method `error` is called on the subject, set the state of the subject to `stopped` and add the `error` notification to the cache; send the `error` notification to all existing subscriptions, remove them from the container
6. If the subject is `stopped`, do not add any new subscriptions to the container and instead replay the values from the cache including complete or error notification immediately on subscription to the corresponding observer
7. If the `stopped` subject is subscribed to a new source observable, ignore the values from this source

The following diagram demonstrates this sequence of steps:

<video>
    <source src="https://images.indepth.dev/references/rxjs/subjects/replay-subject.mp4">
</video>

## Usage
`ReplaySubject` is commonly used when you need to replay an event or a series of events. Since `ReplaySubject` doesn’t need a default value as opposed to `BehaviorSubject`, it’s a handy mechanism to use if an event may never even occur.

Imagine you lazy load a library that needs to process user events. Some events will occur before the library is loaded, so you’ll need to replay them to the library. To do that, simply create a `ReplaySubject`, push events to it and let the library subscribe to it when loaded.

Here’s a code block that demonstrates this:

```javascript
const events = setUpListeners();
emulateLibraryLoad(events);

function emulateLibraryLoad(events) {
   setTimeout(() => {
       events.subscribe((event) => console.log(event));
   }, 3000);
}

function setUpListeners() {
   const events = new ReplaySubject();

   const clicks = fromEvent(document, 'click');
   const spacebars = fromEvent(document, 'keyup').pipe(filter((event: any) => event.code === 'Space'));

   merge(clicks, spacebars).pipe(
       tap((event) => events.next(event))
   ).subscribe();

   return events.asObservable();
}
```

## Playground

<iframe src="https://stackblitz.com/edit/indepth-rxjs-replay-subject?embed=1&file=index.ts"></iframe>

## Additional resources

- [Official documentation](https://rxjs-dev.firebaseapp.com/api/index/class/ReplaySubject)

## See also

- [BehaviorSubject](https://indepth.dev/reference/rxjs/subjects/behavior-subject)
- [AsyncSubject](https://indepth.dev/reference/rxjs/subjects/async-subject)
