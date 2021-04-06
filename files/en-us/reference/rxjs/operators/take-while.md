---
name: takeWhile
title: takeWhile - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/take-while
tags: rxjs, javascript, reactive programming
---

# takeWhile

`takeWhile` passes values from the source observable to the observer (mirroring) as long as the function known as the predicate returns `true`. The predicate function is passed as an argument to `takeWhile`. The operator subscribes to the source observable and begins mirroring the source Observable. 

On each value `takeWhile` executes the predicate function passing the value as an argument and checks the returning value. If the value is `true`, it passes the value down to the observer, otherwise it completes the stream by unsubscribing from the source and passing the complete notification to the observer. 

Besides the predicate function `takeWhile` takes the second parameter that defines whether it should pass the value to the observer or ignore it when the predicate function returns `false`.

Note, that if you only need to take a specific number of values from the source observable, you should use [take](https://indepth.dev/reference/rxjs/operators/take) operator.

`takeWhile` operator works in the following way:

1. Subscribe to a source observable
2. When a new value arrives from a source observable, execute the predicate function
3. If the function returns truthy value, send the value to the observer
4. If the function return falsy value, send the value to the observer if the operator is configured to be inclusive, unsubscribe from the source observable and send the complete notification to the observer
5. Once the source observable completes, send the complete notification to the observer
6. If the source observable throws an error, send the error notification to the observer

The following diagram demonstrates this sequence of steps:

<video>
    <source src="https://images.indepth.dev/references/rxjs/operators/take-while.mp4" type="video/mp4">
</video>

## Usage
`takeWhile` can be used to destroy the chain and release resources once a certain value gets through. For example, you subscribe to the `mousemove` event when a mouse enters a box and want to unsubscribe when it leaves the box. 

Here’s how you would do it with `takeWhile`:

```javascript
const box = createBox();

fromEvent(box, 'mouseenter').pipe(
   exhaustMap(() => {
       // we're only interested in the mousemove event
       // as long as we're inside the box
       // once we leave the box, remove the listeners
       return fromEvent(box, 'mousemove').pipe(
           takeWhile((event: MouseEvent) => {
               return isEventInElement(
                   box.getClientRects(),
                   event.clientX,
                   event.clientY
               );
           })
       );
   })
).subscribe();

function isEventInElement(rect, clientX, clientY) {
   if (clientX < rect.left || clientX >= rect.right) return false;
   if (clientY < rect.top || clientY >= rect.bottom) return false;
   return true;
}

function createBox() {
   const box = document.createElement('div');

   box.style.backgroundColor = 'blue';
   box.style.position = 'fixed';
   box.style.top = '100px';
   box.style.left = '400px';
   box.style.width = '300px';
   box.style.height = '300px';

   document.body.appendChild(box);

   return box;
}
```

## Playground

<iframe src="https://stackblitz.com/edit/indepth-rxjs-take-while?embed=1&file=index.ts"></iframe>

## Additional resources

- [Official documentation](https://rxjs.dev/api/operators/takeWhile)

## See also

- [take](https://indepth.dev/reference/rxjs/operators/take)
- [takeUntil](https://indepth.dev/reference/rxjs/operators/take-until)
