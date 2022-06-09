---
title: InDepth Guide for Customizing Angular Material Button - Angular Tutorials | indepth.dev
author_name: Dharmen Shah
author_link: https://twitter.com/shhdharmen
discussion_link: https://github.com/indepth-dev/community/discussions/273
display_name: InDepth Guide for Customizing Angular Material Button
---

# InDepth Guide for Customizing Angular Material Button

## Introduction

It is a very common need in almost all applications to modify the components provided by 3rd party libraries. Those modifications are generally done for:

1. Changing the styles
2. Adding a missing feature

And it is very crucial for such libraries to provide ways to achieve those modifications easily.

In the first part of this tutorial we will learn how to modify styles so that our changes do not conflict with future updates of Angular Material library. As a bonus, I will provide a list of thumb rules which you should follow while making any style changes.

In the second part of this tutorial, we will learn all the ways to customize angular material buttons and decide which is better.

By end of this article, you will have idea about below topics:

1. How to create attribute directive
2. How to create dynamic component
3. When to create component and when to create directive
4. How to effectively modify any 3rd party library’s components, for both, to add a feature and to change the styles

### Angular Material Button

Angular Material’s buttons are already enhanced with Material design and ink ripples, and they also have a range of presentation options.

You can change the look and feel of buttons by using different attribute directives for different situations and needs. For instance `mat-button` is a rectangular button with text content, `mat-raised-button` is the same as `mat-button`, but with elevation and `mat-icon-button` is circular and it’s meant to contain an icon. You can check all variants on the [official site](https://material.angular.io/components/button/overview).

And there are 3 theme colors provided with all variants: `primary`, `accent` and `warn`.

Now, even with these many variants and options, we may need to modify the default Angular Material button to achieve a feature or change of style. Let’s look and learn how to make both the changes effectively.

## How to change styles

Before moving into how to change styles, let’s first understand some rules to avoid any conflicting changes. These rules are consolidated from [customizing component styles guidelines](https://material.angular.io/guide/customizing-component-styles).

### Thumb rules

Keep below rules in mind whenever you want to change styles of any Angular Material component.

1. Define custom styles for a component’s host element
2. Change styles which affect either position or layout of that component
   1. `margin`, `position`, `top`, `left`, `transform`, `z-index`, etc.
3. Apply above styles modifications by defining a custom CSS class and applying it to component’s host element
4. Do not change the styles which affect size or internal layout of the component
   1. `padding`, `height`, `width`, or `overflow`
5. Do not change or override the styles of internal elements of Angular Material components, like in Angular Material button, there are some internal components which produce ripple effect, we should avoid modifying styles of such components
6. Provide custom styles to overlay components, like `MatDialog`, `MatMenu`, etc. through `panelClass` property. Add that class to your global stylesheet after including theme mixins.

By following above rules, not just for Angular Material components but any component library, we can safely achieve needed modifications and avoid causing breaking styles.

Now, keeping the above rules in mind, we will try to change styles of Angular Material buttons. For this tutorial, we will focus on below 2 styles changes:

1. Color
2. Shape

And at the end of the section we will also have a brief look for size and typography.

### Color

The very basic change we may need to `font-color` and `background-color` of buttons. And that, too with different states, like `:hover`, `:focus` and `:active`.

Some time back I wrote an article about “Angular Material Theming System: Complete Guide” and in that I wrote a section titled “[Customizing Angular Material Component Styles](https://indepth.dev/tutorials/angular/angular-material-theming-system-complete-guide#customizing-angular-material-component-styles)”. In that section, I have explained how to modify the Angular Material button’s colors.

But, that approach was complex, difficult to read, hard to debug and not future safe. The reason behind that was I re-created many SASS functions and mixins, which are supposed to be used by only internal elements of buttons and used them to achieve desired changes. Now, if the Angular Material team plans to change any internal styles, those modifications will break.

So, let’s look at a more easy and recommended way to achieve color modifications.

Let’s assume that you have [added Angular Material](https://material.angular.io/guide/getting-started) in your project and selected a pre-built theme to use.

At this time, your `style.scss` looks like below:

```scss
// styles.scss
html,
body {
  height: 100%;
}
body {
  margin: 0;
  font-family: Roboto, "Helvetica Neue", sans-serif;
}
```

We will create a custom-theme, which should get applied only when it’s inside `.custom-theme` class.

```scss
@use "@angular/material" as mat;

$custom-primary: mat.define-palette(mat.$pink-palette, 700, 500, 900);
$custom-accent: mat.define-palette(mat.$blue-grey-palette, A200, A100, A400);

$custom-theme: mat.define-dark-theme(
  (
    color: (
      primary: $custom-primary,
      accent: $custom-accent,
    ),
  )
);

.custom-theme {
  @include mat.button-theme($custom-theme);
  @include mat.progress-spinner-theme($custom-theme);
}
```

Note that we have only included `button-theme` and `progress-spinner-theme`, because in our demo we only use those 2 components. You can also use `all-component-theme` mixin to add all components’ themes, but it will increase the size of the final output style. For a more detailed understanding, please refer to the article “[Angular Material Theming System: Complete Guide](https://indepth.dev/tutorials/angular/angular-material-theming-system-complete-guide)”.

So, now with the above code, if in the HTML code, we simply wrap the main container with `custom-theme` class, it will apply custom-theme to components inside it. Let’s look at the output:

![buttons with default and custom theme](https://imgur.com/DFVkhQq.gif "buttons with default and custom theme")

### Shape

Next, let’s change the shape. We want to add a shape variant such that buttons have a rounded borders.

Now, according to thumb-rules discussed earlier, we can change the styles of host-element which affect the layout of the component itself. So, to achieve the shape change, we can simply add a class with needed changes and apply it safely to Angular Material buttons:

```scss
.button-rounded {
  border-radius: 25% / 50%;
}
```

Now, if you apply the class `button-rounded`, you won’t see the change. The reason behind that is all variants of Angular Material buttons have their own `border-radius` already applied like below:

```scss
.mat-raised-button {
    // rest
    border-radius: 4px;
}
```

So, considering [selector specificity](https://developer.mozilla.org/en-US/docs/Web/CSS/Specificity), we will have to modify our code like below:

```scss
.button-rounded {
  &.mat-button,
  &.mat-raised-button,
  &.mat-flat-button {
    border-radius: 25% / 50%;
  }
}
```

Now, it will look perfect in the output:

![raised and rounded](https://imgur.com/C0tydUx.gif "raised and rounded")

### Other styles changes

Apart from color and size, there can be more changes needed. Let’s briefly look at some and how to modify them.

#### Size

Modifications of size are not recommended, because it violates our thumb rules. Size customizations can lead to breaking changes with future updates.

And the Angular Material team has already followed [material guidelines](https://material.io/design/layout/spacing-methods.html) for the size, which we should not change.

#### Typography

This can be easily changed by using standard Angular Material theme mixins.

```scss
$custom-theme: mat.define-light-theme((
   color: (
     primary: $custom-primary,
     accent: $custom-accent,
   ),
   typography: $custom-typography,
  ));
```

For more details, see "[Modify typography](https://indepth.dev/tutorials/angular/angular-material-theming-system-complete-guide#modify-typography)".

Next, we will look into how to add a spinner in the button.

## How to add `MatProgressSpinner`

As mentioned in the heading, we will show a `MatProgressSpinner` when `loading` is set with Angular Material’s button.

Now, there are 3 ways to achieve that. Let’s look at them below and what are the pros and cons of each.

1. Template Interpolation
2. Wrapper Component
3. Directive

### Template Interpolation

For template interpolation, your code may look like this:

```html
<button mat-button [disabled]="isLoading">
    <mat-spinner *ngIf="isLoading"></mat-spinner>
    Action
</button>
```

#### Pros

1. The main advantage of above code is quick, readable and easy to customize as and when needed.

#### Cons

1. Repetition: Above code is fine, but you will have to repeat the same lines and conditions at all places wherever you want to show `MatProgressSpinner` in buttons.
2. Changes at all places: If you want to change something, for example size of `MatProgressSpinner`, you will have to find out all such instances and do the change.

### Wrapper Component

Another approach and to overcome challenges faced with template interpolation, we can think of proceeding with creating a wrapper component with needed inputs, like below:

```typescript
@Component({
    selector: 'app-button',
    template: `
        <button mat-button>
            <mat-spinner *ngIf="loading"></mat-spinner>
            <ng-content></ng-content>
        </button>
    `
})
export class AppButtonComponent {
    @Input() loading: boolean;
}
```

#### Pros

1. Changes at all places: With the above, now you can use `app-button` everywhere to get the same button with `mat-spinner`.
2. Reusability: And if you want to change anything, you just need to change in this component and it will reflect at all places.
3. Customizations: As we are using component, we can make template customizations easily

#### Cons

1. Native component properties: Let’s assume that at different places, we want to use different variants of the Angular Material button. Now for color, you can simply add one more input and get all the variants of color. But if you want to use different presentations, like `mat-flat-button` or `mat-icon-button`, things will start becoming more complex.
2. Events: Apart from variants, you will also have to handle events, like `(click)`. You will have to propagate the click event using `@Output()` up to its parent component.
3. Directive support: Angular Material button supports it’s own `MatTooltip` and `MatBadge` directives out of the box. To achieve support of all of the above in a wrapper component is not only difficult but complex and hard to maintain.

### Directive

With directive, we will first start with an `input` of `loading` state, which will show/hide `MatProgressSpinner` and also disable/enable the `MatButton`. Let’s start with basic code:

```typescript
@Directive({
  selector: `button`,
})
export class ButtonDirective implements OnChanges {
  @Input() loading = false;

  constructor() {}

  ngOnChanges(changes: SimpleChanges): void {
    if (!changes['loading']) {
      return;
    }

    // Create/destroy spinner
  }

  private createSpinner(): void {}

  private destroySpinner(): void {}
}
```

In the above code, we are creating a directive with tag selector, so that it works with all `&lt;button>`s. We have added an `@Input()` called `loading`, which will show/hide the spinner inside button.

Now, to show the spinner, we are going to create the MatProgressSpinner` component dynamically and will place it inside the `button` when `loading` is set to true.

```typescript
@Directive({
  selector: `button`,
})
export class ButtonDirective implements OnChanges {

  private spinner!: ComponentRef<MatProgressSpinner> | null;

  ngOnChanges(changes: SimpleChanges): void {
    if (!changes['loading']) {
      return;
    }

    if (changes['loading'].currentValue) {
        // disable the `MatButton`
        this.createSpinner();
      } else if (!changes['loading'].firstChange) {
        // enable the `MatButton`
        this.destroySpinner();
      }
  }
}
```

Above code is simple, we are creating and destroying spinner based on `loading`’s current value.

```typescript
@Directive({
  selector: `button`,
})
export class ButtonDirective implements OnChanges {

  @Input() color: ThemePalette;

  constructor(
    private matButton: MatButton,
    private viewContainerRef: ViewContainerRef,
    private renderer: Renderer2
  ) {}

  private createSpinner(): void {
    if (!this.spinner) {
      this.spinner = this.viewContainerRef.createComponent(MatProgressSpinner);
      this.spinner.instance.color = this.color;
      this.spinner.instance.diameter = 20;
      this.spinner.instance.mode = 'indeterminate';
      this.renderer.appendChild(
        this.matButton._elementRef.nativeElement,
        this.spinner.instance._elementRef.nativeElement
      );
    }
  }

  private destroySpinner(): void {
    if (this.spinner) {
      this.spinner.destroy();
      this.spinner = null;
    }
  }
}
```

In the above code, first we added an `@Input()` to read the current `color`. We will use this property to set the color of the spinner.

Then, we provided `MatButton`, `ViewContainerRef` and `Renderer2` classes in the constructor.

In the `createSpinner` method, we are simply creating the `MatProgressSpinner` dynamically and storing its reference in `spinner`, so that we can destroy it later on. Notice how we created it dynamically:

```typescript
this.spinner = this.viewContainerRef.createComponent(MatProgressSpinner);
```

And after creating, we are appending it to the HTML element of `MatButton`, with help of `Renderer2`:

```typescript
      this.renderer.appendChild(
        this.matButton._elementRef.nativeElement,
        this.spinner.instance._elementRef.nativeElement
      );
```

And at last, in the `destroySpinner` method, we are destroying the `spinner` component and cleaning it up by assigning `null` value.

```typescript
@Directive({
  selector: `button`,
})
export class ButtonDirective implements OnChanges {

  @Input() disabled = false;

  ngOnChanges(changes: SimpleChanges): void {

  // ...

  if (changes['loading'].currentValue) {
      this.matButton._elementRef.nativeElement.classList.add('button-loading');
      this.matButton.disabled = true;
      this.createSpinner();
    } else if (!changes['loading'].firstChange) {
      this.matButton._elementRef.nativeElement.classList.remove(
        'button-loading'
      );
      this.matButton.disabled = this.disabled;
      this.destroySpinner();
    }
  }
}
```

The last part is to make the `MatButtton` disabled when `loading` is `true`. Apart from disabling, we are also toggling a class `button-loading` with it to achieve the desired styles.

Below is the styles code for `button-loading` class:

```scss
.button-loading {
  .mat-button-wrapper {
    visibility: hidden;
  }

  .mat-progress-spinner {
    position: absolute;
    top: calc(50% - 10px);
    left: calc(50% - 10px);
  }
}
```

And the final code for directive looks like below:

```typescript
@Directive({
  selector: `button`,
})
export class ButtonDirective implements OnChanges {
  private spinner!: ComponentRef<MatProgressSpinner> | null;

  @Input() loading = false;
  @Input() disabled = false;
  @Input() color: ThemePalette;

  constructor(
    private matButton: MatButton,
    private viewContainerRef: ViewContainerRef,
    private renderer: Renderer2
  ) {}

  ngOnChanges(changes: SimpleChanges): void {
    if (!changes['loading']) {
      return;
    }

    if (changes['loading'].currentValue) {
      this.matButton._elementRef.nativeElement.classList.add('button-loading');
      this.matButton.disabled = true;
      this.createSpinner();
    } else if (!changes['loading'].firstChange) {
      this.matButton._elementRef.nativeElement.classList.remove(
        'button-loading'
      );
      this.matButton.disabled = this.disabled;
      this.destroySpinner();
    }
  }

  private createSpinner(): void {
    if (!this.spinner) {
      this.spinner = this.viewContainerRef.createComponent(MatProgressSpinner);
      this.spinner.instance.color = this.color;
      this.spinner.instance.diameter = 20;
      this.spinner.instance.mode = 'indeterminate';
      this.renderer.appendChild(
        this.matButton._elementRef.nativeElement,
        this.spinner.instance._elementRef.nativeElement
      );
    }
  }

  private destroySpinner(): void {
    if (this.spinner) {
      this.spinner.destroy();
      this.spinner = null;
    }
  }
}
```

_Above code is referenced from: [Button | Angular Material Extensions (ng-matero.github.io)](https://ng-matero.github.io/extensions/components/button/overview)_

Now, with Angular Material buttons, you just need to set `loading` to show a spinner inside of it. Let’s take a look at output:

![button with spinner](https://imgur.com/MGGe17R.gif "button with spinner")

Let’s look at the above approach’s pros and cons.

#### Pros

1. Native component properties: As you can see in the output, the directive works with all variants of `MatButton`
2. Events: Also, there is no need to write extra code handle event
3. Directive support: As we used directive, other library directives’ support, like `MatBadge`, `MatTooltip` still exists

#### Cons

1. No template control: We do not have template control with this approach compared to wrapper component and inline template interpolation
2. More DOM manipulation: As we do not have template control, we have to do every template change through DOM manipulation

So, compared with template interpolation and wrapper components, the reusability without losing default features is the main and biggest advantage of this approach. And that’s why, one should try to achieve such customizations with usage of directive.

## Conclusion

We started with understanding why and which customizations can be needed when using any 3rd party UI components library. Then we understood what Angular Material components library provides especially for buttons.

Next, we compared all the approaches mentioned below to add a spinner in Angular Material buttons:

1. Template interpolation -  quick and easy to understand, but reusability is missing
2. Wrapper component - reusability is achieved, but more complex code and setup required to keep support of default functionalities
3. Directive - support for default functionalities and reusability, both achieved with less control over template

Then, we understood some thumb rules to prevent our custom styling from breaking with major updates. Next we learned how to effectively modify color, size and typography. And why we shouldn’t modify the size of the Angular Material button.

I have uploaded the code at [GitHub](https://github.com/shhdharmen/indepth-customizing-angular-material-button), you can also take a look at it on [stackblitz](https://stackblitz.com/github/shhdharmen/indepth-customizing-angular-material-button).
