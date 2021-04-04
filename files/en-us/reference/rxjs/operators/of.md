---
title: of - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/of
tags:
    -rxjs 
    -javascript 
    -reactive programming
---

# of

`of` is used to emit arguments as values in a sequence and then complete the stream. 

Unlike [from](https://indepth.dev/reference/rxjs/operators/from), it does not do any flattening or conversion and emits each argument as the same type it receives as arguments. If you pass it an Array (including array-like objects), a Promise and an iterable object it won’t be flattened into an observable sequence of values. Those arguments will be emitted as the same type, i.e. an Array, a Promise or an Iterable object without any conversion.

If you need to emit items asynchronously, the operator takes a scheduler as the second argument.

`of` operator works in the following way:

1. Create an observable instance
2. Take the next argument in queue and send it to the observer
3. When there are no more values in the data source, send the complete notification to the observer

The following diagram demonstrates this sequence of steps:

<video>
    <source src="https://images.indepth.dev/references/rxjs/operators/of.mp4" type="video/mp4">
</video>

## Usage
`of` is commonly used when you simply need to return a value where an observable is expected or start an observable chain. This is a common need when using combination operators like `mergeMap`. Here’s an example that checks if there’s a cached value for the URL and if so, returns the value immediately, otherwise it makes a request:

```javascript
from(urls).pipe(
   mergeMap((url) => {
       const cached = cache[url];
       // here we use `of` to create an observable from a plain value
       if (cached) return of(cached);
       return fromFetch(url).pipe(switchMap(response => response.json()));
   })
).subscribe((value) => console.log(value));
```

Check [this section](https://indepth.dev/reference/rxjs/operators/merge-map) to learn how `mergeMap` operator works.

## Playground

<iframe src="https://stackblitz.com/edit/indepth-rxjs-of?embed=1&file=index.ts"></iframe>

## Additional resources

- [Official documentation](https://rxjs.dev/api/operators/of)

## See also

- [from](https://indepth.dev/reference/rxjs/operators/from)
