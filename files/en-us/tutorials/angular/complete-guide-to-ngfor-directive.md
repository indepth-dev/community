---
title: Complete guide to ngFor directive in Angular - Angular Tutorials | indepth.dev
author_name: Maciej Wójcik
author_link: https://twitter.com/maciej_wwojcik
discussion_link: https://github.com/indepth-dev/community/discussions/196
display_name: Complete guide to ngFor directive in Angular
---
# **Complete guide to ngFor directive in Angular**

## Why do I need the ngFor directive?

Whenever you have an iterable list with elements you need to render, you will find the ngFor perfect for this job. The ngFor directive is extremely powerful when it comes to rendering a list of elements in Angular, because of all configuration options and built-in efficiency to reuse DOM nodes.

What is ngFor? It's an Angular directive - the structural one. This means it can change the DOM layout - which is precisely what it is doing. We, as developers, provide a simple HTML block that tells Angular how to render a single item, and the directive adds that element multiple times into DOM to render the list.

## How can I use ngFor?

The syntax for the basic use case is straightforward to implement and remember:

```HTML
<div *ngFor="let item of elements">...</div>
```

You can apply the directive to any DOM element that you want to be rendered multiple times depending on a list length. For instance, let's say we want to render such an array of fruits, stored as a component's property:

```Typescript
@Component({
  selector: 'my-app',
  templateUrl: './app.component.html',
})
export  class  AppComponent {
  fruits = ['apple', 'pear', 'banana', 'coconut'];
}
```

In the template we can simply use ngFor directive to render the same DOM element for each fruit and use text interpolation `{{}}` to display the fruits name:

```HTML
<ul>
  <li *ngFor="let fruit of fruits">{{ fruit }}</li>
</ul>
```

The result is simply a list of fruits - which we wanted to achieve.

