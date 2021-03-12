---
title: Operators - RxJS Reference | indepth.dev
slug: reference/rxjs/operators
---

# Operators

RxJS library implements the [Observable primitive](https://indepth.dev/reference/rxjs) and adds operators to enable more efficient workflows when dealing with streams of asynchronous events. While Observable is the foundation, operators allow composing and transforming streams of data in a declarative manner.

Operators are chained together using pipe method that is available on Observable. Here’s a basic example that transforms and filters a stream of mouse clicks to only notify a subscriber if the click happens inside an element:

```javascript
const rect = document.querySelector('.rect').getClientRects();

function isEventInElement(rect, clientX, clientY) {
   if (clientX < rect.left || clientX >= rect.right) return false;
   if (clientY < rect.top || clientY >= rect.bottom) return false;
   return true;
}

fromEvent(document, 'click').pipe(
   map((event: MouseEvent) => {
       return {x: event.clientX, y: event.clientY};
   }),
   filter(({x, y}) => {
       return isEventInElement(rect[0], x, y);
   })
).subscribe(() => console.log('inside'));
```

There are many different categories of operators. Here are some of the most common operators placed under their respective categories:

- Creation
  - of, from, fromEvent
- Filtering
- Transformation
- Combination
- Error handling
- Multicasting

An operator is a function that takes a source observable, creates a new observable inside and connects both observables through a subscription. This new observable is returned from the operator function and passed down an operators chain. The signature for an operator function looks like this:

```javascript
(source: Observable<T>) => Observable<R>
```

Here’s a super simple operator that simply connects a source observable with a observer (subscriber) and passes the value along the chain:

```javascript
function operator(source) {
   return new Observable(observer => {
       source.subscribe((value) => {
           observer.next(value);
       });
   });
}
```

This operator would be used like this:

```javascript
fromEvent(document, 'click').pipe(
   operator
).subscribe((value: MouseEvent) => console.log(value.clientX));
```

It’s not a complete implementation since we don’t handle `complete` and `error` events. Nevertheless, this implementation clearly demonstrates the essence of operators. We have a new Observable created and returned and the subscription to the source is established.

In RxJS library all operators are implemented as higher-order functions that can take parameters and return an operator function. For example, the `map` operator takes a function this transforms a value that goes through the operator.

We can easily turn our custom operator from above into a higher-order function take takes a transformation function similar to the `map` operator:

```javascript
function customOperator(transform) {
   return (source) => {
       return new Observable(observer => {
           source.subscribe((value) => {
               const transformed = transform(value);
               observer.next(transformed);
           });
       });
   };
}
```

That will be used like this:

```javascript
fromEvent(document, 'click').pipe(
   customOperator((event) => event.clientX)
).subscribe((value: MouseEvent) => console.log(value));
```

To learn more about building your own operators check out [this article](https://indepth.dev/posts/1421/rxjs-custom-operators).
