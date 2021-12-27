---
title: Angular Pipes: Complete Guide - Angular Tutorials | indepth.dev
author_name: Maria Zayed
author_link: https://twitter.com/mariawzayed
discussion_link:  https://github.com/indepth-dev/community/discussions/203
display_name: Angular Pipes: Complete Guide
---

# **Angular Pipes: Complete Guide**

## What are Pipes?
A pipe takes in data as input and transforms it into a different shape for displaying. Pipes are simple functions to use in template expressions to accept an input value and return a transformed value. Pipes are useful because you can use them throughout your application, while only declaring each pipe once.

![1](https://user-images.githubusercontent.com/27064594/147465973-deccba45-bf93-41f0-961a-d5f86bcadb3b.png)

Let’s create an Angular project to better understand the practical usage of pipes. Run the following command to create a brand new project to start exploring 

`ng new pipes`

Once it's done, open the project with your favorite code editor and remove all the code from the `app.component.html` file

### Pipes parameters
Let's take a look at the following code snippet where we will use the default Angular pipe - the date pipe (will take a closer look at it in the next section).

```javascript
// app.component.ts
export class AppComponent {
  birthday = new Date(1996, 3, 21);
}
```

```html
<!-- app.component.html -->
<h1>
  My birthday is {{ birthday | date }}
</h1>
```

![3](https://user-images.githubusercontent.com/27064594/147466283-906bdcb7-32fb-458a-bcd1-cf15fb2d3fc3.PNG)

As we can see, the pipe used in templates and its syntax is as follows:

`{{ input | pipeName }}`

The `input`  is the data we pass to the pipes, then we add the pipe symbol “|”, then the pipe name.

**What if we want to pass more parameters to pipes?**
Well, no worries, we can use as much as you want parameters. The only special thing that we need to keep in mind is the next syntax:

 `{{ input | pipeName: param1: param2: ...}}`

Date pipe provides some parameters and options, you can check them from [here](https://angular.io/api/common/DatePipe).
The general date pipe format looks like this:

`{{ value_expression | date [ : format [ : timezone [ : locale ] ] ] }}`

It takes a format, timezone, and locale optional parameters.

Let’s go back to our example and use the date pipe in a more advanced way.

```html
<!-- app.component.html -->
<h1>
  My birthday is {{ birthday | date: 'dd/MM/yyyy' }}
</h1>
```

![4](https://user-images.githubusercontent.com/27064594/147466399-94565b29-fe2e-465c-9027-427b1cd923af.PNG)

Notice that the display of the date is changed.

### Pipe chaining

Luckily, we can chain pipes as much as we want. But we need to take care of the order of the pipes.

Let’s get back to our example and capitalize the date text.

```html
<!-- app.component.html -->
<h1>
  My birthday is {{ birthday | date: 'fullDate' | uppercase }}
</h1>
```

![5](https://user-images.githubusercontent.com/27064594/147466473-5d6f85bf-e040-4eff-8959-865822fb3d98.PNG)

What if we added the `uppercase` pipe first? Let’s explore.

```html
<!-- app.component.html -->
<h1>
  My birthday is {{ birthday | uppercase | date: 'fullDate' }}
</h1>
```

We should get this compilation error.

![6](https://user-images.githubusercontent.com/27064594/147466541-5e09ea7e-6a7a-4f41-a7f6-31f58dcd0f3f.PNG)

This happens because pipes are normally applied from left to right, which will apply the `uppercase` pipe on the `birthday` variable, which is not allowed because the `birthday` variable is a date object, not a string.

## Built-in Pipes
Angular provides built-in pipes for typical data transformations, including transformations for internationalization (i18n), which use locale information to format data. The following are commonly used built-in pipes for data formatting:
DatePipe: Formats a date value according to locale rules.
* **UpperCasePipe/LowerCasePipe**: Transforms text to all upper/lower case.
* **CurrencyPipe**: Transforms a number to a currency string, formatted according to locale rules.
* **DecimalPipe**: Transforms a number into a string with a decimal point, formatted according to locale rules.
* **PercentPipe**: Transforms a number to a percentage string, formatted according to locale rules.

### Date pipe
Date pipe is executed only when it detects a pure change to the input value. A pure change is either a change to a primitive input value (such as String, Number, Boolean, or Symbol), or a changed object reference (such as Date, Array, Function, or Object).

Note that mutating a Date object does not cause the pipe to be rendered again. To ensure that the pipe is executed, you must create a new Date object. You can check the official [docs](https://angular.io/api/common/DatePipe) for more information.

Let’s see how we can play with date using the `DatePipe`

```html
<!-- app.component.html -->
<h1>
  My birthday is {{ birthday | date: 'shortDate' }}
</h1>
```

![14](https://user-images.githubusercontent.com/27064594/147466844-4fa92939-72bd-4b32-8032-d8dbe01a5e8e.PNG)

Notice that, passing the `shortDate` option changes the display of the date. There are many pre-defined options we can pass for the `format` option, you can check them from [here](https://angular.io/api/common/DatePipe#pre-defined-format-options).

What if we need to display our date in a format that isn’t pre-defined? Luckily, we can do this by passing our custom format.

Let’s customize our format like this `dd/MM/yyyy`. You can check the official [docs](https://angular.io/api/common/DatePipe#custom-format-options) for more information.

```html
<!-- app.component.html -->
<h1>
   My birthday is {{ birthday | date: 'dd/MM/yyyy' }}
</h1
```

![15](https://user-images.githubusercontent.com/27064594/147466971-836ab7a3-3cdd-416a-a4ac-cfe9f1841287.PNG)

### UpperCase pipe

Another interesting supported pipe is the `UpperCasePipe` which is simple and saves much time.

The following code is the [official](https://github.com/angular/angular/blob/13.1.1/packages/common/src/pipes/case_conversion_pipes.ts#L91-L115) implementation for the `UpperCasePipe`

```javascript
transform(value: string|null|undefined): string|null {
   if (value == null) return null;
   if (typeof value !== 'string') {
       throw invalidPipeArgumentError(UpperCasePipe, value);
   }
   return value.toUpperCase();
}
```

Although it looks simple and easy, but believe me it helps a lot and saves a decent amount of time.

There are also two more built-in pipes similar to this:
* **[LowerCasePipe](https://angular.io/api/common/LowerCasePipe)** which transforms text to all upper case.
* **[TitleCasePipe](https://angular.io/api/common/TitleCasePipe)** which capitalizes the first letter of each word and transforms the rest of the word to lower case.

## Custom Pipes

Angular gives us the freedom to create custom pipes to encapsulate transformations that are not provided with the built-in pipes and to use them the same way as the built-in pipes.

### Age pipe
Let’s create a custom pipe called `age` which takes a date and finds the age as represented in the following image:

![2](https://user-images.githubusercontent.com/27064594/147467205-8294dca9-f422-4c9d-a372-7aa06a0bba1b.png)

Angular CLI supports a command to create pipes and initialize the needed code for us.

`ng g p pipes/age`

This command will generate the pipe class and import it into the `app.module.ts`. We can delete the .spec  file, we will not use it

![7](https://user-images.githubusercontent.com/27064594/147467251-90391f2f-9a5b-4813-a22d-0f1923807e91.PNG)

Let’s take a closer look at the generated file.

```javascript
// age.pipe.ts
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
 name: 'age'
})
export class AgePipe implements PipeTransform {

 transform(value: any, ...args: any[]): any {
   return null;
 }
}
```

The pipe class has some requirements:
* Should be decorated with `@Pipe` decorator, and has a name for external usage (`age` in our case)
* Should implement the `PipeTransform` interface, which forces us to add the `transform` function
* `transform` function takes `value` of type `any` as an input + other parameters (optional)

Now, let’s add our touch:

```javascript
// age.pipe.ts
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'age',
})
export class AgePipe implements PipeTransform {

  transform(value: Date): any {
     const currentYear = new Date().getFullYear();
     const dobYear = value.getFullYear();     const age = currentYear - dobYear;

     return age + ' years old';
  }
}
```

```html
<!-- app.component.html -->
<h1>
  My age is {{ birthday | age }}
</h1>
```

![8](https://user-images.githubusercontent.com/27064594/147467359-c97d485c-1d76-43b2-87b1-6e28ddcbf325.PNG)

Our custom pipe takes the date of birth as type `Date` and returns the age as a `String`.

#### Adding more parameters for better functionality

Let’s make our pipe more fun and add some optional parameters. We will add 2 parameters `capitalize` and `withEmojis` flags.

```javascript
// age.pipe.ts
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({
  name: 'age',
})
export class AgePipe implements PipeTransform {

  transform(value: Date, capitalize: boolean = false, withEmojis: boolean = false): any {
     const currentYear = new Date().getFullYear();
     const dobYear = value.getFullYear();

     const age = currentYear - dobYear;

     let text = ' years old';
     const emojis = ' 🍰🥳';

     if (withEmojis) {
        text += emojis;
     }

     if (capitalize) {
        text = text.toUpperCase();
     }

     return age + text;
  }
}
```

```html
<!-- app.component.html -->
<h1>
  My age is {{ birthday | age: true: true }}
</h1>
```

![9](https://user-images.githubusercontent.com/27064594/147467458-3a3e6dd5-d56c-4f48-9ea8-a2029d90b224.PNG)

This isn’t the limit, you can add as many options as you want, just be aware of the order for the used options.

## Pure and Impure Pipes

The last yet important thing I want to mention is that there are two types of pipes in Angular, pure and impure pipes.

A pure pipe (the default) is only called when Angular detects a change in the value or the parameters passed to a pipe. An impure pipe is called for every change detection cycle no matter whether the value or parameter(s) changes.

### Filter Pipe

To demonstrate the difference between the pure and impure pipes we will create a new custom pipe called `filter`. This pipe will filter a list of names based on the user input. And we will create a button to add a new name to the list.

#### Pure Version

Take a look at the following code:

```javascript
// filter.pipe.ts
import {Pipe, PipeTransform} from '@angular/core';

@Pipe({
   name: 'filter'
})
export class FilterPipe implements PipeTransform {

   transform(value: any, filterString: string): any {
       if (value.length === 0 || filterString === '') {
           return value;
       }
       const resultArray = [];
       for (const name of value) {
           if (name.toLowerCase() === filterString.toLowerCase()) {
               resultArray.push(name);
           }
       }
       return resultArray;
   }
}
```

```javascript
// app.component.ts
export class AppComponent {
   filteredName = '';
   names = ['Maria', 'Carlos', 'Jessica', 'Emilia', 'Charlie'];

  constructor() {
   }

   addName(newName: string) {
       this.names.push(newName)
   }
}
```

```html
<!-- app.component.html -->
<input type="text" placeholder="Search ..." [(ngModel)]="filteredName">

<hr>

<input type="text" placeholder="Enter a name to add" #newName>
<button (click)="addName(newName.value)">Add Name</button>

<ul>
   <li *ngFor="let name of names | filter : filteredName">{{name}}</li>
</ul>
```

This is our demo UI

![10](https://user-images.githubusercontent.com/27064594/147467638-591bf5fb-429a-44c9-be86-f9cb66007f55.PNG)

If we type a new name and click on the `Add Name` button it will work.

![11](https://user-images.githubusercontent.com/27064594/147467686-cdb4e856-1934-4a3a-add0-856abd2af018.PNG)

But, if we typed anything in the search field and tried to add a new name it will **NOT** work.

![12](https://user-images.githubusercontent.com/27064594/147467721-7e000042-777e-4f4d-8f73-bff14f1fd742.png)

Actually, the new name is added to the list, but we don’t see it when there is a value in the search input.

The reason for this behavior is that Angular (thankfully I should say) does not recall our pipe’s transform function whenever the data changes. As soon as we change what we enter in the search input, we would update our pipe again (changing the input of the pipe will trigger a recalculation of the pipe). This is the pure pipe which is the default behavior in Angular.

#### Impure Version

On the other hand, we can force Angular to re-calculate our pipe whenever we change the data on the page (eg, adding a new name to the list). **But be aware that this might lead to performance issues**.

We could do this by changing the following in the `filter.pipe.ts`

```javascript
// filter.pipe.ts

...

@Pipe({
   name: 'filter',
   pure: false // <-- Add this
})

...
```

This small change will make our pipe `impure`, which means the pipe will be recalculated whenever the data changes.

![13](https://user-images.githubusercontent.com/27064594/147467834-c481fcab-a249-4ee5-86e6-92ebe565685d.PNG)

## Conclusion

In this article, we covered many important points, like understanding the idea behind pipes, pipes parameters, and pipe chaining. Then, we covered some of the built-in pipes and took a deep look at `DatePipe` and `UppereCasePipe`. Next, we talked about two important concepts the custom pipes, and pure/impure pipes.

In conclusion, we could say that pipes are an easy way to format data for rendering purposes. When you use a pipe, you don’t have to change your data model nor make a copy of it to make it “fit” what it should look like on the screen.
