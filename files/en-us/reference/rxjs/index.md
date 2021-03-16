---
title: RxJS Reference | indepth.dev
slug: reference/rxjs/operators
---

# What is RxJS and Observable???

Most web applications today are driven by a range of asynchronous events. User clicks on a button, the application may react by sending a network request. The network request comes back, the application may react by updating the DOM. The reactive pattern like this is something web developers deal with everyday.

To deal with those events we use variations of [the observer pattern](https://en.wikipedia.org/wiki/Observer_pattern), which is a software design pattern that efficiently allows components of an application to react to certain incoming events or streams of data. The architecture that uses this pattern is known as event-driven code, where functions are triggered in response to incoming data.

Two most common patterns of working with these events are callbacks and promises. Here’s how we’d implement our scenario from above using these techniques:

```javascript
// callback
document.addEventListener('clicks', () => {
    // promise
    fetch('https://api.mocki.io/v1/b043df5a').then((response) => {
        response.json().then((data) => {
            // update state
        });
    });
});
```

In the case above `document` acts as a continues stream of `clicks` events. Similarly, a network response comes in chunks and represents a sequence of [the progress](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/progress_event) events. So we have an asynchronous nature of events and a sequence of possible data values. That's where the Observable type and RxJS comes in handy.

An Observable represents a sequence of values which may be observed. Observables are attached to streams of data, and are responsible for processing that data and deliver it to Observers. There's [an ongoing effort](https://github.com/tc39/proposal-observable) to bring Observable as a standalone type to the ECMAScript standard library.

> RxJS library implements the Observable primitive and adds operators that allow composing sequences of asynchronous events together.

Observable is a very simple type for a unified API that can represent a wide variety of things: events, multiple values, single values, user interactions, streaming data, synchronous values, asynchronous values, etc. The real power of Observables lies in their guarantees and characteristics.

Observable is a lazy type that guarantees:
- Once it's complete, errored, or unsubscribed, you will get no more messages
- Registered teardown will occur
- If you complete, error, or unsubscribe, you are guaranteed to clean up resources

Observables are also:
- Compositional: Observables can be composed with higher-order combinators
- Lazy: Observables do not start emitting data until an observer has subscribed
- Cancellable: Observables give you the ability to cancel the underlying operation of the source by unsubscribing

Let’s take our code from example above and see how we can transform it to leverage all characteristics of an observable listed above.

First, we'll use the Observable constructor to create an observable stream of `clicks` events for the `document`:

```javascript
import { Observable } from 'rxjs';

const producer = observer => {
    // Create an event handler which sends data to the observer
    let handler = event => observer.next(event);

    // Attach the event handler
    document.addEventListener('click', handler, true);

    // Return a cleanup function which will cancel the event stream
    return () => {
        // Detach the event handler from the element
        document.removeEventListener('click', handler, true);
    };
};

const events = new Observable(producer);
```

The logic to set up and teardown a stream of values is packed inside the `producer` function. Observable simply takes that function and handles infrastructure logic like executing the `producer` function on subscription and running the cleanup function when subscription is closed (on unsubscribe).

RxJS introduces lots of operators, and `fromEvent` allows setting up a stream from events. Here’s how you can use it:

```javascript
const clicks = fromEvent(document, 'click');
```

Once we have the stream of `click` events set up, we can start listening to events by subscribing:

```javascript
let subscription = clicks.subscribe(() => console.log('click event occurred'));
```

This is a short version that just passes a function to listen to `next` event. The full syntax is the following:

```javascript
let subscription = clicks.subscribe({
    next(val) {console.log('click event occurred')},
    error(err) {console.log('received an error: ' + err)},
    complete() {console.log('stream completed')},
});
```

We mentioned earlier that an observable is **lazy** and **cancellable**.

It is lazy because the `producer` function is not executed until the observable is subscribed. This is a stark contrast to a Promise which executes a function passed to the constructor immediately without waiting for the call to `then` or other methods.

It is cancellable because once we call the `unsubscribe` method on an object returned by `subscribe` the observable will execute the cleanup function. This function can remove an event listener, or in case of XHR call [abort](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/abort) method (effectively cancelling the request).

```javascript
subscription.unsubscribe();
```

For the long time Promise didn't have an API for cancelling, but you can do it today with [AbortController](https://developer.mozilla.org/en-US/docs/Web/API/AbortController).

The last characteristic of Observables we haven’t looked is that they are **composable**. Let’s say we have two observables - one that listens for the `click` events and the other sends a network request using `fetch` API:

```javascript
const clicks = fromEvent(document, 'click');
const request = fromFetch('https://api.mocki.io/v1/b043df5a');
```

We can combine (chain) them with the help of operators via the `pipe` method:

```javascript
clicks.pipe(operator).subscribe((data) => console.log(data));
```

Where the `operator` is implemented as a custom function that connects source observables with the observer:

```javascript
function operator(source) {
    return new Observable(observer => {
        source.subscribe(() => {
            request.subscribe((response) => {
                response.json().then((data) => {
                    observer.next(data);
                });
            });
        });
    });
}
```

Using standard operators from RxJS library is more common though. The functionality above can be implemented like this:

```javascript
clicks.pipe(
    mergeMap(() => request.pipe(
        switchMap((response) => fromPromise(response.json())))
    )
).subscribe((data) => console.log(data));
```

You can learn more about how operators work [here](https://indepth.dev/reference/rxjs/operators#what-are-operators).

We will briefly describe some other concepts you’ll likely encounter in RxJS:
- Observer: is a collection of callbacks that knows how to listen to values delivered by the Observable.
- Subscription: represents the execution of an Observable, is primarily useful for cancelling the execution.
- Subject: is the equivalent to an EventEmitter, and the only way of multicasting a value or event to multiple Observers.
- Schedulers: are centralized dispatchers to control concurrency, allowing us to coordinate when computation happens on e.g. setTimeout or requestAnimationFrame or others.