!['image presenting list of fruits'](https://i.imgur.com/kcEnHZU.png)

  
Taking a close look at the actual DOM We see a ul entry tag and then a list of li elements with fruit in it.

```HTML
<ul _ngcontent-cfh-c83="">
  <li _ngcontent-cfh-c83="">apple </li>
  <li _ngcontent-cfh-c83="">pear </li>
  <li _ngcontent-cfh-c83="">banana</li>
  <li _ngcontent-cfh-c83="">coconut </li>
  <!--bindings={
  "ng-reflect-ng-for-of": "apple, pear, banana, coconut"
  }-->
</ul>
```

## Advanced use cases

In the previous example, I used just one HTML element to render the list. However, you often need to render a more complicated DOM structure with children. Angular calls it a template. This template can include not only regular DOM elements but also Angular components.

Take a look on such example when we want to render a list of nested children in the DOM:

```HTML
<section>
  <h3>Fruits</h3>
  <ul>
    <li>
      <img [src]="fruits[0].image" />
      <p>{{ fruits[0].name }}</p>
      <button (click)="select(0)">Select</button>
    </li>
    <li>
      <img [src]="fruits[1].image" />
      <p>{{ fruits[1].name }}</p>
      <button (click)="select(1)">Select</button>
    </li>
    <li>
      <img [src]="fruits[2].image" />
      <p>{{ fruits[2].name }}</p>
      <button (click)="select(2)">Select</button>
    </li>
  </ul>
</section>
```

A lot of code, and not really dynamically created. Let’s introduce ngFor here:

```HTML
<section>
  <h3>Fruits</h3>
  <ul>
    <li *ngFor="let fruit of fruits">
      <img [src]="fruit.image" />
      <p>{{ fruit.name }}</p>
      <button (click)="select(fruit)">Select</button>
    </li>
  </ul>
</section>
```

Looks better, but what if we would like to wrap that nested template in the Angular component:

```Typescript
@Component({
  selector: 'app-fruit',
  template: `<img [src]="fruit.image" />
             <p>{{ fruit.name }}</p>
             <button (click)="onSelect()">Select</button>`,
})
export  class  FruitComponent {
  @Input() fruit: string;
  @Output() select = new  EventEmitter();

  onSelect(){
    this.select.emit(this.fruit);
  }
}
```

It has an input to pass the fruit string into the component, and it uses that to display the fruit in a paragraph element. Then the template could look like this:

```HTML
<section>
  <h3>Fruits</h3>
  <app-fruit [fruit]="fruits[0]" (select)="select($event)"></app-fruit>
  <app-fruit [fruit]="fruits[1]" (select)="select($event)"></app-fruit>
  <app-fruit [fruit]="fruits[2]" (select)="select($event)"></app-fruit>
</section>
```

Finally let's use ngFor to render that dynamically:

```HTML
<section>
  <h3>Fruits</h3>
  <app-fruit *ngFor="let fruit of fruits" [fruit]="fruit" (select)="select($event)"></app-fruit>
</section>
```

## Extra features of ngFor

ngFor comes with a set of additional features that could be used to get extra information useful for styling or some advanced logic.

### Index of an element

There are cases when knowing a particular element’s position (index) is very helpful. The simplest use case would be to display the element’s position next to the actual element

This can be achieved by adding a new variable into the equation and assigning to it the special keyword `index`:

```Javascript
*ngFor="let item of elements; let i = index"
```

Here’s a more involved example, imagine you want to highlight a selected list item - to persist what is chosen at the moment, you could use a component property that keeps the currently selected tile in its position.

I will use the same example as before - a list of fruits. We need a slight change in the component class:

```Typescript
@Component({
  selector: 'my-app',
  templateUrl: './app.component.html',
})
export  class  AppComponent {
  fruits = ['apple', 'pear', 'banana', 'coconut'];
  activeIndex: number;
}
```

As you see, I added the activeIndex property to persist the currently selected item.

Now the template code. I will use an index to display it next to the fruit and also to add a class for selected item dynamically:

```HTML
<ul>
  <li
    *ngFor="let fruit of fruits; let i = index"
    [class.active]="i === activeIndex"
    (click)="activeIndex = i"
  >
    {{i}} - {{ fruit }}
  </li>
</ul>
```

This results in a list with position numbers and a element with special styles applied on user select.

!['image presenting list of fruits with positions and active element selected'](https://i.imgur.com/RkhEcFK.png)

### Odd and even rows

Sometimes it would be essential to know which list elements are odd and which even. For instance, when implementing a table, we often want to distinguish rows visually.

How to get information about whether the element is in an odd or even position? The directive comes with two new variables that we could use for that:

```Javascript
*ngFor="let item of elements; let evenItem = even; let oddItem = odd"
```

To present that in the context of our fruits example, let's assume that we want to apply styles for odd and even elements, based on such classes:

```CSS
.odd {
  background-color: #ffca28;
}

.even {
  background-color: #66bb6a;
}
```

We need to extend our ngFor declaration in the template, and dynamically add classes for two types for items:

```HTML
<ul>
  <li
    *ngFor="let fruit of fruits; let evenItem = even; let oddItem = odd;"
    [class.even]="evenItem"
    [class.odd]="oddItem"
  >
    {{ fruit }}
  </li>
</ul>
```

Now the result:

!['image presenting list of fruits styled differently based on the position'](https://i.imgur.com/876r3AS.png)

Extra trick: Even and odd elements are built in, but what if you want to highlight every third argument? Remember you still know the index of the element. So how to apply the same even / odd styles without using built-in features?

```HTML
<li
  *ngFor="let fruit of fruits; let i = index;"
  [class.even]="index % 2 === 0"
  [class.odd]="index % 2 !== 0"
>
```

Just like that! Remember, knowledge about the item position is very powerful.

### The first and last element

A very similar feature to the even/odd is first/last. This functionality will tell us if the element is first or last on the list.

Syntax is also very similar:

```Javascript
*ngFor="let item of elements; let first = first; let last = last"
```

For example, I will apply the same styles as before, but using new classes just for the first and last elements:

```CSS
.first-element {
  background-color: #ffca28;
}

.last-element {
  background-color: #66bb6a;
}
```

Based on that, we need to update the template and use new set of variables:

```HTML
<ul>
  <li
    *ngFor="let fruit of fruits; let first = first; let last = last;"
    [class.first-element]="first"
    [class.last-element]="last"
  >
    {{ fruit }}
  </li>
</ul>
```

The result is simply:

!['image presenting list of fruits styled differently for first and last item'](https://i.imgur.com/UydzONV.png)

## Performance side

It's always important to have in mind the performance of the solution we implement. In case of the ngFor directive, there are two things I want to bring to your attention.

### Tracking items

Adding or removing elements in DOM are expensive operations, so Angular has a strategy to keep it as efficient as possible. Imagine you rendered a list of items, and then you add a new item. Rendering the whole list from scratch will not be a good idea, especially for long lists. So how to prevent that and create just this one element?

Angular takes care of that by identifying particular elements associated with a specific instance of DOM node created from the template. So even if we reorder our list, but the items do not change, Angular won't re-create the DOM nodes for each item. It will simply render existing DOM nodes in a different order. For new items in the list that don’t have corresponding DOM nodes, Angular will create a new DOM node, but only for them! Items deleted from the list will result into deletion of corresponding DOM nodes.

How does Angular know how to identify items? Be default ngFor tracks items using object identity.

That means it will work in most common cases, but keep in mind that if you run into a performance issue, you should help Angular with that task and tell it exactly how to track your objects.

Let's extend our fruits example a bit and add more properties:

```Typescript
@Component({
  selector: 'my-app',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css'],
})
export  class  AppComponent  implements  OnInit {

fruits = [
  { id: 'f1', emoji: '🍎', name: 'apple' },
  { id: 'f2', emoji: '🍐', name: 'pear' },
  { id: 'f3', emoji: '🍌', name: 'banana' },
  { id: 'f6', emoji: '🥥', name: 'coconut' },
];

  ngOnInit() {
    setTimeout(() => {
      const bananaPosition = 3;
      const bananaClone = { ...this.fruits[bananaPosition] };
      this.fruits[bananaPosition] = bananaClone;
    }, 5000);
  }
}
```

That code results in rendering a list of fruits as before, and after 5 seconds it will shallow copy the banana object and assign it to the same position as it was before. So nothing really changed! There are the same items, in the same place. However ngFor will re-crate the banana DOM node, because even though none of the values changed, the reference to that object changed so Angular will recreate that DOM node. To avoid that we have to tell Angular not to compare object references but instead use something unique to distinguish the real change. In our case, the id of each fruit will be perfect for that.

The trackBy function is a way to provide a custom mechanism for tracking items.

We need to add function into ngFor declaration:

```HTML
<ul>
  <li *ngFor="let fruit of fruits; trackBy: trackFruit">
    {{ fruit.emoji }}
  </li>
</ul>
```

Now we need to supply definition for that function:

```Typescript
@Component({
  selector: 'my-app',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css'],
})
export  class  AppComponent  implements  OnInit {

fruits = [
  { id: 'f1', emoji: '🍎', name: 'apple' },
  { id: 'f2', emoji: '🍐', name: 'pear' },
  { id: 'f3', emoji: '🍌', name: 'banana' },
  { id: 'f6', emoji: '🥥', name: 'coconut' },
];

  ngOnInit() {
    setTimeout(() => {
      const bananaPosition = 3;
      const bananaClone = { ...this.fruits[bananaPosition] };
      this.fruits[bananaPosition] = bananaClone;
    }, 5000);
  }

  trackFruit(index, fruit) {
    return fruit.id;
  }
}
```


The tracking function takes a few arguments, and the first one is an index of the element, next one is the actual object from the array.

### Virtual scroll

If you have a very long list of items with a complex template, you can run into the problem of adding too many nodes into the DOM, which will cause performance issues.

In such situations, you may find it helpful not to add every item into DOM at the start but instead, keep track of what is currently visible for the user and, according to that, display only those items which could be seen.

Many libraries provide such a mechanism. I, for instance, prefer to use the Scrolling module from Material CDK. You can find it here: [https://material.angular.io/cdk/scrolling/overview](https://material.angular.io/cdk/scrolling/overview).

## Summary

NgFor directive is a powerful technique for rendering multiple items. It provides many useful extra features, such as:

-   exposing information about the position of element - `index`
-   information about odd and even elements - `odd` | `even`
-   information about first and last element - `first` | `last`

Keep in mind the usefulness of the trackBy function if you run into performance issues.