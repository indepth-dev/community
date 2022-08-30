---
title: InDepth guide on how to customize styles in the ag-grid in Angular - Angular Tutorials | indepth.dev
author_name: Varvara Sandakova
author_link: https://twitter.com/varvarasandako1
discussion_link: https://github.com/indepth-dev/community/discussions/279
display_name: InDepth guide on how to customize styles in the AgGrid in Angular
---
# **InDepth guide on how to customize styles in the ag-grid in Angular**

## What you will learn
1. Basic ag-grid set up.
2. Multiple approaches for defining static styles in ag-grid:
- global overriding default styles of ag-grid;
- customize provided ag-grids theme or create a custom theme;
- redefine styles using `ViewEncapsulation.None`
3. Dynamic approaches of customization styles using ag-grid mechanisms such as `CellClass, cellClassRules, CellStyle`.

## Intro
Almost every application has a table that represents data. In most cases, this table should be prettified. in this tutorial, we are going to use ag-grid and I suggest you consider the task of customization ag-grid styles. Let's assume that the table displays the user's information: id, name,  avatar image. As a result, we will change some styles: font, grid colors: colors for the table header, borders, and styles for the hover event.
In this tutorial, we briefly go through setting up the configuration of ag-grid, and investigate several approaches on how to customize ag-grid styles in Angular such as :global overriding default ag-grid styles creating a new custom theme; redefining styles using `ViewEncapsulation.None,` customizing styles using cells styling mechanism - `CellClass, cellClassRules,` `CellStyle`.

## How to
## 1. Configurate ag-grid styles

Firstly, let's add the ag-grid framework to our project. Ag-grid provides detailed [instructions](https://www.ag-grid.com/angular-data-grid/getting-started/) on how to set up a grid in your application.

One of the important steps which are interesting for our guide is a configuration of styles in the style.scss.
According to the documentation firstly we need to import ag-grid.css for defining the default styles for the table, and then import the style for the theme.

