---
title: How to manage object in Angular FormControl - Angular Tutorials | indepth.dev
author_name: Dharmen Shah
author_link: https://twitter.com/shhdharmen
discussion_link: https://github.com/indepth-dev/community/discussions/174
display_name: How to manage object in Angular FormControl
---

# How to manage object in Angular FormControl

### In this tutorial, we will learn how to manage object in FormControl and handle conversion for input

Generally we use `FormControl` with either `string` or `boolean` types and hence it manages only simple values. But what if we want to manage just more than primitive data types? We can do that, let’s see how.

For example, in this tutorial we will learn how to get country-code and number as separate entities from the same form-control, and we will also create custom input for the same to handle the conversion between UI and value.


## The Telephone Class

Let’s assume that we have a `Telephone` class, which will hold the related values:

```typescript
export class Telephone {
  constructor(
    public countryCode: string,
    public phoneNumber: string
  ) {}
}
```


Next, let’s take an `input[type=tel]` in which we want to hold the `Telephone`:

```typescript
@Component({
  selector: "app-root",
  template: `
    <input
      type="tel"
      name="telephone"
      id="telephone"
      [formControl]="telephone"
    />
    <div>Value: {{ telephone.value | json }}</div>
  `
})
export class AppComponent {
  telephone = new FormControl(new Telephone("", ""));
}
```

Notice how we have initialized the new `Telephone` class in `FormControl`:

Let’s take a look at output now:

![Form control with object](https://i.imgur.com/9O98N6z.gif "Form control with object")

The first thing you will notice is that `input` is showing `[object] [Object]` in it’s value, because it's a string representation of the object (in our case, it's a member of `Telephone` class). To fix `input`’s value, we can simply provide the `toString()` method in the `Telephone` class. You can read more about it on [MDN docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/toString).

```typescript
export class Telephone {
  constructor(public countryCode: string, public phoneNumber: string) {}
  toString() {
    return this.countryCode && this.phoneNumber
      ? `${this.countryCode}-${this.phoneNumber}`
      : "";
  }
}
```

The second thing is that `FormControl`’s value has the desired `Telephone`’s structure initially.

But, if you modify the `input`, `FormControl`’s value will change to `string`. We will need to work on the value conversion from UI to `FormControl`.

For that, we will create a custom input directive for `input[type=tel]` using `CustomValueAccessor`.

## Custom Input for Telephone

The initial code for `InputTelDirective` directive looks like below:

```typescript
// src/app/shared/directives/input-tel.directive.ts

import { Directive } from '@angular/core';
import { ControlValueAccessor, NG_VALUE_ACCESSOR } from '@angular/forms';

@Directive({
  selector: 'input[type=tel]',
  providers: [
    {
      provide: NG_VALUE_ACCESSOR,
      useExisting: InputTelDirective,
      multi: true
    }
  ]
})
export class InputTelDirective implements ControlValueAccessor {
  constructor() {}
  writeValue(obj: any): void {}
  registerOnChange(fn: any): void {}
  registerOnTouched(fn: any): void {}
}
```

