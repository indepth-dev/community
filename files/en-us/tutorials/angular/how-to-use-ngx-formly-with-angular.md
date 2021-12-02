---
title: How to use ngx-formly in Angular - Angular Tutorials | indepth.dev
author_name: Varvara Sandakova
author_link: https://twitter.com/varvarasandako1
discussion_link: https://github.com/indepth-dev/community/discussions/225
display_name: How to use ngx-formly in Angular
---
# **How to use ngx-formly in Angular**

How often do you create a form in your work routine? Have you ever had an idea to improve this process?

From my experience, I can say that implementing forms is the most common task in software engineer work. Since not every engineer had the need to implement complicated forms, it's very easy to underestimate how tedious and convoluted that process might be.

More information about how to implement reusable and reactive forms in Angular you can find [here](https://indepth.dev/posts/1415/implementing-reusable-and-reactive-forms-in-angular-2).

As with everything in our trade, we have a library that helps us works with forms. Today we'll review [ngx-formly](https://formly.dev) and learn how to transform the template (from JSON)  to an application form.
There are some valuable features of ngx-formly:

- automatic forms generation (render form reactively).
- easy to extend with custom field type, validation, wrapper, and extension.
- form control customization.

Our form will look like this:

![](https://images.indepth.dev/tutorials/angular/how-to-use-ngx-formly-in-angular/tutorial-ngx-formly-form.png)

If you are curious to learn more about how to build a reusable and reactive form in Angular, check out this article [here](https://indepth.dev/posts/1415/implementing-reusable-and-reactive-forms-in-angular-2)

Let's observe it in practice.

## 1.Set up a project and add ngx-formly

First, we need to add ngx-formly packages in our application:

`ng add @ngx-formly/schematics --ui-theme=material`

As you can see, in our example we installed **ngx-formly** with the **material theme.**

According to the [documentation](https://formly.dev/guide/getting-started) you can choose an optional flag for **-ui-theme.** There is a list of possible themes: `bootstrap, material, ionic, primeng, kendo, nativescript`.

After installation, we need register Formly in the application by importing `FormlyModule` in the `AppModule`:

```javascript
import { AppComponent }from './app.component';
import { ReactiveFormsModule }from '@angular/forms';
import { FormlyModule }from '@ngx-formly/core';
import { FormlyBootstrapModule }from '@ngx-formly/bootstrap';

@NgModule({
    imports: [
        BrowserModule,
        AppRoutingModule,
        ReactiveFormsModule,
        FormlyMaterialModule,
        BrowserAnimationsModule,
        FormlyModule.forRoot({})
    ],
    ...
})
exportclass AppModule { }
```
`FormlyModule.forRoot({})` call is required at the application's root level.

The [`forRoot()`](https://formly.dev/guide/getting-started) method accepts a config argument where you can pass extra config, register a custom field type, wrapper and validation.

## 2.Configuration fields

After form creation, we can add custom configuration to our fields:

```javascript
public fields: FormlyFieldConfig[] = [
    {
        key: 'name',
        type: 'input',
        templateOptions: {
            label: 'Name',
            placeholder: 'Enter your name',
            required: true,
            minLength: 1,
        }
    },
    {
        key: 'email',
        type: 'input',
        templateOptions: {
            label: 'Email',
            placeholder: 'Enter email',
            required: true,
            minLength: 3
        },
    },
    {
        key: 'motivation',
        type: 'textarea',
        templateOptions: {
            label: 'What is your motivation for writing?',
            placeholder: 'Type your answer',
        }
    },
    {
        key: 'content',
        type: 'radio',
        templateOptions: {
            label: 'What type of articles would you prefer to write?',
            placeholder: 'Fill the type of content',
            required: true,
            options: [
                { value: 1, label: 'Tutorial' },
                { value: 2, label: 'Deep-dive' },
                { value: 3, label: 'News' },
                { value: 4, label: 'Reference' }
            ],
        },
    },
    {
        key: 'technology',
        type: 'select',
        templateOptions: {
            label: 'Select your main technology',
            options: [
                { label: 'Javascript', value: 'javascript' },
                { label: 'Angular', value: 'angular' },
                { label: 'React', value: 'react' },
                { label: 'Vue', value: 'vue' },
                { label: 'Other', value: 'other' }
            ],
        },
    },
    {
        key: 'policy',
        type: 'checkbox',
        templateOptions: {
            label: 'I don\'t mind receiving The Deep Dive newsletter',
            required: true,
        }
    }
];
```
You can see that all fields have common properties such as key, type, template options. More information about the fields description you can find [here](https://formly.dev/guide/properties-options).

Please notice that you can add special properties to your field. Let's add a `className` to textarea field and set a `defaultValue.` Moreover, assume that we want to hide `Motivation` field if the `Name` field is empty:

```javascript
public model = {
    name: 'Test'
};

public fields: FormlyFieldConfig[] = [

    {
        key: 'motivation',
        type: 'textarea',
        className: 'customClass',
        defaultValue: 'Share experience',
        hideExpression: '!model.name',
        templateOptions: {
            label: 'What is your motivation for writing?',
            placeholder: 'Type your answer',
        }
    }
    ]
```
## 3. Creating a template

The next step is adding `<formly-form>` inside the`form` tag to your `AppComponent` template:


```javascript
<form [formGroup]="form" (ngSubmit)="onSubmit()">
    <formly-form [form]="form" [fields]="fields" [model]="model" [options]="options"></formly-form>
    <button class="mat-button" type="submit">Submit</button>
</form>
```
The `<formly-form>` component is the main container of the form, which will build and render form fields, it accepts the following inputs:

- `fields`: The field configurations for building the form.
- `form`: The form instance which allows to track model value and validation status.
- `model`: The model to be represented by the form.

Our form will include the next types of `fields`:

- Input
- Checkbox
- Radio
- Select
- Text area

## 4. Add custom validator for fields

Except for field configuration, we can also add validation to the form field.

Let's set `Emai` to the required field and configure some min length properties:

```javascript
public fields: FormlyFieldConfig[] = [
    {
        key: 'email',
        type: 'input',
        templateOptions: {
            label: 'Email',
            placeholder: 'Enter email',
            required: true,
            minLength: 3
        }
        ...
]
```

Then add custom validation to `Email` input.
We can add validation directly to the accurate field or add it to `FormlyModule` in `AppModule` and share these validation rules for all forms in your application. 

There are two approaches of implementation validation:

```javascript
// 1. locally impementation

export function EmailValidator(control: FormControl | any): ValidationErrors | null {
  return  /^(([^<>()[\]\.,;:\s@\"]+(\.[^<>()[\]\.,;:\s@\"]+)*)|(\".+\"))@(([^<>()[\]\.,;:\s@\"]+\.)+[^<>()[\]\.,;:\s@\"]{2,})$/i
    .test(control.value) ? null : { 'email': true };
}

....
public fields: FormlyFieldConfig[] = [

{
      key: 'email',
      type: 'input',
      templateOptions: {
        label: 'Email',
        placeholder: 'Enter email',
        required: true,
        minLength: 3
      },
      validators: {
        validation: [EmailValidator],
      },
    }
...
]
```

In addition to adding a sharable validator, you can also add sharable validation messages, such as `EmailValidatorMessage`, `minlengthMessage`. You can read more about validation here

```javascript
//2. sharable implementation

export function EmailValidator(control: FormControl | any): ValidationErrors | null {
  return  /^(([^<>()[\]\.,;:\s@\"]+(\.[^<>()[\]\.,;:\s@\"]+)*)|(\".+\"))@(([^<>()[\]\.,;:\s@\"]+\.)+[^<>()[\]\.,;:\s@\"]{2,})$/i
    .test(control.value) ? null : { 'email': true };
}
export function EmailMessage(err:any, field: FormlyFieldConfig) {
  return `"${field?.formControl?.value}" is not a valid email address`;
}
export function minlengthMessage(err: ValidationErrors, field: FormlyFieldConfig) {
  return `Should have atleast ${fild?.templateOptions?.minLength} characters`;
}
@NgModule({
    declarations: [AppComponent],
    imports: [
        BrowserModule,
        AppRoutingModule,
        ReactiveFormsModule,
        FormlyMaterialModule,
        BrowserAnimationsModule,
        FormlyModule.forRoot(
            {
                extras:
                    {lazyRender: true},
                validationMessages: [
                    { name: 'required', message: 'This field is required' },
                    { name: 'minlength', message: minlengthMessage },
                    { name: 'email', message: EmailValidatorMessage }
                ],
                validators: [
                    { name: 'email', validation: EmailValidator },
                ],
            },
        ),
    ],
    providers: [],
    bootstrap: [AppComponent]
})
export class AppModule { }

```

Last but not least we need to implement submit method in our component.

```javascript
public onSubmit() {
  if (this.form.valid) {
    console.log(JSON.stringify(this.model));
  }
}
```

As a result, when you press submit button you can see in the console the next `JSON`:

![](https://images.indepth.dev/tutorials/angular/how-to-use-ngx-formly-in-angular/tutorial-ngx-formly-json.png)

Congratulations, we generate a form automatically using ngx-formly. Full example of the project you can find on my [Github](https://github.com/VarvaraSandakova/formly-tutorial).

If you want to take a deeper look at the source code of ngx-formly you would see that ngx-formly uses reactive forms under the hood.

```javascript
<formly-form [form]="form" [fields]="fields" [model]="model" [options]="options" ></formly-form>
```

![](https://images.indepth.dev/tutorials/angular/how-to-use-ngx-formly-in-angular/tutorial-ngx-formly-angular-source.png)
There are several points that I want to notice from this screenshot:

- `FormlyForm` class implements Angular hooks (`DoCheck, OnChanges, OnDestroy`) for close interaction between library and Angular
- `form` property is an instance of `FormGroup`
- constructor injects a bunch of important instances such as:

    - `FormlyFormBuilder` - class responsible for rendering form with `componentFactoryResolver`

    - `FormlyConfig` - maintains a list of formly field directive types, used to register new field templates.

    - `NgZone` - is an execution context that persists across async tasks. In a word, `NgZone` is responsible for detecting changes in the component to update HTML.

    - `FormGroupDirective` - binds an existing FormGroup to a DOM element.

- `fields` implements an instance of `FormlyFieldConfig[]` that consists of sets of form configuration fields.

## Conclusion

In this article, we learned how to use, configure and customize ngx-formly in Angular application.

`Ngx-formly` - is a dynamic form library for Angular that helps you transform the template (from JSON)  to an application form. You can easily implement a dynamic form, extend it with a custom field type and add specific validation to your form template.

## References:

[https://formly.dev/](https://formly.dev/)

[https://angular.io/docs](https://angular.io/docs)

[https://indepth.dev/posts/1415/implementing-reusable-and-reactive-forms-in-angular-2](https://indepth.dev/posts/1415/implementing-reusable-and-reactive-forms-in-angular-2)