---
name: filter
title: filter - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/filter
tags: rxjs, javascript, reactive programming
---

# filter

`filter` emits all items received from the source observable that satisfy a specified comparison function known as the predicate. The idea is very similar to the standard [filter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter) method on `Array`.

The predicate is passed as a parameter and is executed for each value emitted by the source observable. If it returns `true`, the value is emitted, if `false` the value is ignored and not passed to the observer. The function also gest `index` parameter which is the counterer of values that have been emitted by the source observable since the subscription, starting from the number `0`.

If you just need to filter out consecutive similar values, you can use [distinctUntilChanged](https://indepth.dev/reference/rxjs/operators/distinct-until-changed). When you need to complete an observable if a condition fails use [takeWhile](https://indepth.dev/reference/rxjs/operators/take-while)!

`filter` operator works in the following way:

1. Subscribe to a source observable
2. When a new value arrives from a source observable, execute the predicate function for the current value
3. If the function returns `false, ignore the value, otherwise, send the value to the observer
4. Once the source observable completes, send the complete notification to the observer
5. If the source observable throws an error, send the error notification to the observer

The following diagram demonstrates this sequence of steps:

<video>
    <source src="https://images.indepth.dev/references/rxjs/operators/filter.mp4" type="video/mp4">
</video>

## Usage
`filter` quite often used in everyday development to filter out some values. For example, you only need to listen for the `click` event on `DIV` elements. For the efficiency, you add an event listener to the `document` and simply ignore DOM nodes that aren’t `DIV`. Here’s how you can do it:

```javascript
const div = document.createElement('div');
const span = document.createElement('span');
document.body.appendChild(div);
document.body.appendChild(span);

fromEvent(document, 'click').pipe(
   filter((event: any) => event.target.tagName === 'DIV')
).subscribe(event => console.log(event));

setTimeout(() => div.click(), 1000);

// this click event is filtered out and not emitted to the observer
setTimeout(() => span.click(), 2000);
```

## Playground

<iframe src="https://stackblitz.com/edit/indepth-rxjs-filter?embed=1&file=index.ts"></iframe>

## Additional resources

- [Official documentation](https://rxjs.dev/api/operators/filter)

## See also

- [distinctUntilChanged](https://indepth.dev/reference/rxjs/operators/distinct-until-changed)
