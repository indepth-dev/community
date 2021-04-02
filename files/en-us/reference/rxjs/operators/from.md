---
title: from - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/from
tags: rxjs, javascript, reactive programming
---

# from

`from` is used to convert pure JavaScript object types like an Array (including array-like objects), a Promise and an iterable object into an observable sequence of values. It even works on an Observable-like objects, objects containing a function named with the ES2015 Symbol for Observable (Symbol.observable).

In the case of arrays and iterables, all contained values will be emitted as a sequence. The operator also emits a string as a sequence of characters. If you need to emit items asynchronously, the operator takes a scheduler as the second argument. When a promise needs to be used as part of an observable chain,  `from` is often used to convert a promise into an observable.

`from` operator works in the following way:

1. Create an observable instance
2. If the parameter is Array, iterate over the source and emit array members as standalone values
3. If the parameter is Iterable, iterate over the source and emit members as standalone values
4. If the source is Promise, wait until the promise is resolved, and send the value to the observer. If the promise is rejected, send the error notification to the observer
5. When there are no more values in the data source, send the complete notification to the observer

The following diagram demonstrates this sequence of steps:

<video>
    <source src="https://images.indepth.dev/references/rxjs/operators/from.mp4" type="video/mp4">
</video>

## Usage
`from` is commonly used operator when you need to create an observable sequence out of pure JavaScript types. Here’s an example that shows how to turn an Array, a Promise and an Iterable into a sequence of values:

```javascript
const array = ['array-1', 'array-2', 'array-3'];
const promise = Promise.resolve('promise-1');
const iterable = iterator();

concat(
   from(array),
   from(iterable),
   from(promise)
).subscribe(x => console.log(x));

function* iterator() {
   const values = ['iterator-1', 'iterator-2', 'iterator-3'];
   for (let i = 0; i < values.length; i++) {
       yield values[i];
   }
}
```

Check [this section](https://indepth.dev/reference/rxjs/operators/concat-map) to learn how `concat` operator works.

## Playground

<iframe src="https://stackblitz.com/edit/indepth-rxjs-from?embed=1&file=index.ts"></iframe>

## Additional resources

- [Official documentation](https://rxjs.dev/api/operators/from)

## See also

- [of](https://indepth.dev/reference/rxjs/operators/of)
