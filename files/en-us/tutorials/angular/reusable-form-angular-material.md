---
title: Creating a custom form field control group using Angular Material - Angular Tutorials | indepth.dev
author_name: Dharmen Shah
author_link: https://twitter.com/shhdharmen
discussion_link: TBA
display_name: Creating a custom form field control group using Angular Material
---

# Creating a custom form field control group using Angular Material

In this tutorial, we will learn that it is possible to create custom form field controls group that can be used inside `<mat-form-field>`.

## Introduction

In one of [my last tutorial](https://indepth.dev/tutorials/angular/indepth-guide-for-nested-and-reusable-form), we learned how to create a completely reusable forms Angular.

In this guide we'll learn how to create a custom form for inputting address information and hook it up to work with `<mat-form-field>`.

## How to build custom form group using Angular Material

### Basic Address Form Component

In order to learn how to build custom form field controls, let's start with a simple form component that we want to work inside the form field. An address form that segments the parts of the address into their own form field inputs.

I created this form quickly using Angular Material schematic, simply run below command to create one:

```bash
ng generate @angular/material:address-form shared/address-form
```

It will generate template and class files like below:

```html
<!-- src\app\shared\address-form\address-form.component.html -->

<form [formGroup]="addressForm" novalidate (ngSubmit)="onSubmit()">
  <mat-card class="shipping-card">
    <mat-card-header>
      <mat-card-title>Shipping Information</mat-card-title>
    </mat-card-header>
    <mat-card-content>
      <div class="row">
        <div class="col">
          <mat-form-field>
            <input matInput placeholder="Company" formControlName="company">
          </mat-form-field>
        </div>
      </div>
      <div class="row">
        <div class="col">
          <mat-form-field>
            <input matInput placeholder="First name" formControlName="firstName">
            <mat-error *ngIf="addressForm.controls['firstName'].hasError('required')">
              First name is <strong>required</strong>
            </mat-error>
          </mat-form-field>
        </div>
        <div class="col">
          <mat-form-field>
            <input matInput placeholder="Last name" formControlName="lastName">
            <mat-error *ngIf="addressForm.controls['lastName'].hasError('required')">
              Last name is <strong>required</strong>
            </mat-error>
          </mat-form-field>
        </div>
      </div>
      <div class="row">
        <div class="col">
          <mat-form-field>
            <textarea matInput placeholder="Address" formControlName="address"></textarea>
            <mat-error *ngIf="addressForm.controls['address'].hasError('required')">
              Address is <strong>required</strong>
            </mat-error>
          </mat-form-field>
        </div>
      </div>
      <!-- Other form fields -->
    </mat-card-content>
    <mat-card-actions>
      <button mat-raised-button color="primary" type="submit">Submit</button>
    </mat-card-actions>
  </mat-card>
</form>

```

```ts
// src\app\shared\address-form\address-form.component.ts

@Component({
  selector: 'app-address-form',
  templateUrl: './address-form.component.html',
  styleUrls: ['./address-form.component.css']
})
export class AddressFormComponent {
  private fb = inject(FormBuilder);
  addressForm = this.fb.group({
    company: null,
    firstName: [null, Validators.required],
    lastName: [null, Validators.required],
    address: [null, Validators.required],
    address2: null,
    city: [null, Validators.required],
    state: [null, Validators.required],
    postalCode: [null, Validators.compose([
      Validators.required, Validators.minLength(5), Validators.maxLength(5)])
    ],
    shipping: ['free', Validators.required]
  });

  hasUnitNumber = false;

  // rest of auto generated code
}
```

Now, in above code please make a few changes as listed below:
1. Remove usage of `<mat-card>` so that parent or consumer component has more control over UI.
2. Remove submit button, becuase this form is not going be used as a stand-alone form, rather this will be always used insider some other form.
3. Change all the fields' appearance to `outline`
4. Create a class called `AddressForm` and use it to create typed forms

So, now after the 1st, 2nd and 3rd change, the template will look like below:

```html
<!-- src\app\shared\address-form\address-form.component.html -->

<form
  novalidate
  [formGroup]="addressForm"
  class="address-form-container"
>
  <div class="row">
    <div class="col">
      <mat-form-field appearance="outline">
        <input
          matInput
          placeholder="Company"
          formControlName="company"
        />
      </mat-form-field>
    </div>
  </div>
  <div class="row">
    <div class="col">
      <mat-form-field appearance="outline">
        <input
          matInput
          placeholder="First name"
          formControlName="firstName"
        />
        <mat-error
          *ngIf="addressForm.controls['firstName'].hasError('required')"
        >
          First name is <strong>required</strong>
        </mat-error>
      </mat-form-field>
    </div>
    <div class="col">
      <mat-form-field appearance="outline">
        <input
          matInput
          placeholder="Last name"
          formControlName="lastName"
        />
        <mat-error
          *ngIf="addressForm.controls['lastName'].hasError('required')"
        >
          Last name is <strong>required</strong>
        </mat-error>
      </mat-form-field>
    </div>
  </div>
  <div class="row">
    <div class="col">
      <mat-form-field appearance="outline">
        <textarea
          matInput
          placeholder="Address"
          formControlName="address"
        ></textarea>
        <mat-error *ngIf="addressForm.controls['address'].hasError('required')">
          Address is <strong>required</strong>
        </mat-error>
      </mat-form-field>
    </div>
  </div>
  <!-- Other address fields, submit button removed -->
</form>

```

And after 4th change, the will have a new file called `address.ts`:

```ts
// src\app\shared\address-form\address.ts

import { FormControl } from '@angular/forms';

export class Address {
  constructor(
    public company: string | null = null,
    public firstName: string = '',
    public lastName: string = '',
    public address: string = '',
    public address2: string | null,
    public city: string = '',
    public state: string = '',
    public postalCode: number = 0,
    public shipping: AddressShipping = 'free'
  ) {}
}

export type AddressForm = {
  company: FormControl<string | null>;
  firstName: FormControl<string>;
  lastName: FormControl<string>;
  address: FormControl<string>;
  address2: FormControl<string | null>;
  city: FormControl<string>;
  state: FormControl<string>;
  postalCode: FormControl<number>;
  shipping: FormControl<AddressShipping>;
};

export type AddressShipping = 'free' | 'priority' | 'nextday';

```

and the component's class file will look like below:

```ts
// src\app\shared\address-form\address-form.component.ts

@Component({
  selector: 'app-address-form',
  templateUrl: './address-form.component.html',
  styleUrls: ['./address-form.component.css']
})
export class AddressFormComponent {
  private fb = inject(FormBuilder);

  readonly freeShipping: AddressShipping = 'free';
  
  addressForm = this.fb.group<AddressForm>({
    company: this.fb.control(null),
    firstName: this.fb.control('', { nonNullable: true, validators: [Validators.required]}),
    lastName: this.fb.control('', { nonNullable: true, validators: [Validators.required]}),
    address: this.fb.control('', { nonNullable: true, validators: [Validators.required]}),
    address2: this.fb.control(null),
    city: this.fb.control('', { nonNullable: true, validators: [Validators.required]}),
    state: this.fb.control('', { nonNullable: true, validators: [Validators.required]}),
    postalCode: this.fb.control(0, { nonNullable: true, validators: Validators.compose([
      Validators.required, Validators.minLength(5), Validators.maxLength(5)]) }
    ),
    shipping: this.fb.control(this.freeShipping, { nonNullable: true, validators: Validators.required})
  });

  // rest of auto generated code
}
```

### Providing our component as a `MatFormFieldControl`

The first step is to provide our new component as an implementation of the `MatFormFieldControl` interface that the `<mat-form-field>` knows how to work with. To do this, we will have our class implement `MatFormFieldControl`. Since this is a generic interface, we'll need to include a type parameter indicating the type of data our control will work with, in this case `Address`. We then add a provider to our component so that the form field will be able to inject it as a `MatFormFieldControl`.

```ts
@Component({
  ...
  providers: [
    { provide: MatFormFieldControl, useExisting: AddressFormComponent },
  ],
})
export class AddressFormComponent implements MatFormFieldControl<Address> {
  ...
}
```

This sets up our component, so it can work with `<mat-form-field>`, but now we need to implement the various methods and properties declared by the interface we just implemented. To learn more about the `MatFormFieldControl` interface, see the [form field API documentation](https://material.angular.io/components/form-field/api).

### Implementing the methods and properties of MatFormFieldControl

#### `value`

This property allows someone to set or get the value of our control. Its type should be the same type we used for the type parameter when we implemented `MatFormFieldControl`. Since our component already has a value property, we don't need to do anything for this one.

#### `stateChanges`

Because the `<mat-form-field>` uses the `OnPush` change detection strategy, we need to let it know when something happens in the form field control that may require the form field to run change detection. We do this via the `stateChanges` property. So far the only thing the form field needs to know about is when the value changes. We'll need to emit on the stateChanges stream when that happens, and as we continue flushing out these properties we'll likely find more places we need to emit. We should also make sure to complete `stateChanges` when our component is destroyed.

```ts
stateChanges = new Subject<void>();

set value(address: Address | null) {
  address = address || new Address('', '', '', '', '', '', '', 0, this.freeShipping);
  this.addressForm.setValue(address);

  this.stateChanges.next();
}

ngOnDestroy() {
  this.stateChanges.complete();
}
```

#### `id`

This property should return the ID of an element in the component's template that we want the `<mat-form-field>` to associate all of its labels and hints with. In this case, we'll use the host element and just generate a unique ID for it.

```ts
static nextId = 0;

@HostBinding() id = `address-form-${AddressFormComponent.nextId++}`;
```

#### `placeholder`

This property allows us to tell the `<mat-form-field>` what to use as a placeholder. In this example, we'll do the same thing as `matInput` and `<mat-select>` and allow the user to specify it via an `@Input()`. Since the value of the placeholder may change over time, we need to make sure to trigger change detection in the parent form field by emitting on the `stateChanges` stream when the placeholder changes.

```ts
@Input()
get placeholder() {
  return this._placeholder;
}
set placeholder(plh) {
  this._placeholder = plh;
  this.stateChanges.next();
}
private _placeholder: string;
```

#### `ngControl` and `ControlValueAccessor`

This property allows the form field control to specify the `@angular/forms` control that is bound to this component.

Let's implement `ControlValueAccessor` so that our component can work with
`formControl` and `ngModel`. As we are implementing `ControlValueAccessor` we will need to get a reference to the `NgControl` associated with our control and make it publicly available.

The easy way is to add it as a public property to your constructor and let dependency injection
handle it:

```ts
constructor(
  ...,
  @Optional() @Self() public ngControl: NgControl,
  ...,
) { }
```

Now as our component implements `ControlValueAccessor`, to avoid *cannot instantiate cyclic dependency* error, instead of using the `NG_VALUE_ACCESSOR` provider, we will set the value accessor directly:

```ts
@Component({
  ...
})
export class AddressFormComponent implements MatFormFieldControl<Address>, ControlValueAccessor {
  constructor(
    ...,
    @Optional() @Self() public ngControl: NgControl,
    ...,
  ) {

    if (this.ngControl != null) {
      // Setting the value accessor directly (instead of using
      // the providers) to avoid running into a circular import.
      this.ngControl.valueAccessor = this;
    }
  }
}
```

For additional information about `ControlValueAccessor` see the [API docs](https://angular.io/api/forms/ControlValueAccessor).

#### `focused`

This property indicates whether the form field control should be considered to be in a focused state. When it is in a focused state, the form field is displayed with a solid color underline. For the purposes of our component, we want to consider it focused if any of the part inputs are focused. We can use the `focusin` and `focusout` events to easily check this. We also need to remember to emit on the `stateChanges` when the focused stated changes stream so change detection can happen.

In addition to updating the focused state, we use the `focusin` and `focusout` methods to update the internal touched state of our component, which we'll use to determine the error state.

```ts
focused = false;

onFocusIn(event: FocusEvent) {
  if (!this.focused) {
    this.focused = true;
    this.stateChanges.next();
  }
}

onFocusOut(event: FocusEvent) {
  if (!this._elementRef.nativeElement.contains(event.relatedTarget as Element)) {
    this.touched = true;
    this.focused = false;
    this.onTouched();
    this.stateChanges.next();
  }
}
```

#### `empty`

This property indicates whether the form field control is empty. For our control, we'll consider it empty if all the fields are empty.

```ts
get empty() {
    const {
      value: {
        company,
        firstName,
        address2,
        address,
        city,
        lastName,
        postalCode,
        shipping,
        state,
      },
    } = this.addressForm;

    return (
      !company &&
      !firstName &&
      !address &&
      !address2 &&
      !city &&
      !lastName &&
      !postalCode &&
      !shipping &&
      !state
    );
  }
```

#### `shouldLabelFloat`

This property is used to indicate whether the label should be in the floating position. We'll use the same logic as `matInput` and float the placeholder when the input is focused or non-empty.

```ts
@HostBinding('class.floating')
get shouldLabelFloat() {
  return this.focused || !this.empty;
}
```

```css
span {
  opacity: 0;
  transition: opacity 200ms;
}
:host.floating span {
  opacity: 1;
}
```

#### `required`

This property is used to indicate whether the input is required. `<mat-form-field>` uses this information to add a required indicator to the placeholder. Again, we'll want to make sure we run change detection if the required state changes.

```ts
@Input()
get required() {
  return this._required;
}
set required(req: BooleanInput) {
  this._required = coerceBooleanProperty(req);
  this.stateChanges.next();
}
private _required = false;
```

#### `disabled`

This property tells the form field when it should be in the disabled state. In addition to reporting the right state to the form field, we need to set the disabled state on the individual inputs that make up our component.

```ts
  @Input()
  get disabled(): boolean {
    return this._disabled;
  }
  set disabled(value: BooleanInput) {
    this._disabled = coerceBooleanProperty(value);
    this._disabled ? this.addressForm.disable() : this.addressForm.enable();
    this.stateChanges.next();
  }
  private _disabled = false;
```

#### `errorState`

This property indicates whether the associated `NgControl` is in an error state. In this example, we show an error if the input is invalid and our component has been touched.

```ts
get errorState(): boolean {
    return this.addressForm.invalid && this.touched;
  }
```

#### `controlType`

This property allows us to specify a unique string for the type of control in form field. The `<mat-form-field>` will add a class based on this type that can be used to easily apply special styles to a `<mat-form-field>` that contains a specific type of control. In this example we'll use `address-form` as our control type which will result in the form field adding the class `mat-form-field-type-address-form`.

```ts
controlType = 'address-form';
```

#### `setDescribedByIds(ids: string[])`

This method is used by the `<mat-form-field>` to set element ids that should be used for the`aria-describedby` attribute of your control. The ids are controlled through the form field as hints or errors are conditionally displayed and should be reflected in the control's `aria-describedby` attribute for an improved accessibility experience.

The `setDescribedByIds` method is invoked whenever the control's state changes. Custom controls need to implement this method and update the `aria-describedby` attribute based on the specified element ids. Below is an example that shows how this can be achieved.

Note that the method by default will not respect element ids that have been set manually on the control element through the `aria-describedby` attribute. To ensure that your control does not accidentally override existing element ids specified by consumers of your control, create an input called `userAriaDescribedby`  like followed:

```ts
@Input('aria-describedby') userAriaDescribedBy: string;
```

The form field will then pick up the user specified `aria-describedby` ids and merge them with ids for hints or errors whenever `setDescribedByIds` is invoked.

```ts
setDescribedByIds(ids: string[]) {
    const controlElement = this._elementRef.nativeElement.querySelector(
      '.address-form-container'
    )!;
    controlElement.setAttribute('aria-describedby', ids.join(' '));
  }
```

#### `onContainerClick(event: MouseEvent)`

This method will be called when the form field is clicked on. It allows your component to hook in and handle that click however it wants. The method has one parameter, the `MouseEvent` for the click. In our case we'll just focus the first `<input>` if the user isn't about to click an `<input>` anyways.

```ts
  @ViewChild('company')
  companyInput!: HTMLInputElement;
  @ViewChild('firstName')
  firstNameInput!: HTMLInputElement;
  @ViewChild('lastName')
  lastNameInput!: HTMLInputElement;
  @ViewChild('address')
  addressInput!: HTMLTextAreaElement;
  @ViewChild('address2')
  address2Input!: HTMLTextAreaElement;
  @ViewChild('city')
  cityInput!: HTMLInputElement;
  @ViewChild('state')
  stateInput!: MatSelect;
  @ViewChild('postalCode')
  postalCodeInput!: HTMLInputElement;
  @ViewChild('shipping')
  shippingInput!: MatRadioGroup;

onContainerClick() {
    if (this.addressForm.controls.company.valid) {
      this._focusMonitor.focusVia(this.companyInput, 'program');
    } else if (this.addressForm.controls.firstName.valid) {
      this._focusMonitor.focusVia(this.firstNameInput, 'program');
    } else if (this.addressForm.controls.lastName.valid) {
      this._focusMonitor.focusVia(this.lastNameInput, 'program');
    } else if (this.addressForm.controls.address.valid) {
      this._focusMonitor.focusVia(this.addressInput, 'program');
    } else if (this.addressForm.controls.address2.valid) {
      this._focusMonitor.focusVia(this.address2Input, 'program');
    } else if (this.addressForm.controls.city.valid) {
      this._focusMonitor.focusVia(this.cityInput, 'program');
    } else if (this.addressForm.controls.state.valid) {
      this._focusMonitor.focusVia(
        this.stateInput._elementRef.nativeElement,
        'program'
      );
    } else if (this.addressForm.controls.postalCode.valid) {
      this._focusMonitor.focusVia(this.postalCodeInput, 'program');
    } else {
      this.shippingInput._radios.first.focus(undefined, 'program');
    }
  }
```

### Improving accessibility

One significant piece of information is missing for screen reader users. They won't be able to tell what this input group represents. To improve this, we should add a label for the group element using either `aria-label` or `aria-labelledby`.

It's recommended to link the group to the label that is displayed as part of the parent `<mat-form-field>`. This ensures that explicitly specified labels (using `<mat-label>`) are actually used for labelling the control.

In our concrete example, we add an attribute binding for `aria-labelledby` and bind it to the label element id provided by the parent `<mat-form-field>`.

```typescript
export class AddressFormComponent implements ... {
  ...

  constructor(...
              @Optional() public parentFormField: MatFormField,) {
```

```html
@Component({
  selector: 'app-address-form',
  template: `
    <form [formGroup]="addressForm"
         [attr.aria-labelledby]="parentFormField?.getLabelId()">
```

### Trying it out

Now that we've fully implemented the interface, we're ready to try our component out! All we need to do is place it inside a `<mat-form-field>`, which is inside another `form`:

```html
<div [formGroup]="form">
  <mat-card class="shipping-card">
    <mat-card-header>
      <mat-card-title>Shipping Information</mat-card-title>
    </mat-card-header>
    <mat-card-content>
      <mat-form-field  appearance="outline">
        <app-address-form formControlName="address" required></app-address-form>
      </mat-form-field>
    </mat-card-content>
  </mat-card>
</div>

```

Full code is available on [gitHub/reusable-material-form](https://github.com/shhdharmen/reusable-material-form).
