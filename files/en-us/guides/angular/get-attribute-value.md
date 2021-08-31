---
title: How to get an attribute value in Angular's components and directives - Angular Tutorials | indepth.dev
author_name: Dharmen Shah
author_link: https://twitter.com/shhdharmen
discussion_link: https://github.com/indepth-dev/community/discussions/151
---

# How to get an attribute value in Angular's components and directives


In this tutorial, we will explore all the ways to read HTML attribute values passed in the component or directive.


## Reading HTML Attributes

_Elements in HTML have **attributes**; these are additional values that configure the elements or adjust their behavior in various ways to meet the criteria the users want. - [MDN Docs](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes)_

For example:

```html
<input type="text">
<td colspan="3"></td>
```

`type`, and `colspan` are some examples of HTML attributes.

In some cases, you need to handle the behaviour of a component or directive based on a value set for the HTML attribute.

For example, there is a component called `smart-input`. You want to render different layouts for different `type`s passed as values to the HTML attribute.

```html
<!-- should render UI for text -->
<smart-input type="text"></smart-input>

<!-- should render UI for number -->
<smart-input type="number"></smart-input>
```

## `ElementRef` Class

One way to read such attribute values, is through [`ElementRef`](https://angular.io/api/core/ElementRef):

```typescript
import { Component, ElementRef } from '@angular/core';

@Component({
  selector: 'smart-input',
  template: `
    <div [ngSwitch]="type">
      <input type="text" *ngSwitchCase="'text'" />
      <input type="number" *ngSwitchCase="'number'" />
      <p *ngSwitchDefault>Unsupported type: {{ type }}</p>
    </div>
  `
})
export class SmartInputComponent {
  type: string;
  constructor(private elementRef: ElementRef) {
    this.type = this.elementRef.nativeElement.getAttribute('type');
  }
}
```

Above is fine and works as expected. But, there is a shorter way available for the above in Angular.

## The `@Attribute()` Parameter Decorator

You can use the [`Attribute`](https://angular.io/api/core/Attribute) parameter decorator to pass the value of the HTML attribute to the component or directive constructor through dependency injection.

So for an earlier example, to read the `type` attribute, we will do something like below:


```typescript
import { Attribute, Component } from '@angular/core';

@Component({
  selector: 'smart-input',
  template: `...`
})
export class SmartInputComponent {
  constructor(@Attribute('type') public type: string) {}
}
```


With the above code, `smart-input` will render

1. `<input type="text">` for `<smart-input type="text"></smart-input>`
2. `<input type="number">` for `<smart-input type="number"></smart-input>`

### Usage with providers

`@Attribute()` decorator can also be used in [`deps`](https://angular.io/api/core/FactorySansProvider#deps) when making a [`FactoryProvider`](https://angular.io/api/core/FactoryProvider).

So, for `smart-input`, we can also write our code like below:

```typescript
import { Component, Inject, InjectionToken, Attribute } from '@angular/core';

export const TOKEN = new InjectionToken<string>('TypeAttribute');

export function factory(token: string): string {
  return token;
}

@Component({
  selector: 'smart-input',
  template: `...`,
  providers: [
    {
      provide: TOKEN,
      deps: [[new Attribute('type')]],
      useFactory: factory
    }
  ]
})
export class SmartInputComponent {
  constructor(@Inject(TOKEN) readonly type: string) {}
}
```

### Usage in other built-in directives

This decorator is also used by few built-in directives:

1. [`RouterLink`](https://angular.io/api/router/RouterLink) reads `tabindex`
2. [`RouterOutlet`](https://angular.io/api/router/RouterOutlet) reads [`name`](https://angular.io/api/router/RouterOutlet#description)
3. [`ngPluralCase`](https://angular.io/api/common/NgPluralCase) reads self-value

### Limitations

Below are two limitations of using `@Attribute()` decorator:

1. It only works with static HTML attribute values. It doesn’t work with [attribute binding](https://angular.io/guide/attribute-binding#binding-to-an-attribute) and [property binding](https://angular.io/guide/property-binding).
2. As it only works with static values, only `string` type is supported.

To overcome the above limitations, we can use `@Input()` decorator.

## The `@Input()` Property Decorator

[`@Input()`](https://angular.io/api/core/Input) is a property decorator, used in the child component or directive, which signifies that property can receive its value from the parent component.

So, for the above example, we can rewrite it as below:

```typescript
@Component({
  selector: 'smart-input',
  template: `...`
})
export class SmartInputComponent {
  @Input() type: string;
}
```

Above works fine. And unlike `@Attribute()`, this supports all data-types and if you use [property binding](https://angular.io/guide/property-binding) or [text interpolation](https://angular.io/guide/interpolation#text-interpolation), it also tracks changes to the values.

Also note that, `@Input()` works with static and dynamic, both bindings.

### Usage in other built-in directives

This decorator is also used by many built-in directives. Below are a couple of examples:

1. [`ngClass`](https://angular.io/api/common/NgClass) to read self-value
2. [`ngStyle`](https://angular.io/api/common/NgStyle) to read self-value

### Limitation

The only limitation of using `@Input()` decorator is that it’s value is available only after component or directive is initialized, i.e. in [`ngOnInit`](https://angular.io/guide/lifecycle-hooks#oninit) life-cycle hook.

## `ElementRef` / `@Attribute()` vs `@Input()`

We learned three ways to read `type`’s value for our`smart-input` component, how they’re used in some built-in directives and limitations. Let’s look at the major differences again:

### Availability

1. `ElementRef` and `@Attribute()` makes attribute’s value available in `constructor` through dependency injection
2. `@Input()` makes the value available after component/directive initialization, i.e. in the `ngOnInit` life-cycle hook.

### Type support

1. `ElementRef` and `@Attribute()` supports only `string`
2. `@Input()` supports all types

### Value updates

1. `ElementRef` and `@Attribute()` don’t track value update
2. `@Input()` tracks value updates if property binding or text interpolation is used

### General usage

1. As `ElementRef` and `@Attribute()` supports only `string` type, they are generally used to read static HTML attribute’s value
2. As `@Input()` supports all types and it also tracks updates, it is generally used to read DOM property or custom data passed to the component/directive

## Conclusion

We learned how to read the HTML attribute value in a component or directive using the  `ElementRef` class, `@Attribute()` and `@Input()` decorators.

We also learned their usages, how they’re used in some built-in directives and limitations.

I have also created a [stackblitz](https://stackblitz.com/edit/angular-ivy-8cyazj?file=src/app/app.component.ts) for all of the code above.
