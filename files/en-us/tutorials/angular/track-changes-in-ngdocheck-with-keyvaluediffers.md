---
title: How to track changes in ngDoCheck with KeyValueDiffer - Angular Tutorials | indepth.dev
author_name: Dharmen Shah
author_link: https://twitter.com/shhdharmen
discussion_link: https://github.com/indepth-dev/community/discussions/190
display_name: How to track changes in ngDoCheck with KeyValueDiffer
---

# How to track changes in ngDoCheck with KeyValueDiffer

> When we use `ngDoCheck` to detect changes, we need to make sure that our implementation is extremely lightweight and fast, so it doesn’t affect user-experience. In this tutorial, we will learn how to efficiently track and process those changes using `KeyValueDiffer`.


## `ngDoCheck` life-cycle hook

The official definition of this life-cycle hook goes like this:

_"Detect and act upon changes that Angular can't or won't detect on its own. Called immediately after ngOnChanges() on every change detection run, and immediately after ngOnInit() on the first run."_

Simply put, Angular tracks binding inputs by object reference. It means that if an object reference hasn’t changed, the binding change is not detected and change detection is not executed. This is where we need [`ngDoCheck`](https://angular.io/api/core/DoCheck).


### Practical usage

It is very important to understand when to use `ngDoCheck` life-cycle hook when working with the code and how it’s different from `ngOnChanges`.

For example, we are going to consider two components:

1. `my-app` - Has the basic layout and `rates` property, which represents the rates of INR for 1 USD over time.
2. `app-rates` - Accepts single `@Input` for `rates`

Our goal is to track changes of `rates.inr` and display the same in `app-rates`. Let’s start with coding:

```typescript
// app.component.ts

import { Component } from '@angular/core';

@Component({
  selector: 'my-app',
  template: `
  <button (click)="updateRates()">Update rates</button>
  <div>
      <h4>{{ 1 | currency }} = {{ rates.inr | currency: 'INR' }}</h4>
      <app-rates [rates]="rates"></app-rates>
  </div>
  `,
})
export class AppComponent {
  rates: { inr: number } = { inr: 0 };

  updateRates() {
    this.rates.inr = 75;
  }
}
```


`my-app`’s code is basic. It just displays the `rates` and we have also given a `button` which will update the `rates` by calling `updateRates`.

Let’s look at `app-rates`’s code:

```typescript
// rates.component.ts

import {
  Component,
  DoCheck,
  Input,
  OnChanges,
  SimpleChanges,
} from '@angular/core';

@Component({
  selector: 'app-rates',
  template: `
  <span
    *ngIf="diff !== undefined; else noDiff"
    class="badge"
    [class.bg-success]="diff > 0"
    [class.bg-danger]="diff < 0"
  >
    {{ diff | number: '1.0-2' }}
  </span>
  <ng-template #noDiff>
    <span class="badge bg-secondary">
      No difference
    </span>
  </ng-template>
  `,
})
export class RatesComponent {
  @Input() rates: { inr: number } = { inr: 0 };
  diff = undefined;
}
```

`app-rates`’s template only displays `diff`, which represents how much `rates.inr` has changed since last time. And if there is no change, it will show “No difference” text.

Now, to simply get `diff`, we will need to calculate the difference between new value and old value.

#### Why not `ngOnChanges`

We may think to do this with `ngOnChanges`. Let’s first see what changes we are getting in `ngOnChanges` life-cycle hook:

```typescript
export class RatesComponent implements OnChanges {
  // ...

  ngOnChanges(changes: SimpleChanges) {
    console.log('Is first change?', changes.rates.firstChange);
  }
}
```

Now, let’s keep an eye on the console and click on the “Update rates” button:

![console output on click of Update Rates button](https://i.imgur.com/hn7bVJt.gif "console output on click of Update Rates button")

Notice that `ngOnChanges` is getting called only when the `rates` is assigned for the first time. This is happening because we are not changing the `rates` object by reference from `my-app`. If we write something like below in `my-app`, then `ngOnChanges` will capture the changes:

```typescript
updateRatesByReference() {
    this.rates = { ...this.rates, inr: 70 };
}
```

#### Usage of `ngDoCheck`

Unlike `ngOnChanges`, `ngDoCheck` tracks all the changes, whether they are by reference or not and even more. Let’s utilise it in our example:

```typescript
export class RatesComponent implements DoCheck {
  @Input() rates: { inr: number } = { inr: 0 };
  diff = undefined;
  oldRate = 0;

  ngDoCheck() {
    if (this.rates.inr !== this.oldRate) {
      this.diff = this.rates.inr - this.oldRate;
      this.oldRate = this.rates.inr;
    }
  }
}
```

In the above code, we introduced a new property called `oldRate`. And in `ngDoCheck` we are checking if the new `rates.inr` is not same as `oldRate`, then it should update the `diff`. Let’s look at the output now:

![output after ngDoCheck](https://i.imgur.com/OgQkfa5.gif "output after ngDoCheck")

_For more on `ngDoCheck`, I would recommend you to read the article: [If you think `ngDoCheck` means your component is being checked — read this article - Angular inDepth](https://indepth.dev/posts/1131/if-you-think-ngdocheck-means-your-component-is-being-checked-read-this-article)._

This example is available on [stackblitz](https://stackblitz.com/edit/angular-ivy-tcrchs?file=src/app/rates/rates.component.ts). This code gives the result as expected. But Angular provides few utilities to efficiently track changes made to an object over time. Let’s look into those.

## KeyValueDiffer and utilities

There are a few interfaces and a service involved when we want to use `KeyValueDiffer`. Below is the illustration which covers them all:

![KeyValueDiffer flow](https://i.imgur.com/e7BjYGp.jpg "KeyValueDiffer flow")

Below is the summary:

1. We will inject the service [`KeyValueDiffers`](https://angular.io/api/core/KeyValueDiffers) and use its [`find()`](https://angular.io/api/core/KeyValueDiffers#find) method to get a `KeyValueDifferFactory`
2. Next, we will use  [`KeyValueDifferFactory`](https://angular.io/api/core/KeyValueDifferFactory)’s [`create()`](https://angular.io/api/core/KeyValueDifferFactory#create) method to create [`KeyValueDiffer`](https://angular.io/api/core/KeyValueDiffer)
3. We will track the changes through the `KeyValueDiffer`’s [`diff()`](https://angular.io/api/core/KeyValueDiffer#diff) method. It returns `KeyValueChanges`
4. And at last, we will analyse the changes from [`KeyValueChanges`](https://angular.io/api/core/KeyValueChanges) using one of its [methods](https://angular.io/api/core/KeyValueChanges#methods), for example [`forEachChangedItem`](https://angular.io/api/core/KeyValueChanges#foreachchangeditem)
    1. All methods provide access to change-record `KeyValueChangeRecord`
    2. The [`KeyValueChangeRecord`](https://angular.io/api/core/KeyValueChangeRecord) interface is a record representing the item change information

### Practical usage

We will use the above utilities in the `app-rates` which we created previously. We will start with blank `ngDoCheck`:

```typescript
export class RatesComponent implements DoCheck {
  @Input() rates: { inr: number } = { inr: 0 };
  diff = undefined;

  ngDoCheck() {}
}
```

Our goal here is to track the changes made to `rates` property with `KeyValueDiffer` utilities. 

#### Property of type `KeyValueDiffer`

Let’s first create a `differ`:

```typescript
differ: KeyValueDiffer<string, number>;
```

As the `rates` object has the key of type `string` and value of type `number`, we are passing two types, `string` and `number` respectively with `KeyValueDiffer`. You can change this as per your need.

#### Inject `KeyValueDiffers` service

Next, let’s inject the `KeyValueDiffers` service:

```typescript
constructor(private _differsService: KeyValueDiffers) {}
```

#### Initialize `KeyValueDiffer`

It’s time to initialize the `differ` from service. We will do it in `ngOnInit` life-cycle hook:

```typescript
ngOnInit() {
    this.differ = this._differsService.find(this.rates).create();
}
```

In the above code, first we are calling the `find()` method. This method internally first checks if the object passed as argument is either a `Map` or JSON and if the check is successful then it returns `KeyValueDiffersFactory`. You can checkout it’s source-code on [GitHub](https://github.com/angular/angular/blob/b1c028677f45e704342e81d7957d024c137340ce/packages/core/src/change_detection/differs/keyvalue_differs.ts#L179), but overall, below is how it looks:

```typescript
find(kv: any): KeyValueDifferFactory {
    const factory = this.factories.find(f => f.supports(kv));
    if (factory) {
      return factory;
    }
    throw new Error(`Cannot find a differ supporting object '${kv}'`);
  }
```

After `find()`, we are calling the `create()` method of `KeyValueDiffersFactory`, which creates a `KeyValueDiffer` object.

#### Track changes in `ngDoCheck`

Next, we will use the `differ` and call it’s `diff()` method inside `ngDoCheck`:

```typescript
ngDoCheck() {
    if (this.differ) {
      const changes = this.differ.diff(this.rates);
    }
  }
```

The `diff()` method returns `KeyValueChanges` or `null`. As mentioned earlier `KeyValueChanges` provides methods to track all the changes, additions, and removals.

In our case, we need to track changes made to `rates`, so we will use `forEachChangedItem()` and calculate the `diff`:

```typescript
ngDoCheck() {
    if (this.differ) {
      const changes = this.differ.diff(this.rates);
      if (changes) {
        changes.forEachChangedItem((r) => {
          this.diff = r.currentValue.valueOf() - r.previousValue.valueOf();
        });
      }
    }
  }
```

The final code of `app-rates` looks like below:

```typescript
@Component({
  selector: 'app-rates',
  template: `
  <span
    *ngIf="diff !== undefined; else noDiff"
    class="badge"
    [class.bg-success]="diff > 0"
    [class.bg-danger]="diff < 0"
  >
    {{ diff | number: '1.0-2' }}
  </span>
  <ng-template #noDiff>
    <span class="badge bg-secondary">
      No difference
    </span>
    </ng-template>
  `,
})
export class RatesComponent implements DoCheck, OnInit {
  @Input() rates: { inr: number } = { inr: 0 };
  oldRate = 0;
  diff = undefined;
  differ: KeyValueDiffer<string, number>;

  constructor(private _differsService: KeyValueDiffers) {}

  ngOnInit() {
    this.differ = this._differsService.find(this.rates).create();
  }

  ngDoCheck() {
    if (this.differ) {
      const changes = this.differ.diff(this.rates);
      if (changes) {
        changes.forEachChangedItem((r) => {
          this.diff = r.currentValue.valueOf() - r.previousValue.valueOf();
        });
      }
    }
  }
}
```

This example is also available on [stackblitz](https://stackblitz.com/edit/angular-ivy-nwzydo?file=src/app/rates/rates.component.ts).

## Conclusion

We first started with a brief intro to [`ngDoCheck`](https://angular.io/api/core/DoCheck). Then we learned the utilities needed to track the changes, i.e. interfaces [`KeyValueDiffer`](https://angular.io/api/core/KeyValueDiffer), [`KeyValueChanges`](https://angular.io/api/core/KeyValueChanges), [`KeyValueChangeRecord`](https://angular.io/api/core/KeyValueChangeRecord) and [`KeyValueDifferFactory`](https://angular.io/api/core/KeyValueDifferFactory) and [`KeyValueDiffers`](https://angular.io/api/core/KeyValueDiffers) service.

Finally, we implemented it all in the code and tracked the changes made to the `rates` object over time using [`KeyValueChanges.forEachChangedItem`](https://angular.io/api/core/KeyValueChanges#forEachChangedItem).

This strategy is also used by Angular’s built-in directive [`ngStyle`](https://angular.io/api/common/NgStyle), you can check it’s code on [GitHub](https://github.com/angular/angular/blob/master/packages/common/src/directives/ng_style.ts).

In this tutorial, we learned about tracking changes made to an object. It is also possible to track changes made to an array. For that, you will need to use [`IterableDiffers`](https://angular.io/api/core/IterableDiffers) service and related interfaces in the same manner. For more on it, checkout [`ngClass`](https://angular.io/api/common/NgClass)’s code on [GitHub](https://github.com/angular/angular/blob/master/packages/common/src/directives/ng_class.ts), where the Angular team have used `IterableDiffers`.
