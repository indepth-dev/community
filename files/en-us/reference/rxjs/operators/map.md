---
name: map
title: map - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/map
tags: rxjs, javascript, reactive programming
---

# map

`map` is part of the so-called transformation operators group because it’s used to transform each item received from the source observable. The operator passes each source value through a projection function to get corresponding output value and emits it to an observer. The idea is very similar to the standard [map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map) method on `Array`.

The projection function applied to each value emitted by the source also receives the `index` parameter, which indicates the number of an element that has happened since the subscription, starting from the number 0.

`map` operator works in the following way:

1. Subscribe to a source observable
2. When a new value arrives from a source observable, execute the projection function for the current value
3. Send the returned value to the observer
4. Once the source observable completes, send the complete notification to the observer
5. If the source observable throws an error, send the error notification to the observer

The following diagram demonstrates this sequence of steps:

<video>
    <source src="https://images.indepth.dev/references/rxjs/operators/map.mp4" type="video/mp4">
</video>

## Usage
`map` operator is one of the most used operators in RxJS library. For example, here’s how you can use it to trim spaces from strings:

```javascript
const strings = [' some', 'another '];
from(strings).pipe(
   map((value) => value.trim())
).subscribe((s) => console.log(s));
```

One common usage includes retrieving a set of properties from an object:

```javascript
const clicks = fromEvent(document, 'click');
const positions = clicks.pipe(
   map((event: MouseEvent) => {
       return {
           x: event.clientX,
           y: event.clientY
       };
   })
);
positions.subscribe(x => console.log(x));
```

## Playground

<iframe src="https://stackblitz.com/edit/indepth-rxjs-map?embed=1&file=index.ts"></iframe>

## Additional resources

- [Official documentation](https://rxjs.dev/api/operators/map)
