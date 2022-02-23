---
title: No div around - writing semantic progress indicators in Angular - Angular Tutorials | indepth.dev
author_name: Nikita Barsukov
author_link: https://twitter.com/nsbarsukov
discussion_link: https://github.com/indepth-dev/community/discussions/254
display_name: No div around - writing semantic progress indicators in Angular
---

# No div around: writing semantic progress indicators in Angular

> In this article, I will show how to develop semantic Angular-components
[ProgressBar](https://taiga-ui.dev/components/progress-bar) and [ProgressCircle](https://taiga-ui.dev/components/progress-circle).

Developing a progress indicator is one of the simplest tasks for the frontend-developer. 
All you need is basic knowledge of HTML and CSS.
JavaScript is used just to calculate the percentage of task completion.

However, this simplicity is deceiving.
The Internet is teeming with a plethora of community solutions proposing to build the progress indicator with nested `div`-containers and CSS to spice things up.
Steer clear from these solutions! Those are almost a crime against humanity since they worsen template semantics and violate accessibility.

In this article, I will show how in [Taiga UI](https://github.com/Tinkoff/taiga-ui) we developed Angular-components
[ProgressBar](https://taiga-ui.dev/components/progress-bar) and [ProgressCircle](https://taiga-ui.dev/components/progress-circle).

## Developing ProgressBar
### Task formulation:
There is a built-in HTML-tag `<progress />`. [MDN claims](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/progress):
> The `<progress>` HTML element displays an indicator showing the completion progress of a task, typically displayed as a progress bar.

The important thing is that each browser differently renders this HTML-tag.
We require such a progress indicator which looks in the same way on all browsers.

### Wrong solution / De/IVious solution 
The first idea which springs to one’s mind might be the superficial one:
"Let us not use the built-in HTML-tag at all. It is not fancy and looks differently in each browser".
This idea cultivates another wrong solution within the community.
The proposal revolves around creating two `<div />`-containers: the first container is the progress’s track, and the second one is the progress’s indicator.
Following step is to customize the containers and change the width of the progress’s indicator via JavaScript (from 0 to 100%).

The simplified version of this wrong solution is the following. The HTML-file contains the two mentioned containers:
```html
<div class="track">
    <div class="indicator"></div>
</div>
```
The CSS-file:
```css
.track {
    position: relative;
    background-color: grey;
    
    width: 300px;
    height: 20px;
}

.indicator {
    position: absolute;
    top: 0;
    left: 0;
    height: 100%;
    width: 50%; /* change it via JS */
    background-color: yellow;
}
```
It does work visually. But that is only visually.
However, what if a visually impared user (e.g., cannot distinguish colors or completely blind)
decides to visit the website and doesn’t see the progress indicator.

In such cases people use screen readers.
These programs read outloud everything important that is happening on the screen to the user.
Nevertheless, in the case of the `div` solution the screen reader will not be able to comprehend this type of progress indicator.
It will "see" only two `<div />`-containers filled in with different colors and will not pronounce it to the user.

This proposed solution not only worsens HTML-semantics, but also violates accessibility of our web application.
Of course, we can help the screen reader to detect progress indicator: we can set
[role="progressbar"](https://www.w3.org/TR/wai-aria-1.1/#progressbar) and also add
[aria-valuenow](https://www.w3.org/TR/wai-aria-1.1/#aria-valuenow),
[aria-valuemax](https://www.w3.org/TR/wai-aria-1.1/#aria-valuemax) and
[aria-valuemin](https://www.w3.org/TR/wai-aria-1.1/#aria-valuemin) on the outer container.
However, there is no need to reinvent the wheel and overcomplicate basic notions!

### Improving DIV solution
There is a popular approach which can correct the accessibility of the previous solution.
We can put visually-hidden native `<progress />` HTML-tag inside our custom progress-component.
Then we should pass attributes `value` and `max` to it and leave everything else as it was in the previous solution.

There are several approaches on how to visually hide an HTML-container but leave it detectable for screen readers.
Usually, it is `sr-only` CSS-class.
For example, the popular CSS-frameworks
[Bootstrap](https://getbootstrap.com/docs/4.0/utilities/screenreaders) and
[Tailwind CSS](https://tailwindcss.com/docs/screen-readers)
have such classes.

> Warning! Do not use `display: none`, `height: 0` and `width: 0`.
> They hide content not only visually but also for screen readers.
> Read more about this in the article
> ["CSS in Action: Invisible Content Just for Screen Reader Users"](https://webaim.org/techniques/css/invisiblecontent).

After it, the progress indicator is not only visually fancy, but also does not violate the accessibility principle.
For example, built-in macOS screen reader pronounces it as «n-percentage progress indicator» (where `n` – 100 × `value` / `max`).

Despite the progress, we are still not content with the template's semantics!
Moreover, we should create many `@Input()`-properties in our component
to pass all required attributes to injected native `<progress />`-tag without any mutations.
For now, it is only `value` and `max`, and who knows, maybe tomorrow we will need to add `id` or one of `data-*` attributes.
As a result, our component carries the attributes, which are of no use!


### Our solution with Angular attribute component
We propose a solution which requires no extra HTML-tags.
All we need to do is to extend the native `<progress />`-element.
Whenever we extend any native HTML-element,
the [official Angular documentation](https://angular.io/guide/accessibility#augmenting-native-elements)
recommends creating a component that uses an attribute selector with this element.
This practice is actively used in our UI-Kit library Taiga: for example,
[button](https://github.com/Tinkoff/taiga-ui/blob/main/projects/core/components/button/button.component.ts#L33),
[link](https://github.com/Tinkoff/taiga-ui/blob/main/projects/core/components/link/link.component.ts#L28) and
[label](https://github.com/Tinkoff/taiga-ui/blob/main/projects/core/components/label/label.component.ts#L18) components.

Let’s create Typescript-file with our attribute component:
```typescript
import {
  ChangeDetectionStrategy,
  Component,
  HostBinding,
  Input
} from '@angular/core';

@Component({
  selector: 'progress[tuiProgressBar]',
  template: '',
  styleUrls: ['./progress-bar.component.less'],
  changeDetection: ChangeDetectionStrategy.OnPush,
})
export class TuiProgressBarComponent {
  @Input()
  @HostBinding('style.--tui-progress-color')
  color?: string;
}
```

The Less-file includes some [less-mixins](https://lesscss.org/features/#mixins-feature).
The following one clears all built-in customization from the browser:
```less
.clearProgress() {
    -webkit-appearance: none;
    -moz-appearance: none;
    appearance: none;
    border: none;
}
```

And this one help to customize progress’s track:
```less
.progressTrack(@property, @value) {
    @{property}: @value; // Edge | Mozilla

    &::-webkit-progress-bar {
        @{property}: @value; // Chrome | Opera | Safari
    }
}
```

The last mixin helps to customize the color of the progress indicator:
```less
.progressIndicatorColor(@color) {
    color: @color; // Not Chromium Edge

    &::-webkit-progress-value {
        background: @color; // Chromium Edge | Chrome | Opera | Safari
    }

    &::-moz-progress-bar {
        background: @color; // Mozilla
    }
}
```

Finally, apply all created mixins to `:host`-element of our component
(reminder: it is a native `<progress />` on which our attribute component was applied).
```less
:host {
    .clearProgress();
    .progressIndicatorColor(var(--tui-progress-color, currentColor));
    .progressTrack(background-color, grey);

    color: yellow;
}
```

That's all! We developed a nice Angular attribute component which is wrapped around a native `<progress />` element.
The color of this indicator can be set via the CSS `color`-property, or via the input-property of the component
(for example, if we want to create a complex gradient color).

In the code, the component declares in the following way:
````html
<progress tuiProgressBar [value]="60" [max]="100"></progress>
````

You can see the component in action [on the showcase of our project Taiga UI](https://taiga-ui.dev/components/progress-bar).
The final source code is available [on GitHub](https://github.com/Tinkoff/taiga-ui/tree/main/projects/kit/components/progress/progress-bar).

## Developing ProgressCircle
Unfortunately, creation of the [ProgressCircle](https://taiga-ui.dev/components/progress-circle) component
from only native `<progress />`-element is not possible.
We need some extra template tags.

And here the state of things is the same, curious Internet users might stumble upon various frivolous solutions
which propose to create a circular progress indicator from `div`-containers which have been rounded using `border-radius: 50%`.
Also, such solutions use a significant amount of JavaScript.

Yet, since we are in 'no-div club', this task can be resolved without appeals to `div`-s.
We will use svg-tag `<circle />`.
Moreover, we will use as little Javascript as possible.
Less-preprocessor and CSS variables help us with it.

Let’s create a Typescript-file of our future component:
```typescript
import {
    ChangeDetectionStrategy,
    Component,
    HostBinding,
    Input,
} from '@angular/core';

@Component({
    selector: 'tui-progress-circle',
    templateUrl: './progress-circle.template.html',
    styleUrls: ['./progress-circle.style.less'],
    changeDetection: ChangeDetectionStrategy.OnPush,
})
export class TuiProgressCircleComponent {
    @Input()
    value = 0;

    @Input()
    max = 1;

    @Input()
    @HostBinding('style.--tui-progress-color')
    color: string | null = null;

    @Input()
    @HostBinding('attr.data-size')
    size: 'm' | 'l' = 'm';

    @HostBinding('style.--progress-percentage')
    get progressPercentage(): number {
        return this.value / this.max;
    }
}
```

The HTML-file contains:
```html
<progress
  class="hidden-progress"
  [value]="value"
  [max]="max"
></progress>

<svg class="svg" height="100%" width="100%" aria-hidden="true">
  <circle
    class="track"
    cx="50%"
    cy="50%"
  ></circle>

  <circle
    class="progress"
    cx="50%"
    cy="50%"
  ></circle>
</svg>
```

And the last step – creation of the LESS-file.
We will use many features of Less:
[mixins](https://lesscss.org/features/#mixins-feature),
[Maps](https://lesscss.org/features/#maps-feature), and
built-in [Math functions](https://lesscss.org/functions/#math-functions).

Firstly, let’s create map-constants which store values for the different sizes of circular indicators
(there are 4 sizes in our project, but for the sake of code’s simplicity we will leave only 2).
It is worth noting that Safari does not support `rem`-units inside `svg`-elements.
However, it supports `em`-units. Therefore, we set `font-size: 1rem` for the host-element to use `em`-units inside it.
```less
@width: {
    @m: 2em;
    @l: 7em;
};

@track-stroke: {
    @m: 0.5em;
    @l: 0.25em;
};

@progress-stroke: {
    @m: 0.5em;
    @l: 0.375em;
};
```

The process of progress’s filling happens due to setting of `stroke-dasharray` and recalculations of the `stroke-dashoffset`.
Look into the article ["Building a Progress Ring, Quickly"](https://css-tricks.com/building-progress-ring-quickly) to understand how it works.
Our solution is just an improved version of what was suggested by its author.
Create mixin which calculates the state of the progress indicator for different component’s size:
```less
.circle-params(@size) {
    width: @width[ @@size];
    height: @width[ @@size];

    .track {
        r: (@width[ @@size] - @track-stroke[ @@size]) / 2;
        stroke-width: @track-stroke[ @@size];
    }

    .progress {
        @radius: (@width[ @@size] - @progress-stroke[ @@size]) / 2;
        @circumference: 2 * pi() * @radius;

        r: @radius;
        stroke-width: @progress-stroke[ @@size];
        stroke-dasharray: @circumference;
        stroke-dashoffset: calc(@circumference - var(--progress-percentage) * @circumference);
    }
}
```

Finally, apply our mixin on `host`-element and add some cosmetic improvements:
```less
:host {
    display: block;
    position: relative;
    color: yellow;
    transform: rotate(-90deg);
    transform-origin: center;
    font-size: 1rem;

    &[data-size='m'] {
        .circle-params(m);
    }

    &[data-size='l'] {
        .circle-params(l);
    }
}

.track {
    fill: transparent;
    stroke: grey;
}

.progress {
    fill: transparent;
    stroke: var(--tui-progress-color, currentColor);
    transition: stroke-dashoffset 300 linear;
}

.hidden-progress {
    .sr-only(); // see this mixin in the chapter with ProgressBar
}

.svg {
    overflow: unset;
}
```

That is all! You can see the component in action
[on the showcase of our Taiga UI project](https://taiga-ui.dev/components/progress-circle).
The final source code is available
[on GitHub](https://github.com/Tinkoff/taiga-ui/tree/main/projects/kit/components/progress/progress-circle).

## Wrapping up
There are no absolutely correct solutions in the frontend-development.
The same functionality can be implemented in different ways.
But there are superficial and lazy solutions, which deliver the final product by sacrificing convenience and optimization.
I have shown what mistakes can be made if you want to create progress indicators: poor semantics and neglected accessibility.

In this article, I have shown my point of view on these components.
I don’t claim that my solutions are the best ones in the field.
And still, during development of those, I took all mentioned problems into consideration,
ensured compatibility with all modern browsers and got the simple solutions with minimum usage of JavaScript.
