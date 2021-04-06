---
name: distinctUntilChanged
title: distinctUntilChanged - RxJS Reference | indepth.dev
slug: reference/rxjs/operators/distinct-until-changed
tags: rxjs, javascript, reactive programming
---

# distinctUntilChanged

`distinctUntilChanged` emits all items emitted by the source observable that are distinct by comparison from the previous item. By default, it uses simple comparison operator that doesn’t check any keys, so object references must match for an object to be considered the same.

The operator takes an optional comparison function that will be called to test if an item is distinct from the previous item. The function receives the current and the previous values as parameters and must return a boolean value that determines whether or not that value should be emitted to the observer or be ignored. Since it’s a comparison function, if it returns true, it means that values are the same and hence the value is ignored, otherwise it’s sent to the observer.

Please note that this operator doesn’t select unique values, it simply prevents two equal values in a row. It means, that if you have a sequence `{1,2,3,2}` with the number `2` appearing two times, it will still end up in the resulting output twice because those numbers do not follow each other. 

For more sophisticated use cases you can use the [filter](https://indepth.dev/reference/rxjs/operators/filter) operator instead.

`distinctUntilChanged` operator works in the following way:

1. Subscribe to a source observable
2. When a new value arrives from a source observable, run a comparison with the previous value either using a comparison function if provided or the strict comparison operator
3. If values match, ignore the current value, otherwise, send the value to the observer
4. Once the source observable completes, send the complete notification to the observer
5. If the source observable throws an error, send the error notification to the observer

The following diagram demonstrates this sequence of steps:

<video>
    <source src="https://images.indepth.dev/references/rxjs/operators/distinct-until-changed.mp4" type="video/mp4">
</video>

## Usage
One common usage of the `distinctUntilChanged` is to prevent dublicate emissions of the values on inputs. Consider the following setup:

```javascript
const input = document.createElement('input');
document.body.appendChild(input);

fromEvent(input, 'input').pipe(
   debounceTime(1000),
   map((event: any) => event.target.value)
).subscribe((value) => console.log(value));
```

Using `debounceTime(1000)` means we only receive a value when the user stopped typing for 1 second. However, during this 1 second the user can write some characters but then erase them. For example, a user types `ab`, then adds `c`, then removes it `c` within the 1s interval, but you’ll still both `ab` in the resulting set `{ab, ab}`.

By simply adding `distinctUntilChanged` we can prevent that:

```javascript
fromEvent(input, 'input').pipe(
   debounceTime(1000),
   map((event: any) => event.target.value),
   distinctUntilChanged()
).subscribe((value) => console.log(value));
```

The operator prevents emission of the second `ab` since it’s the same as the last emission.


## Playground

<iframe src="https://stackblitz.com/edit/indepth-rxjs-distinct-until-changed?embed=1&file=index.ts"></iframe>

## Additional resources

- [Official documentation](https://rxjs.dev/api/operators/distinctUntilChanged)

## See also

- [filter](https://indepth.dev/reference/rxjs/operators/filter)