_If you’re new to `ControlValueAccessor`, you can read more about it at: [Never again be confused when implementing ControlValueAccessor in Angular forms](https://indepth.dev/posts/1055/never-again-be-confused-when-implementing-controlvalueaccessor-in-angular-forms) and [How to use ControlValueAccessor to enhance date input with automatic conversion and validation](https://indepth.dev/posts/1467/how-to-use-controlvalueaccessor-to-enhance-date-input-with-automatic-conversion-and-validation)_

For this example, we are only concerned about `writeValue` and `registerOnChange`. Simply put, `writeValue` is used to convert `FormControl`’s value to UI value and `registerOnChange` is used to convert UI value to `FormControl`’s value.

### Conversion from UI to `FormControl`

We are going to assume that the user will enter the value in this form: `+CC-XXXXXX`, where characters before hyphen (-) combine to country-code and the rest is actual contact number. Note that this is not intended to be a robust directive, just something from which we can start learning.

To handle that, let’s first add a listener on `input` event:

```typescript
@HostListener("input", ["$event.target.value"])
  onInput = (_: any) => {};
```

Next, let’s modify the `registerOnChange` method:

```typescript
registerOnChange(fn: any): void {
    this.onInput = (value: string) => {
      let telephoneValues = value.split("-");
      const telephone = new Telephone(
        telephoneValues[0],
        telephoneValues[1] || ""
      );
      fn(telephone);
    };
  }
```

Let’s look at the output now:

![Form control with Custom Value Accessor](https://i.imgur.com/0DobyB5.gif "Form control with Custom Value Accessor")

Works nice! It is converting UI value to a valid `FormControl`’s value, i.e. `Telephone` class’s member.

### Conversion from `FormControl` to UI

If you try to set the initial state through `FormControl`, it won’t reflect on `input`:

```typescript
telephone = new FormControl(new Telephone("+91", "1234567890"));
```

Let’s modify the `writeValue` method to handle the above:

```typescript
writeValue(value: Telephone | null): void {
    const telephone = value || new Telephone("", "");
    this._renderer.setAttribute(
      this._elementRef.nativeElement,
      "value",
      telephone.toString()
    );
  }
```

Now the value set through `FormControl` will get reflected to `input`.

### Validation

Now we will add the validation part so that input supports validation out-of-the box.

We will first add `NG_VALIDATORS` in providers:

```typescript
// src/app/shared/directives/input-tel.directive.ts

// …

@Directive({
  selector: 'input[type=tel]',
  providers: [
    {
      provide: NG_VALUE_ACCESSOR,
      useExisting: InputTelDirective,
      multi: true,
    },
    {
      provide: NG_VALIDATORS,
      useExisting: InputTelDirective,
      multi: true,
    },
  ],
})
```

Next, we will add `isValid` method in `Telephone` class to check validity:

```typescript
export class Telephone {
  // ...
  isValid() {
    return !!(this.countryCode && this.phoneNumber);
  }
}
```

Lastly, we will implement the `Validator` interface and add `validate` method:

```typescript
export class InputTelDirective implements ControlValueAccessor, Validator {

  // ...

  validate(control: AbstractControl): ValidationErrors | null {
    const telephone = control.value as Telephone;
    return telephone.isValid() ? null : { telephone: true };
  }
}
```

Let’s modify the template to utilize validation:

```typescript
@Component({
  selector: "app-root",
  template: `
    <input
      type="tel"
      name="telephone"
      id="telephone"
      [formControl]="telephone"
      [class.is-invalid]="
        (telephone?.touched || telephone?.dirty) && telephone?.invalid
      "
    />
    <div
      class="invalid-feedback"
      *ngIf="(telephone.touched || telephone.dirty) && telephone.invalid"
    >
      Invalid Telephone
    </div>
    <div>Value: {{ telephone.value | json }}</div>
  `
})
export class AppComponent {
  telephone = new FormControl(new Telephone("", ""));
}
```

Let's look at the output now:

![Form control with Validation](https://i.imgur.com/97sXQ1i.gif "Form control with Custom Validation")

## Conclusion

We learned how we can manage the object in `FormControl` and use `ControlValueAccessor` to handle the conversion.

The whole code for directive looks like below:

```typescript
import { Directive, ElementRef, HostListener, Renderer2 } from "@angular/core";
import {
  AbstractControl,
  ControlValueAccessor,
  NG_VALIDATORS,
  NG_VALUE_ACCESSOR,
  ValidationErrors,
  Validator,
} from "@angular/forms";

@Directive({
  selector: "input[type=tel]",
  providers: [
    {
      provide: NG_VALUE_ACCESSOR,
      useExisting: InputTelDirective,
      multi: true,
    },
    {
      provide: NG_VALIDATORS,
      useExisting: InputTelDirective,
      multi: true,
    },
  ],
})
export class InputTelDirective implements ControlValueAccessor, Validator {
  constructor(
    private _elementRef: ElementRef<HTMLInputElement>,
    private _renderer: Renderer2
  ) {}

  @HostListener("input", ["$event.target.value"])
  onInput = (_: any) => {};

  writeValue(value: Telephone | null): void {
    const telephone = value || new Telephone("", "");
    this._renderer.setAttribute(
      this._elementRef.nativeElement,
      "value",
      telephone.toString()
    );
  }

  registerOnChange(fn: any): void {
    this.onInput = (value: string) => {
      let telephoneValues = value.split("-");
      const telephone = new Telephone(
        telephoneValues[0],
        telephoneValues[1] || ""
      );
      fn(telephone);
    };
  }

  registerOnTouched(fn: any): void {}

  validate(control: AbstractControl): ValidationErrors | null {
    const telephone = control.value as Telephone;
    return telephone.isValid() ? null : { telephone: true };
  }
}

export class Telephone {
  constructor(public countryCode: string, public phoneNumber: string) {}
  toString() {
    return this.countryCode && this.phoneNumber
      ? `${this.countryCode}-${this.phoneNumber}`
      : "";
  }
  isValid() {
    return !!(this.countryCode && this.phoneNumber);
  }
}

```

I have also created a [GitHub repo](https://github.com/shhdharmen/multi-value-form-control) for all the code above.
