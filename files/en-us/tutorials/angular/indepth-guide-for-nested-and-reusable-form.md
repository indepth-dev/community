---
title: InDepth Guide for Creating Nested and Reusable Forms - Angular Tutorials | indepth.dev
author_name: Dharmen Shah
author_link: https://twitter.com/shhdharmen
discussion_link: https://github.com/indepth-dev/community/discussions/280
display_name: InDepth Guide for Creating Nested and Reusable Forms
---

# InDepth Guide for Creating Nested and Reusable Forms

In this tutorial you will learn how to create and use form-groups. And we will also learn how to achieve complete reliability by using the composite ControlValueAccessor.

## Introduction

In web applications, we can have large forms, like account registration form, profile creation form, credit card form, address form, etc. These large forms generally have a set of fields which are repeated in them.

For example, address fields are generally available in all the registration forms. In this tutorial, we will learn how to wrap such fields in a reusable form, so that it can be used as nested form any other form.

In this tutorial, we will discuss 4 approaches to create a form and we will also look at pros and cons for each:

1. How to create and use form-group
2. How to create nested form-groups
3. How to use child component with nested form-group
4. How to create reusable form with Composite `ControlValueAccessor`

## How to create and use FormGroup

Forms typically contain several related controls. [Reactive forms](https://angular.io/guide/reactive-forms) provide two ways of grouping multiple related controls into a single input form: [`FormGroup`](https://angular.io/api/forms/FormGroup) and [`FormArray`](https://angular.io/api/forms/FormArray). For this tutorial, we will focus on `FormGroup`.

Just as a form control instance gives you control over a single input field, a form group instance tracks the form state of a group of form control instances (for example, a form). Each control in a form group instance is tracked by name when creating the form group.

We will take an example of Profile form, which will have input fields for name, email, address lines, city, state, zip-code and country.

![profile form](https://i.imgur.com/NBmJUwT.png "profile form")

The form-group for the same looks like below:

```typescript
// src/app/simple-form-group/simple-form-group.component.ts

@Component({
  selector: "app-simple-form-group",
  templateUrl: "simple-form-group.component.html",
})
export class SimpleFormGroupComponent {
  profileFormGroup = new FormGroup<SimpleProfileForm>({
    name: new FormControl("", {
      validators: [Validators.required],
      nonNullable: true,
    }),
    email: new FormControl("", {
      validators: [Validators.email, Validators.required],
      nonNullable: true,
    }),
    line1: new FormControl("", {
      validators: [Validators.required],
      nonNullable: true,
    }),
    // other address form controls
  });

  readonly COUNTRIES = COUNTRIES;
  constructor() {}

  get nameFC() {
    return this.profileFormGroup.get("name");
  }
  get emailFC() {
    return this.profileFormGroup.get("email");
  }
  get line1FC() {
    return this.profileFormGroup.get("line1");
  }
  // other form-control getters
}

export type SimpleProfileForm = {
  name: FormControl<string>;
  email: FormControl<string>;
  line1: FormControl<string>;
  line2?: FormControl<string | null>;
  zipCode: FormControl<string>;
  city: FormControl<string>;
  state: FormControl<string>;
  country: FormControl<string>;
};
```

Notice that we are also taking advantage of the strictly typed forms feature, [which is introduced in Angular 14](https://github.com/angular/angular/pull/43834).

In the above code, we are simply creating a `FormGroup` with all the fields as [`FormControl`](https://angular.io/api/forms/FormControl). The HTML template code can look like below:

```html
<!-- src/app/simple-form-group/simple-form-group.component.html -->

<form [formGroup]="profileFormGroup">
  <h2>Basic Information</h2>
  <div>
    <label for="name">Name*</label>
    <input
      type="name"
      formControlName="name"
      id="name"
      required
      [ngClass]="{
        'is-invalid': nameFC?.invalid && (nameFC?.touched || nameFC?.dirty)
      }"
    />
    <div class="invalid-feedback">Name is required</div>
  </div>
  <!-- other basic information fields -->
  <div>
    <h2>Address</h2>
    <div>
      <label for="line1">Line 1*</label>
      <input
        [id]="'line1'"
        [name]="'line1'"
        autocomplete="address-line1"
        formControlName="line1"
        required
        type="text"
        [ngClass]="{
          'is-invalid': line1FC?.invalid && (line1FC?.touched || line1FC?.dirty)
        }"
      />
      <div class="invalid-feedback">Address line 1 is required</div>
    </div>
    <!-- other address fields -->
  </div>
  <button type="submit" [disabled]="profileFormGroup.invalid">
    Submit form
  </button>
</form>
```

The `formControlName` input provided by the [`FormControlName`](https://angular.io/api/forms/FormControlName) directive binds each individual input to the form control defined in `FormGroup`. The form controls communicate with their respective elements. They also communicate changes to the form group instance, which provides the source of truth for the model value.

### Pros

1. Simple and straight-forward template and form-group

### Cons

1. Difficult to create and maintain complex form models
2. Logical segregation is missing between set of fields

## How to create nested form-groups

When building complex forms, managing the different areas of information is easier in smaller sections. Using a nested form group instance lets you break large form groups into smaller, more manageable ones.

Let’s group address related fields in a nested group:

```typescript
// src/app/nested-from-group/nested-from-group.component.ts

@Component({
  selector: "app-nested-form-group",
  templateUrl: "nested-form-group.component.html",
})
export class NestedFormGroupComponent {
  profileFormGroup = new FormGroup<ProfileForm>({
    name: new FormControl("", {
      validators: [Validators.required],
      nonNullable: true,
    }),
    email: new FormControl("", {
      validators: [Validators.email, Validators.required],
      nonNullable: true,
    }),
    address: new FormGroup<AddressForm>({
      line1: new FormControl("", {
        validators: [Validators.required],
        nonNullable: true,
      }),
      // other address fields...
    }),
  });
}

type ProfileForm = {
  name: FormControl<string>;
  email: FormControl<string>;
  address: FormGroup<AddressForm>;
};

type AddressForm = {
  line1: FormControl<string>;
  line2?: FormControl<string | null>;
  zipCode: FormControl<string>;
  city: FormControl<string>;
  state: FormControl<string>;
  country: FormControl<string>;
};
```

Notice that now in `profileForm` address related fields are grouped in `address` element. We also created a `AddressForm` type to get better type safety, and better autocomplete in IDEs. Let’s group group nested form in the template:

```html
<!-- src/app/nested-from-group/nested-form-group.component.html -->

<form [formGroup]="profileFormGroup">
  <h2>Basic Information</h2>
  <!-- basic information fields -->
  <!-- notice the usage of formGroupName below -->
  <div formGroupName="address">
    <h2>Address</h2>
    <div>
      <label for="line1">Line 1*</label>
      <input
        [id]="'line1'"
        [name]="'line1'"
        autocomplete="address-line1"
        formControlName="line1"
        required
        type="text"
        [ngClass]="{
          'is-invalid': line1FC?.invalid && (line1FC?.touched || line1FC?.dirty)
        }"
      />
      <div class="invalid-feedback">Address line 1 is required</div>
    </div>
    <!-- other address related fields -->
  </div>
  <button type="submit" [disabled]="profileFormGroup.invalid">
    Submit form
  </button>
</form>
```

We added the [`formGroupName`](https://angular.io/api/forms/FormGroupName) directive to indicate that children form controls are part of `address` form-group.

### Pros

1. Easy to create and maintain complex form models
2. Logical segregation is present between set of fields

### Cons

1. Larger template due to more fields and complex form

## How to use child component with nested form-group

In the previous section, we achieved logical separation in class, but our template is still complex. We will create another component to handle template complexity.

```typescript
// src/app/nested-form-group-child/address-form/address-form.component.ts

@Component({
  selector: "app-address-form",
  templateUrl: "address-form.component.html",
})
export class AddressFormComponent {
  @Input("formGroup") addressFormGroup!: FormGroup;

  get line1FC() {
    return this.addressFormGroup.get("line1");
  }
  // other form-control getters...
}
```

We created `AddressFormComponent`. It has an input property called `formGroup`, through which parent components can pass the form-group. Let’s take a look at template:

```html
<!-- src/app/nested-form-group-child/address-form/address-form.component.html -->

<div [formGroup]="addressFormGroup">
  <h2>Address</h2>
  <div>
    <label for="line1">Line 1*</label>
    <input
      [id]="'line1'"
      [name]="'line1'"
      autocomplete="address-line1"
      formControlName="line1"
      required
      type="text"
      [ngClass]="{
        'is-invalid': line1FC?.invalid && (line1FC?.touched || line1FC?.dirty)
      }"
    />
    <div class="invalid-feedback">Address line 1 is required</div>
  </div>
  <!-- other address fields -->
</div>
```

The template is pretty simple, it just has the address related fields. And below is how we would use it in parent component:

```html
<!-- src/app/nested-form-group-child/nested-form-group-child.component.html -->

<form [formGroup]="profileFormGroup">
  <!-- basic fields →

  <!-- notice the usage of child component -->
  <app-address-form [formGroup]="addressFormGroup"></app-address-form>

  <button type="submit" [disabled]="profileFormGroup.invalid">
    Submit form
  </button>
</form>
```

And in the class, we would get the address form-group like below:

```typescript
// src/app/nested-from-group/nested-from-group.component.ts

@Component({
  // ...
})
export class NestedFormGroupChildComponent {
  profileFormGroup = new FormGroup<ProfileForm>({ /*...*/ });

  get addressFormGroup() {
    return this.profileFormGroup.get("address") as FormGroup;
  }
}
```

### Pros

1. Simpler templates
   a. With this approach now we can handle everything related to address in `AddressFormComponent` so that our main component is much simpler to handle.

### Cons

1. Not completely re-usable
   a. If you want to use this component as a stand-alone form component, it’s not possible for now

## How to create reusable form with Composite `ControlValueAccessor`

In the previous section, we created a component and moved the address form’s template into it. But, it’s still not completely re-suable. By completely re-usable what I mean is:

1. The logic, error handling and address form creation should be part of address-component and not the parent component
2. The component should be usable inside any form
3. The component should be usable with both, template driven and reactive forms
4. The component should support native form-control features, so that the parent or consumer component can take advantage of that, for example showing errors if overall address-form is invalid

In this section we will learn how to create a completely reusable form, that can be used as both, inside other forms and as a stand-alone form. And the technique which we are going to use is called “Composite ControlValueAccessor”, referred from [Kara Erickson in AngularConnect 2017](https://youtu.be/CD_t3m2WMM8?t=1523).

Our goal is to use the address form something like below:

```html
<!-- src/app/reusable-form/reusable-form-wrapper/reusable-form-wrapper.component.html -->

<form [formGroup]="profileFormGroup">
  <!-- other basic information fields -->
  <app-reusable-form formControlName="address"></app-reusable-form>
</form>
```

### ControlValueAccessor

A `ControlValueAccessor` acts as a bridge between the Angular forms API and a native element in the DOM. Any component or directive can be turned into form-control by implementing the `ControlValueAccessor` interface and registering itself as an `NG_VALUE_ACCESSOR` provider.

Among others the interface defines two important methods — `writeValue` and `registerOnChange`.

The writeValue method is used by formControl to set the value to the native form control. The registerOnChange method is used by formControl to register a callback that is expected to be triggered every time the native form control is updated. It is your responsibility to pass the updated value to this callback so that the value of respective Angular form control is updated. The registerOnTouched method is used to indicate that a user interacted with a control.

Here is the diagram that demonstrates an interaction:

![How ControlValueAccessor works](https://images.indepth.dev/images/2020/02/1_wvjxZqL4ZZVGmsh3VFV2Ew.jpeg "How ControlValueAccessor works")

_From [Never again be confused when implementing ControlValueAccessor in Angular forms - Angular inDepth](https://indepth.dev/posts/1055/never-again-be-confused-when-implementing-controlvalueaccessor-in-angular-forms)_

To learn more about `ControlValueAccessor`, you can read this [indepth guide](https://indepth.dev/posts/1055/never-again-be-confused-when-implementing-controlvalueaccessor-in-angular-forms). If you are looking for a practical example, you can refer to my earlier [article about date input](https://indepth.dev/posts/1467/how-to-use-controlvalueaccessor-to-enhance-date-input-with-automatic-conversion-and-validation) and [tutorial about managing object in form-control](https://indepth.dev/tutorials/angular/object-in-formcontrol).

### Composite ControlValueAccessors

In earlier articles and tutorials, we saw usage of `ControlValueAccessor` with one `&lt;input>`. But, the advantage of using `ControlValueAccessor` is you can have more inputs with it if you want.

We just need to handle all 4 required methods of that interface and it will just work, no matter how we handle it internally.

### Reusable Form

One thing from previous section is clear, we want to use the same template and internally, we will manage the fields’ states with a form-group, so our template will look like below:

```html
<!-- src/app/reusable-form/reusable-form.component.html -->

<div role="group" [formGroup]="form">
  <div>
    <label for="line1">Line 1*</label>
    <input
      id="line1"
      name="line1"
      autocomplete="address-line1"
      formControlName="line1"
      required
      type="text"
      [ngClass]="{
        'is-invalid': line1FC?.invalid && (line1FC?.touched || line1FC?.dirty)
      }"
    />
    <div class="invalid-feedback">Address line 1 is required</div>
  </div>
  <!-- other address fields -->
</div>
```

To manage the address object, we will create a class. This will be used in later stage in class of above template:

```typescript
// src/app/reusable-form/address.ts

export class Address {
  constructor(
    public line1: string = "",
    public zipCode: string = "",
    public city: string = "",
    public state: string = "",
    public country: string = "",
    public line2?: string | null
  ) {}

  isValid() {
    return (
      this.line1 && this.city && this.country && this.state && this.zipCode
    );
  }
}
```

We will have the above `Address` class as value for the form. Just like `string`, `number` and `boolean`, we can also have object stored in form-control. You can read more about it at the tutorial[ about managing object in form-control](https://indepth.dev/tutorials/angular/object-in-formcontrol).

And for validation, you can have your own logic to validate the address, but for now we will keep it simple. We will use this when we implement validation.

Next, based on previous articles about `ControlValueAccessor`, we will start with implementing `ControlValueAccessor` interface and the `NG_VALUE_ACCESSOR` as provider:

```typescript
// src/app/reusable-form/reusable-form.component.ts

@Component({
  selector: "app-reusable-form",
  templateUrl: "reusable-form.component.html",
  providers: [
    {
      provide: NG_VALUE_ACCESSOR,
      useExisting: ReusableFormComponent,
      multi: true,
    },
  ],
})
export class ReusableFormComponent implements ControlValueAccessor {
  form = new FormGroup<AddressForm>({
    line1: new FormControl("", {
      validators: Validators.required,
      nonNullable: true,
    }),
    // other address fields' form-controls
  });

  writeValue(value: Address | null): void {}

  registerOnChange(fn: (val: Partial<Address> | null) => void): void {}

  registerOnTouched(fn: () => void): void {}

  setDisabledState(disabled: boolean) {}
}
```

#### writeValue

```typescript
// src/app/reusable-form/reusable-form.component.ts

  writeValue(value: Address | null): void {
    const address = this.createAddress(value);
    this.form.patchValue(address);
  }

  private createAddress(value: Partial<Address> | null) {
    return new Address(
      value?.line1 || "",
      value?.zipCode || "",
      value?.city || "",
      value?.state || "",
      value?.country || "",
      value?.line2
    );
  }
```

In this, we will first convert the value coming in to the instance of `Address` and then, will call `form.patchValue` and it will update all the fields, of which values are updated.

#### registerOnChange

```typescript
// src/app/reusable-form/reusable-form.component.ts

registerOnChange(fn: (val: Partial<Address> | null) => void): void {
    this.form.valueChanges.subscribe((value) => {
      const address = this.createAddress(value);
      fn(address);
    });
  }
```

We are using `valueChanges` observable to handle `registerOnChange`. `valueChanges` observable emits for all the child form-fields, so it is a perfect fit for `registerOnChange`.

Now, as we are listening for `valueChanges`, we will have to revisit and fix `writeValue`’s `patchValue`, because everytime `patchValue` is called, `valueChanges` will trigger, we don’t want that.

```typescript
// src/app/reusable-form/reusable-form.component.ts

  writeValue(value: Address | null): void {
    const address = this.createAddress(value);
    this.form.patchValue(address, { emitEvent: false });
  }
```

As you notice above, we are setting `emitEvent` to `false`, so that it doesn’t trigger `valueChanges`.

#### registerOnTouched

```typescript
// src/app/reusable-form/reusable-form.component.ts

onTouched: any;

registerOnTouched(fn: () => void): void {
    this.onTouched = fn;
  }
```

We want the form to be touched, anytime any child form-control is touched. So, we will call `onTouched` in the template as well, with all child form-controls:

```html
<!-- src/app/reusable-form/reusable-form.component.html -->

<div role="group" [formGroup]="form">
  <div>
    <label for="line1">Line 1*</label>
    <!-- removed other attributes/properties for brevity -->
    <input
      (blur)="onTouched()"
    />
  </div>
  <!-- other address fields -->
</div>
```

#### setDisabledState

```typescript
// src/app/reusable-form/reusable-form.component.ts

setDisabledState(disabled: boolean) {
    disabled ? this.form.disable() : this.form.enable();
  }
```

### Validation

To handle validation, we will first add provider, and implement the `Validator` interface:

```typescript
// src/app/reusable-form/reusable-form.component.ts

@Component({
  // ...
  providers: [
    // ...
    {
      provide: NG_VALIDATORS,
      useExisting: ReusableFormComponent,
      multi: true,
    },
  ],
})
export class ReusableFormComponent implements ControlValueAccessor, Validator {

  // ...
  validate(control: AbstractControl<Address>): ValidationErrors | null {}
}
```

#### validate

```typescript
// src/app/reusable-form/reusable-form.component.ts

validate(control: AbstractControl<Address>): ValidationErrors | null {
    const value = control.value;
    return value && value.isValid() ? null : { address: true };
  }
```

Here we are going to use the `Address` class’s `isValid` method and based on that we will return the error.

### Multiple instances

Although our form is ready now to be used, there is still one issue with it. If this component is being used within the same UI, then the form controls may not work properly, because they all will have have same `name` and `id` attributes. And, this component itself doesn’t have `id`, which can be helpful for testing.

To fix this, first let’s introduce an [`host`](https://angular.io/api/core/Directive#host) property called `id`:

```typescript
// src/app/reusable-form/reusable-form.component.ts

@Component({
  // ...
  host: {
    "[id]": "id",
  },
})
export class ReusableFormComponent {
  id = `address-input`;
}
```

Next, we will add a static counter, which we will increment with each instance of this component:

```typescript
// src/app/reusable-form/reusable-form.component.ts

@Component({
  // ...
})
export class ReusableFormComponent {

  static nextId = 0;
  id = `address-input-${ReusableFormComponent.nextId++}`;

}
```

And lastly, we will use this `id` with all `input`s’ `id` and `name` attributes. And also in their `&lt;label>`s’ `for` attribute:

```html
<!-- src/app/reusable-form/reusable-form.component.html -->

<div role="group" [formGroup]="form">
  <div>
    <label [htmlFor]="id + '-line1'">Line 1*</label>
    <!-- other attributes removed for brevity -->
    <input
      [id]="id + '-line1'"
      [name]="id + '-line1'"
    />
  </div>
  <!-- other address fields -->
</div>
```

With this, our completely reusable address form is ready.

## Conclusion

In this tutorial we learned about how to create and use form-groups. Then we learned about how to create nested form-groups and how to use it with child components. And with each of those approaches, we also learned their pros and cons.

In the last, but very important section, we learned about how to create a completely reusable form component with composite `ControlValueAccessor`.

I have uploaded the code at [GitHub](https://github.com/shhdharmen/nested-reusable-form), you can also take a look at it on [stackblitz](https://stackblitz.com/github/shhdharmen/nested-reusable-form).
