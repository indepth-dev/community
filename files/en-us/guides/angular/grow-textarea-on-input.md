---
title: How to make textarea grow on input in Angular - Angular Tutorials | indepth.dev
author_name: Touhid Rahman
author_link: https://twitter.com/touhidrahman87
discussion_link: https://github.com/indepth-dev/community/discussions/159
---

# **How to make textarea grow on input in Angular**

![img](https://media.giphy.com/media/u0PXN3EErcDJmQHJ1v/giphy.gif?cid=790b761117c2ed6130bab1fac77819c78e423e266c565741&rid=giphy.gif&ct=g)

Recently I faced the task of implementing a basic auto-expanding text field at my workplace. Basically, it was a chat input that is initially presented as a single-line text field. As the user writes on, the field should expand up to 4 lines and then scroll. The first thing that came to my mind was just to pick up a library. Of course, there are npm packages for auto-sizing a text field in Angular that do the job very well. However, that library was going to cost me double-digit kilobytes when extracted. A quick googling revealed a few ways I could achieve that without adding a library. And the bare minimum functionality was sufficient for my use case. So if you are too, satisfied with just a quick and simple solution, then this guide is for you!

## The Directive

Let’s create a directive so that we can reuse it in other text areas.

```javascript
@Directive({
  selector: "[appTextareaAutosize]"
})
export class TextareaAutosizeDirective {
  constructor(private element: ElementRef) {}
}
```

Please note the selector name. We are using an attribute selector, meaning - whenever a textarea will have this exact attribute name, the directive will apply to that. Alternatively, you could target all the textarea of your app by modifying the selector to `textarea`.

We get a hand on the underlying host DOM using the ElementRef from `@angular/core`. It injects the reference to the host DOM where the directive will be applied  -  for our case, a textarea element. We can access the textarea in the controller/directive using `nativeElement` property of the ElementRef.

To define how many rows should the textarea show initially and how many to grow to, we will use two inputs . If no inputs are explicitly provided then the directive will default to a minimum of 2 rows and a maximum of 4 rows.

```javascript
export class TextareaAutosizeDirective {
  @Input() minRows = 2;
  @Input() maxRows = 4;
  ...
}
```

To preserve a couple of internal states, introduce and initialize a few variables.

```javascript
import { Directive, ElementRef, HostListener, Input } from "@angular/core";

@Directive({
 selector: "[appTextareaAutosize]"
})
export class TextareaAutosizeDirective {
 @Input() minRows = 2;
 @Input() maxRows = 4;

 private input: HTMLTextAreaElement;
 private offsetHeight = 0;
 private readonly avgLineHeight = 20;

 construtor(private element: ElementRef) {
   this.input = this.element.nativeElement;
 }
}
```

We need to watch the text field input value to determine whether the textarea should grow. Let's bind the input event to the `HostListener`. Now whenever the HTML `oninput` event will occur to the text field, the `onInput` method will trigger. This is the place where we do some calculations.

```javascript
@HostListener("input")
onInput(): void {

  if (this.offsetHeight <= 0) {
    this.offsetHeight = this.input.scrollHeight;
  }

  this.input.rows = this.minRows;

  const rows = Math.floor(
    (this.input.scrollHeight - this.offsetHeight) /
    this.avgLineHeight
  );

  const rowsCount = this.minRows + rows;

  this.input.rows = rowsCount > this.maxRows
     ? this.maxRows
     : rowsCount;
}
```

- `offsetHeight` is determined only once at the beginning to guess how much to offset scrolling when a text row is inserted.
- First, we set the rows to our input `minRows`. This will help the text field reduce in size when the user deletes input text.
- Then we calculate the number of rows required from the element's scroll height divided by line-height.
- Finally, we set the new row count to the element.


## Usage

Now we can use the directive:

```html
<textarea
    appTextareaAutosize
    rows="2"
    [minRows]="2"
    [maxRows]="4"
    placeholder="Type your message here..."
    class="text-input"
></textarea>
```

Below you can see the text field auto-size in action with some CSS applied:

```css
.text-input {
  padding: 6px 10px;
  color: #333;
  border: 1px solid #444;
  border-radius: 6px;
  width: 100%;
  resize: none; /* hide browser provided resize handles */
  overflow-y: overlay;
  font-family: Arial, sans-serif;
  font-size: 14px;
  line-height: 20px; /* equal to avgLineHeight in controller */
  -ms-overflow-style: none; /* hide scrollbar in IE and Edge */
  scrollbar-width: none; /* hide scrollbar in Firefox */
}

.text-input::-webkit-scrollbar {
  display: none; /* hide scrollbar in Chrome and variants */
}
```

Check out the end result in the embedded Codesandbox instance:

[https://codesandbox.io/s/quick-and-dirty-textarea-autosize-cggu2](Codesandbox)

Complete code for your reference:

[https://gist.github.com/touhidrahman/c5dfb10c530397cd94b5143bbeb0987b](Gist)

## Conclusion

Directive is a powerful building block in Angular that enhances reusability and single responsibility. The auto-sizing textarea is a super simple example of that. When developing an angular application, we should always keep an eye on whether we can disassemble some code into directives or not.

Thanks for reading!
