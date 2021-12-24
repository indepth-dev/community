# Angular Material Menu: Nested Menu using Dynamic Data

> The Angular Material Menu is a floating panel containing a list of options. In this tutorial, we will learn how we can create nested menus from dynamic data.

We will first learn the basics of Angular Material Menu and how to render a nested menu with a static HTML template.

Then we will understand why and what changes are needed to dynamically render nested menus from data.


## Angular Material Menu

[`<mat-menu>`](https://material.angular.io/components/menu/overview) is a floating panel containing a list of options. By itself, the `&lt;mat-menu>` element does not render anything. The menu is attached to and opened via application of the `matMenuTriggerFor` directive:


```html
<button mat-button [matMenuTriggerFor]="menu">Menu</button>
<mat-menu #menu="matMenu">
  <button mat-menu-item>Item 1</button>
  <button mat-menu-item>Item 2</button>
</mat-menu>
```

![mat-menu output](https://i.imgur.com/gLlrQCD.gif "mat-menu output")


## Static Nested Menu

To render a nested menu with static data, or simply from HTML template, we will have to define the root menu and sub-menus, in addition to setting the `[matMenuTriggerFor]` on the `mat-menu-item` that should trigger the sub-menu:

```html
<button mat-button [matMenuTriggerFor]="animals">Animal index</button>

<mat-menu #animals="matMenu">
  <button mat-menu-item [matMenuTriggerFor]="vertebrates">Vertebrates</button>
</mat-menu>

<mat-menu #vertebrates="matMenu">
  <button mat-menu-item [matMenuTriggerFor]="fish">Fishes</button>
  <button mat-menu-item>Amphibians</button>
  <button mat-menu-item>Reptiles</button>
  <button mat-menu-item>Birds</button>
  <button mat-menu-item>Mammals</button>
</mat-menu>

<mat-menu #fish="matMenu">
  <button mat-menu-item>Baikal oilfish</button>
  <button mat-menu-item>Bala shark</button>
  <button mat-menu-item>Ballan wrasse</button>
  <button mat-menu-item>Bamboo shark</button>
  <button mat-menu-item>Banded killifish</button>
</mat-menu>
```

And the output will be like below:

![static nested mat-menu](https://i.imgur.com/aMSIM3N.gif "static nested mat-menu")


## Dynamic Nested Menu

Building a menu from dynamic data is often needed, especially in business or enterprise applications. For example, loading features based on logged-in user’s permissions. The data may come from a REST API.

We will take an example where items and their children are loaded from a database. And we will render a nested menu for each item which has children.


### Database

For the database, we are going to assume the following service. You can connect the actual REST API with this service, too:


```typescript
import { Injectable } from "@angular/core";
import { delay, of } from "rxjs";

@Injectable({ providedIn: "root" })
export class DynamicDatabase {
  dataMap = new Map<string, string[]>([
    ["Fruits", ["Apple", "Orange", "Banana"]],
    ["Vegetables", ["Tomato", "Potato", "Onion"]],
    ["Apple", ["Fuji", "Macintosh"]],
    ["Onion", ["Yellow", "White", "Purple"]],
    ["Macintosh", ["Yellow", "White", "Purple"]],
  ]);

  rootLevelNodes: string[] = ["Fruits", "Vegetables"];

  getChildren(node: string) {
    // adding delay to mock a REST API call
    return of(this.dataMap.get(node)).pipe(delay(1000));
  }

  isExpandable(node: string): boolean {
    return this.dataMap.has(node);
  }
}
```

Above service’s code is simple:

* `dataMap` represents data, this could be the actual database
* `rootLevelNodes` represents first nodes to render
* `getChildren` will return the items for a particular node. We will use this to render sub-menu items
* `isExpandable` will return whether there are any children. We will use this to identify whether a sub-menu is needed


### Nested Menu

Now understand that, we can’t simply follow the standard HTML template of `MatMenu` for dynamic data. Below are the reasons:


1. We can’t load the `<mat-menu>` until we know that item has children
2. We can’t attach `[matMenuTrigger]` to `mat-menu-item` until `<mat-menu>` is loaded in the DOM

So, to handle the above problems we will follow the below approach in respective order:

1. Read node from node list
2. Check if any node is expandable
    1. If yes, then create a sub-menu `<mat-menu>` with loader and attach it with `[matMenuTrigger]` in the rendered node’s `mat-menu-item`
        1. Once the user clicks node, get and render child nodes in sub-menu
        2. For sub-menu’s child-nodes, again follow the same approach and start from step 2
    2. If no, then simply create node’s `mat-menu-item`


#### Root Component

To achieve the above approach, we will create a `app-menu` component and use it in `app-root`:

```html
<!-- src/app/app.component.html -->

<app-menu
  [trigger]="'Food'"
  [data]="initialData"
  [isRootNode]="true"
></app-menu>
```

```typescript
// src/app/app.component.ts

import { Component } from "@angular/core";
import { DynamicDatabase } from "./dynamic-database.service";

@Component({
  selector: "app-root",
  templateUrl: "app.component.html",
})
export class AppComponent {
  title = "mat-menu-dynamic-data";
  initialData: string[] = [];
  constructor(private database: DynamicDatabase) {
    this.initialData = this.database.rootLevelNodes.slice();
  }
}
```

We are reading `rootLevelNodes` and passing it as `data` in `app-menu`.


#### Menu Component

For the menu, initially we want to show a button, which will trigger a menu:

```html
<!-- src/app/menu/menu.component.html -->

<button mat-button [matMenuTriggerFor]="menu">
  {{ trigger }}
</button>
<mat-menu #menu="matMenu">
  <button mat-menu-item *ngFor="let node of data">{{ node }}</button>
</mat-menu>
```

And the class looks like this:

```typescript
// src/app/menu/menu.component.ts

export class MenuComponent {
  @Input() data: string[] = [];
  @Input() trigger = "Trigger";
  @Input() isRootNode = false;
}
```


#### Recursion

Now, to render a nested menu, we will just need to handle recursion in this code. And generate the same DOM structure for each nested menu.

So, first we will change the code inside `&lt;mat-menu>`:

```html
<!-- src/app/menu/menu.component.html -->

<button mat-button [matMenuTriggerFor]="menu">
  {{ trigger }}
</button>
<mat-menu #menu="matMenu">
  <ng-container *ngFor="let node of data; let i = index">
    <button mat-menu-item>
      <app-menu
        [trigger]="node"
        *ngIf="isExpandable(node); else menuItem"
      ></app-menu>
    </button>
    <ng-template #menuItem>
      <button mat-menu-item>{{ node }}</button>
    </ng-template>
  </ng-container>
</mat-menu>
```

Now, inside the menu, we are checking for each node, if the `isExpandable` method returns `true`, we are rendering `app-menu` again inside it.

`isExpandable` method will simply call `isExpandable` from the `DynamicDatabase` service:

```typescript
// src/app/menu/menu.component.ts

// ...

export class MenuComponent {

  // ...

  isExpandable(node: string): boolean {
    return this.database.isExpandable(node);
  }
}
```

Let’s look at the output:

![menu component root trigger output](https://i.imgur.com/yjZ91Lc.gif "menu component root trigger output")


Notice that text is also hoverable inside `mat-menu-item`. That’s because of the `mat-button`. When `app-menu` is rendered inside, we will have to change the directive of the button from `mat-button` to `mat-menu-item`, let’s do that:

```html
<!-- src/app/menu/menu.component.html -->

<button *ngIf="isRootNode" mat-button [matMenuTriggerFor]="menu">
  {{ trigger }}
</button>
<button *ngIf="!isRootNode" mat-menu-item [matMenuTriggerFor]="menu">
  {{ trigger }}
</button>
<mat-menu #menu="matMenu">
  <ng-container *ngFor="let node of data; let i = index">
    <button mat-menu-item>
      <app-menu
        [trigger]="node"
        *ngIf="isExpandable(node); else menuItem"
      ></app-menu>
    </button>
    <ng-template #menuItem>
      <button mat-menu-item>{{ node }}</button>
    </ng-template>
  </ng-container>
</mat-menu>
```

Let’s look at the output now:

![menu component with nested menu trigger output](https://i.imgur.com/okSdRKX.gif "menu component with nested menu trigger output")


It’s rendering the root items fine now, but the sub-menu is blank. Let’s add data in it.


#### Data

We want to load the data once the menu is rendered and opened. So, we will use the `(menuOpened)` event to load the `data`. `menuOpened` emits the event when the associated menu is opened.

We only want to load the `data` for non-root items, because for root items, `data` is coming from the parent component.

```html
<!-- src/app/menu/menu.component.html -->

<button *ngIf="isRootNode" mat-button [matMenuTriggerFor]="menu">
  {{ trigger }}
</button>
<button
  *ngIf="!isRootNode"
  mat-menu-item
  [matMenuTriggerFor]="menu"
  (menuOpened)="getData(trigger)"
>
  {{ trigger }}
</button>

<!-- rest remains same -->
```

Let’s create a `getData` method in `menu.component.ts`:

```typescript
// src/app/menu/menu.component.ts

// ...
export class MenuComponent {
  // ...

  isLoading = false;
  dataLoaded = false;

  getData(node: string) {
    if (!this.dataLoaded) {
      this.isLoading = true;
      this.database.getChildren(node).subscribe((d) => {
        this.data = d?.slice() || [];
        this.isLoading = false;
        this.dataLoaded = true;
      });
    }
  }
}
```

With `getData`, we are creating 2 more flags:

1. `isLoading` - Indicates if `data` is being fetched
2. `dataLoaded` - Indicates if `data` is already loaded and prevents further fetching

Let’s look at the output now:

![menu component with dynamic data](https://i.imgur.com/PDRxX4I.gif "menu component with dynamic data")

Notice that data is getting loaded after a particular time, that’s because we have added a `delay` in `DynamicDatabase.getChildren` to simulate an API call. And it’s not fetching the data again if it’s already loaded and in that case menu items are rendered instantly.

#### Loader

The last thing remaining is to show a loader when `data` is getting fetched. We already have `isLoading` flag, let’s use that to show [`&lt;mat-spinner>`](https://material.angular.io/components/progress-spinner/overview):

```html
<!-- src/app/menu/menu.component.html -->

<!-- rest remains same -->

<mat-menu #menu="matMenu">
  <button
    mat-menu-item
    *ngIf="isLoading"
    style="display: flex; justify-content: center; align-items: center"
  >
    <mat-spinner mode="indeterminate" diameter="24"></mat-spinner>
  </button>
  <ng-container *ngFor="let node of data; let i = index">
    <!-- rest remains same -->
  </ng-container>
</mat-menu>
```

Notice that I have added some inline styles so that `&lt;mat-spinner>` is displayed in the center of `mat-menu-item`.

Let’s look at the output now:

![nested menu with loader](https://i.imgur.com/XEYEJCH.gif "nested menu with loader")


## Summary

We started with a simple example of a menu, where we rendered nested menus using static HTML template.

Then we understood the need for dynamic data in nested menus and the problems to achieve dynamicity with the simple HTML template.

We then created a `app-menu` component. First we loaded a menu with root items, provided as `data` input from the parent component.

Then we handled recursion, rendering `app-menu` inside `app-menu`, based on `isExpandable` flag. Next we implemented fetching data based on `menuOpened` event and finally we displayed a loader while fetching the data.

All the above code is available on GitHub repo: [mat-menu-dynamic-data](https://github.com/shhdharmen/mat-menu-dynamic-data).