```scss
//style.scss

// Core grid CSS styles
@import 'ag-grid-community/dist/styles/ag-grid.css';
// Optional CSS theme
@import 'ag-grid-community/dist/styles/ag-theme-alpine.css';
```
`ag-grid.css` is the core structural CSS needed for the grid table. Without this, the Grid will not work. 
Ag-grid provides a list of the [themes](https://www.ag-grid.com/angular-data-grid/themes/) which you can define as a class of your grid, we choose `ag-theme-alpine`, as we imported it above and define it as a class.
Secondly, let's define simple column configurations and add a default style for our table:

```html
<ag-grid-angular
  style="width: 535px; height: 300px;"
  class="ag-theme-alpine" // styles from the ag-grid theme
  [columnDefs]="columnDefs"
  [defaultColDef]="defaultColDef"
  [rowData]="(rowData$ | async).data"
  (gridReady)="onGridReady()">
</ag-grid-angular>
```

As you can see, we have the ability to add `width` and `height` directly to `ag-grid-angular` component.

Let's look at the other attributes in details:

```tsx
//app.component.ts

import {Component} from '@angular/core';
import {ColDef, GridApi, GridReadyEvent} from 'ag-grid-community';
import {HttpClient} from "@angular/common/http";
import {Observable} from "rxjs";

export interface User {
  id: string;
  first_name: string;
  last_name: string;
  avatar: string;
}

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
})
export class AppComponent {
//these properties apply to all columns in the grid
  public defaultColDef: ColDef = {
    width: 200
  };

// rowData - user data
  public rowData$: Observable<{data: User[]}> | undefined;
  public columnDefs: ColDef[] | undefined;

  constructor(private http: HttpClient) {}

  onGridReady() {
    this.columnDefs = this.configColumns();
    this.rowData$ =  this.getUserList();
  }

// set configuration for the columns and values
  private configColumns () {
    return [{
      headerName: 'User Information',
      children: [
        {
          headerName: 'Id',
          field: 'id',
          width: 50
        },
        {
          headerName: 'First Name',
          field: 'first_name',
        },
        {
          headerName: 'Last Name',
          field: 'last_name'
        },
      ]
    }];
  }

// return users data from the server
  private getUserList(): Observable<{ data: User[] } | any> {
    return this.http.get('https://reqres.in/api/users?page');
  }
}
```
In the `defaultColDef` we will set up grid properties that will apply to all columns in the grid - column `width`. `columnDefs` store columns configuration and `rowData$` loads users from the server.

When the grid is initialized, it will fire the `gridReady` event. If you want to use the APIs of the grid, you should put an `onGridReady(params)` callback onto the grid and grab the API(s) from the params. After that, you have the ability to interact with the grid (on top of the interaction that can be done by setting and changing the properties) More details related to the public interface that your application can use to interact with the grid, including methods, properties and events you can read in the [documentation](https://www.ag-grid.com/angular-data-grid/grid-interface/).

Once the grid will be ready we will set up our columns by calling `configColumns()`. The function `configColumns()`  returns the initial configuration for our columns with header name, column names, and related fields. And then load user data from the server by calling `get` method from `HttpClient` in `this.getUserList()`

According to documentation, the grid will create the cell values using simple text by default. If you want more complex HTML inside the cells you can achieve this using CellRenderer.For rendering images in the column ‘Avatar’ we use `CellRenderer`.

```tsx
private configColumns () {
  return [
    ...
        {
            headerName: 'Avatar',
            field: 'avatar',
            width: 80,
            cellRenderer: (params: ICellRendererParams): string => {
                return `<img  width="50px" height="50px" src="${params.value}">`;
            }
        }
  ]
}
```
In our example, `cellRenderer` is a function that received [ICellRendererParams](https://www.ag-grid.com/javascript-data-grid/component-cell-renderer/) as a parameter and returns string - image with defined width and height for it Moreover, we set the image to the src attribute through `params.value`.
The initial view for our table with default styles of ag-grid looks like this:

![](https://i.imgur.com/hqx8gTF.png)

I will propose you play a little bit with other valuable properties from the ag-grid to beautify our table. For example, we will change the height of the row, and header and add the ability to change grid height depending on the item length.

```tsx
<ag-grid-angular
        style="width: 735px; height: 300px;"
        class="ag-theme-alpine"
        [columnDefs]="columnDefs"
        [defaultColDef]="defaultColDef"
        [rowData]="(rowData$ | async)?.data"
        [rowHeight]="50" <-- row height 50px
        [headerHeight]="38"  <-- header height 50px
        [groupHeaderHeight]="24"  <-- group header height 50px
        (gridReady)="onGridReady()"> 
</ag-grid-angular>
```
You can also set the option `domLayout` to `'autoHeight'` to automatically give the element the space it needs (without vertical scrolling within the container)

The result of our work should be the next styled table:

![](https://i.imgur.com/gjjkpiR.png)

## 2. Multiple approaches for defining static styles in ag-grid:

### - global overriding default styles of ag-grid

Let's start with global overriding default ag-grid styles.

We add an additional class `user-grid` to our grid component to have the ability to customize default styles. So, now ag-grid-angular has classes `ag-theme-alpine`  - the provided ag grid theme with basic styles and our custom class `user-grid`

```tsx
<ag-grid-angular
  class="ag-theme-alpine user-grid"
	...
></ag-grid-angular>
```
For applying styles we need to define them under the selected theme styles - `ag-theme-alpine` and `user-grid class`:
```scss
// style.scss

// styles applied for the whole grid
.ag-theme-alpine.user-grid {
  font-family: "Gill Sans"; 
  border-color: #dee5ea;
  background-color: #fff;

// styles applied for the header
  .ag-header {
    background-color: #beccd5;
// styles applied for the header
    .ag-header-cell,
    .ag-header-group-cell {
      border-color: #dee5ea;
      &:hover {
        background-color: #fff
      }
    }
  }

// styles applied for the row
  .ag-row {
    &.ag-row-odd {
      background-color: #beccd5;
      &:hover {
        background-color: #dee5ea;
      }
    }

// styles applied for the cell
    .ag-cell {
      border: none;
    }
  }
}
```
As you can see from the example we defined custom styles under the class `.ag-theme-alpine.user-grid` We also apply our styles to header, row, and cell in the specific order that ag-grid has. From one point of view, it is a good way to give a global base style override in the case when you have a one-styling type of grid in your application. Using this approach we need to take into account [specificity](https://developer.mozilla.org/en-US/docs/Web/CSS/Specificity) and [cascade](https://developer.mozilla.org/en-US/docs/Web/CSS/Cascade) style issues. So, you need to follow the cascade rules to successfully customization the grid style.

### - customize provided ag-grids theme or create a custom them

Another approach for overriding ag-grid styles described by ag-grid documentation - customize a theme using provided ag-grid themes. Before a new version of Ag-Grid you can use the next pattern:

```scss
// style.scss 
@import "~ag-grid-community/src/styles/ag-grid.scss";

.ag-theme-alpine {
  @include ag-theme-alpine(
      (
				// styles applied for the grid
        font-family: "Gill Sans",
        border-color: #dee5ea,
				background-color: #fff,
				// styles applied for the header
        header-background-color: #beccd5,
				// styles applied for the row
        row-hover-color: #dee5ea,
        odd-row-background-color: #beccd5,
				// styles applied for the cell
	      cell-horizontal-border: null,
			  header-cell-hover-background-color: #fff,
      )
  );
}
```
Do not forget to import basic styles for the grid - `ag-grid.scss`

This approach allows you to change elements of the look by overriding variables for the theme `ag-theme-alpine`:

- `font-family:` "Gill Sans" - change font style,
- `border-color:` #dee5ea - change border color,
- `header-background-color`: #beccd5 change color of the header.

The full list of the base theme parameters you can find in the [link](https://www.ag-grid.com/archive/27.3.0/javascript-data-grid/themes-customising/#base-theme-parameters).

It's important to notice that we achieved the exact same result as we have in the previous example  -  overriding default styles of ag-grid globally.

We have the same comments which define for which element we apply the style:

- whole grid
- header
- row
- cell

Comparing these two approaches seems that the second one is easy to read and understand.
![](https://i.imgur.com/KkSpEms.png)
The team of Ag Grid recently released a new version of the grid. In AG Grid version 28 provides [enhanced theming capabilities](https://www.ag-grid.com/react-data-grid/themes/) by using CSS variables.

So, according to the updated documentation, we have an extendable list of the CSS [variables](https://www.ag-grid.com/angular-data-grid/global-style-customisation-variables/) for the customization theme. We will take a look in practice in the next section about creating your own theme.

### - create your own theme

If your desired look and feel are very different from the provided theme, at some point it becomes easier to start from scratch. To do this, you can define your own theme.

A theme is simply a CSS class name matching the pattern `ag-theme-*`, along with CSS rules that target this class name. 
We will apply our class choose a theme name and applying it to the grid:

Before applying your custom theme please ensure that `ag-grid.css` is loaded

```tsx
<ag-grid-angular
  class="ag-theme-user"
	...
></ag-grid-angular>
```
We just created a new theme. You haven't added any styles to it so what you will see is a nearly blank slate - basic customizable borders and padding but no opinionated design elements. You can then add customizations using CSS: The grid exposes many CSS variables that allow you to control its appearance using a small amount of code. For more fine-grained control you can write CSS rules targeting the class names of individual elements: Here is a list of variables accepted by the grid.

```scss
 /* customise with CSS variables */
.ag-theme-user {
// styles applied for the whole grid
  --ag-background-color: #dee5ea;
  --ag-border-color: #dee5ea;
  --ag-font-family:"Gill Sans";

// styles applied for the header 
  --ag-header-background-color: #beccd5;

// styles applied for the row
  --ag-row-hover-color: #dee5ea;
  --ag-odd-row-background-color: #beccd5;
  
// styles applied for the cell
  --ag-header-cell-hover-background-color: #fff;
  --ag-cell-horizontal-border: null;
}
```
This approach is useful when you have several grid that should have one common style. This style is applicable to common style changes such as table colors, colors of the cells or fonts, font size etc. So, you can customize the theme and easily reuse it for all grids with `ag-theme-alpine`  theme in your application. This approach is not very suitable when you need to apply some styles depending on the cell value. For these specific changes, it would be better to use the approach to defining CellClass, and CellStyles. We will investigate it later in the tutorial.

## 3. Redefine styles using `ViewEncapsulation.None`

Another way to override styles is to use one of the CSS styles encapsulation [policies](https://angular.io/api/core/ViewEncapsulation) for the [component](https://angular.io/api/core/Component) - `ViewEncapsulation.None`.

ViewEncapsulation.None is simple no encapsulation tells our component to allow styles for all components -  across component boundaries. In other words, Angular defines our styles like global CSS.

The `None` option adds all the styles defined by components to the head section of the HTML document. But you should always remember that the results are unpredictable, and there is no isolation between the styles defined by different components.

Let's create a new component  `<grid-alpine>` that will wrap our grid. We also add the initial configuration of the columns and add the previous value for the grid height.

```tsx
// grid-alpine.component.html

<ag-grid-alpine>
  <ag-grid-angular
    style="width: 535px; height: 300px;"
    class="ag-theme-alpine"
    [columnDefs]="columnDefs"
    [defaultColDef]="defaultColDef"
    [rowData]="(rowData$ | async)?.data"
    [rowHeight]="50"
    [headerHeight]="38"
    [groupHeaderHeight]="24"
    (gridReady)="onGridReady($event)"
  ></ag-grid-angular>
</ag-grid-alpine>
```

Then we  define template for the component as `<ng-content></ng-content>` . So we will have the ability to paste any other content/component in ag-grid-alpine. After that, we will set encapsulation properties to the ViewEncapsulation.None

```tsx
//grid-alpine.component.ts

import { Component, ViewEncapsulation} from '@angular/core';

@Component({
  selector: 'ag-grid-alpine',
  template: ` <ng-content></ng-content>`,
  styleUrls: ['./grid-alpine.component.scss'],
  encapsulation: ViewEncapsulation.None
})

export class GridAlpineComponent {}
```
Finally, we can use our style definition in the style.scss under the `ag-theme-alpine` theme.

```scss
// grid-alpine.component.scss

grid-alpine {
// styles applied for the whole grid
 .ag-theme-alpine {
    font-family: "Gill Sans";
    border-color: #dee5ea;
    background-color: #fff;

// styles applied for the header
    .ag-header {
      background-color: #beccd5;
      .ag-header-cell,
      .ag-header-group-cell {
        border-color: #dee5ea;
        &:hover{
          background-color: #fff
        }
      }
    }
   
// styles applied for the row
    .ag-row {
      &.ag-row-odd {
        background-color: #beccd5;
        &:hover {
          background-color: #dee5ea;
        }
      }
      
// styles applied for the cell
      .ag-cell{
        border: none;
      }
    }
  }
}
```
I like this approach for easy readability and the ability to reuse wrapper components and their style. Important to notice - If you want to apply this style to another grid you will need to wrap your grid to `ag-grid-alpine` component.

## 4. How we can additionally customize cells or rows of the ag-grid?

Alternatively, if you want to change the appearance of individual columns, headers or cells then consider using [row styles](https://www.ag-grid.com/angular-data-grid/row-styles/), [cell styles](https://www.ag-grid.com/angular-data-grid/cell-styles/) mechanisms:

- `CellClass` - apply styles through class
- `cellClassRules` - apply several classes
- `CellStyle` - apply styles directly
- `getRowStyle, getRowClass` - provide style/class to apply to all rows.

Using these column properties you can highlight  Rows and Columns or Cells in a more flexible way.

For instance, let's do the same styling as we did previously using `odd-row-background-color: #beccd5)`- highlight the odd rows in grey color. But now we will implement styling using different properties of the column - `CellStyle, CellClass` and `getRowStyle`.

- `CellStyle`

As these changes related to all grid rows it would be better to apply them to `defaultColDef`:
```tsx
  public defaultColDef: ColDef = {
// this properties apply for all columns in the grid
			    width: 200, 
			    resizable: true,
          cellStyle: (params:CellClassParamms) => {
                if (params.node.rowIndex % 2 !== 0) {
                    return { backgroundColor: 'beccd5'};
                } else {
                    return null;
                }
            }
  };
```
Here we provide CSS styles directly (not using a class) to the cell.  `cellStyle` is a  function that checks if the row is odd or even `params.node.rowIndex % 2 !== 0`  using `CellClassParams` and returns objects of CSS styles `{ backgroundColor: 'beccd5'}`.  As a result, `backgroundColor` will be applied to the odds row in the grid.

- `getRowClass`
```html
<ag-grid-angular
    [getRowClass]="getRowClass"
     ... 
</ag-grid-angular>
```
We will get the same result -highlight the odd rows in grey color but using getRowClass.

```tsx
 public defaultColDef: ColDef = {
        width: 200, 
        resizable: true, 
        this.getRowClass = params => {
            if (params.node.rowIndex % 2 === 0) {
                return 'odd';
            }
        }
}
```
One difference will be in defining additional styles for a new class `odd`:

```scss
.ag-theme-alpine.user-grid {
    .ag-row {
      &.odd {
        background-color: #beccd5;
      }
    }
  }
```
The similar syntax we  implement using `getRowStyle`. In the HTML we apply `[getRowStyle]="getRowStyle”` to the grid.
```tsx
public defaultColDef: ColDef = {
    width: 200, 
    resizable: true, 
    this.getRowStyle = params => {
        if (params.node.rowIndex % 2 === 0) {
            return 'odd';
        }
    }
}
```
- `cellClass`

Ag grid provides the ability to define a custom class for the cells in your grid. In case you want to add a specific class to your column you can use cellClass property of [ColDef](https://www.ag-grid.com/angular-data-grid/column-properties/) interface. Can be string, array of strings, or a function that returns a string or array of strings.

```tsx
public defaultColDef: ColDef = {
    width: 200,
    resizable: true, 
    cellClass: (params: CellClassParams) => {
        if (params.node.rowIndex % 2 === 0) {
            return 'odd';
        }
    }
}
```
You can also add `cellClass` for some specific column instead of defining class for all columns.

These additional properties are very useful for of the customization your table. You can read more about the properties of the column [here](https://www.ag-grid.com/angular-data-grid/cell-styles/?gclid=Cj0KCQjwzqSWBhDPARIsAK38LY-tIGKTJiX3CenchN2YTFk_0uPSDeyiOZOepYQD9Ahvjxnkgwgcz5AaAr7jEALw_wcB).

## In-depth bits

One of the popular answers on the StackOverflow for overriding styles questions is approach - using :host ::ng-deep pseudo selectors. This selector was firstly presented in Angular 4.3.0 - to  provides the ability to use pseudo  :host ::ng-deep selectors for defining styles:

```scss
:host ::ng-deep .ag-header {
  background-color: #beccd5;
}
```

In other words `ng-deep` completely disables the view-encapsulation style so it conceptually seems more like a common style than a component style. So, If we use it without the: host pseudo-class, it will make the style-rule global and it is not a good [approach.In](http://approach.In) addition to this

`ng::deep` is [deprecated](https://angular.io/guide/component-styles#deprecated-deep--and-ng-deep) and it is a bad idea to use a deprecated feature. There will be always a risk to have some unexpected bug or conflict in your application. More developers choosing to use ng deep approach because they want a “quick fix” without really thinking through the consequences. In the future, it would be hard to track down bugs and find a root cause for it. One of the scenarios where you will have the issue caused by using `ng-deep` you can find in this [article](https://tutorialsforangular.com/2020/04/13/the-dangers-of-ng-deep-bleeding/).

As we discussed previously, Ag-grid provides the ability to customize themes using CSS variables. As a side note, I suggest you consider how we can override  CSS variables in Angular. Let's consider the example step by step:

Firstly, lets declare the CSS property `--background-color: #ffffff` at the root scope in the file `variables.scss`. Declaring your variables at the root scope refers them to the global scope and is available to the whole file.

```scss
// variables.scss

:root {
  --background-color: #ffffff
}
```
Alternatively, you can also declare `--background-color` under the `body` or any `class/element` selector to define the area for them to apply.

After the variable declaration we are going to apply `background-color` to the header for the grid:

```scss
// styles.scss

.ag-theme-alpine.user-grid {
  .ag-header {
		...
    background-color: var(--background-color);
  }
}
```
Let's add a button to change the background-color of the header by clicking on the button:
```tsx
<button (click)="changeHeaderColor()">Change color</button>
```
We need to inject `ElementRef` into the component to have the ability to find the header in the component.

`this.elementRef.nativeElement.querySelector('.ag-header')` Then we apply [setProperty](https://developer.mozilla.org/en-US/docs/Web/API/CSSStyleDeclaration/setProperty) method to set a new value for `background-color` property on a CSS style declaration object.

```tsx
constructor(private elementRef: ElementRef) { }

changeHeaderColor() {
  this.elementRef.nativeElement.querySelector('.ag-header').style.setProperty('--background-color', 'green');
}
```

## Conclusion

In this tutorial, we learned how to customize the styles of the ag-grid table in Angular using different approaches: overriding default ag-grid styles globally, creating a new custom theme; redefine styles using `ViewEncapsulation.None` customize styles using mechanism from out of the ag-grid box - `CellClass, cellClassRules, CellStyle`. Investigating all of these methods we confirmed that ag-grid is a complex solution for displaying tabular data in a series of rows and columns. We can easily adapt our styles using multiple approaches for customization grid avoiding bad practices.

You can find [here](https://github.com/VarvaraSandakova/ag-grid-style-customization) a link with tutorial project.