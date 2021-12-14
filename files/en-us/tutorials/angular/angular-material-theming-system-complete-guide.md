---
title: Angular Material Theming System: Complete Guide - Angular Tutorials | indepth.dev
author_name: Dharmen Shah
author_link: https://twitter.com/shhdharmen
discussion_link: https://github.com/indepth-dev/community/discussions/228
display_name: Angular Material Theming System: Complete Guide
---

# Angular Material Theming System: Complete Guide

> In the latest releases of Angular Material, the SASS theming API has been reworked. In this article, we will learn about custom themes, modifying typography and much more using new SASS mixins.

In this article, you will learn what Angular Material Theming is and what are some recent changes to adhere to the new module system of SASS.

Then, we will set up a project with Angular Material. We will then add a custom theme in it and understand some important mixins, `core`, `define-palette`, `define-light-theme` and we will apply custom theme to Angular Material components. And we will also learn how to use a pre-built theme.

After setting up the theme, we will understand typography and also modify it for header tags (`<h1>`, `<h2>`, etc.) for the application.

Next, we will create a dark theme using `define-dark-theme`. We will implement lazy loading for dark theme, so that it only loads when needed.

After adding support for multiple themes, we will learn how to apply Angular Material’s theming to custom components. We will take an example of an `alert` component and apply themes to it.

We will also learn about how to customize styles of Angular Material components. We will take the example of `MatButton` and add new variants for it.

And at last, we are going to learn how to update an old code-base with Angular Material version 10 to the latest version, i.e. 13. We will see how to resolve SASS errors and what major changes are made in each release.


## Angular Material UI Components

The Angular team builds and maintains both common UI components and tools to help us build our own custom components. `@angular/material` is [Material Design](https://material.io/) UI components for Angular applications.

Angular Material also provides tools that help developers build their own custom components with common interaction patterns.


## Angular Material’s Theming System

In Angular Material, a theme is a collection of color and typography options. Each theme includes three palettes that determine component colors: primary, accent and warn.

Angular Material's theming system comes with a predefined set of rules for color and typography styles. The theming system is based on [Google's Material Design](https://material.io/design/material-theming/overview.html) specification. You can also customize color and typography styles for components in your application.


## SASS Basics

Before moving ahead, It would be great if you have familiarity with [SASS basics](https://sass-lang.com/guide), including [variables](https://sass-lang.com/documentation/variables), [functions](https://sass-lang.com/documentation/at-rules/function), [mixins](https://sass-lang.com/documentation/at-rules/mixin), and [use](https://sass-lang.com/documentation/at-rules/use).


## New changes of SASS in Angular Material

SASS introduced a new module system, including a migration from `@import` to `@use` in 2019. The [@use](https://sass-lang.com/documentation/at-rules/use) rule loads mixins, functions, and variables from other SASS stylesheets, and combines CSS from multiple stylesheets together. Stylesheets loaded by `@use` are called "modules".

By switching to `@use` syntax, we can more easily determine what CSS is unused, and reduce the size of the compiled CSS output. Each module is included only once no matter how many times those styles are loaded.

Angular Material v12 included a migration from `@import` usage to `@use` for all imports into the Angular Material SASS styles. They updated their code base for all styles with version 12. You can check out that [particular release](https://github.com/angular/components/releases/tag/12.0.0) for more information.

This refactor of the theming API surface is easier to understand and read, helping developers like us take better advantage of this new module system.


## Setup project with Angular Material

_Note: While writing this article, I used Angular version 13 and the approach described in this article should also work for version 12. For older versions, you can jump to [the update guide](#angular-update-guide)._

In this section, we are going to learn how to use the new mixins and functions like `core-theme`, `all-components-theme`, `define-palette`, etc. To summarize, below are main tasks which we will be doing:

1. Create a custom theme
2. Using a pre-built theme
3. Modify typography
4. Create a dark theme
5. Apply Angular Material’s theme to custom component
6. Customizing Angular Material Component Styles

Let’s first create a new Angular Project with SASS:

```bash
ng new my-app --style=scss --defaults
```

Use the Angular CLI's installation [schematic](https://material.angular.io/guide/schematics) to set up your Angular Material project by running the following command:

```bash
ng add @angular/material
```

The `ng add` command will install Angular Material, the [Component Dev Kit (CDK)](https://material.angular.io/cdk/categories), [Angular Animations](https://angular.io/guide/animations) and ask you the following questions to determine which features to include:

1. Choose a prebuilt theme name, or "custom" for a custom theme: Select **Custom**
2. Set up global Angular Material typography styles?: **Yes**
3. Set up browser animations for Angular Material?: **Yes**

You're done! Angular Material is now configured to be used in your application.

                                                                                  
### Create a custom theme

A **theme file** is a SASS file that uses Angular Material SASS mixins to produce color and typography CSS styles.

Let’s jump to `src/styles.scss` file and have a look at our theme:

```scss
// src/styles.scss

@use "@angular/material" as mat;

@include mat.core();

$my-app-primary: mat.define-palette(mat.$indigo-palette);
$my-app-accent: mat.define-palette(mat.$pink-palette, A200, A100, A400);
$my-app-warn: mat.define-palette(mat.$red-palette);

$my-app-theme: mat.define-light-theme(
  (
    color: (
      primary: $my-app-primary,
      accent: $my-app-accent,
      warn: $my-app-warn,
    ),
  )
);

@include mat.all-component-themes($my-app-theme);

html,
body {
  height: 100%;
}
body {
  margin: 0;
  font-family: Roboto, "Helvetica Neue", sans-serif;
}
```

Let’s break the above code into pieces to understand more.

#### The `core` mixin

```scss
@include mat.core();
```

The first thing in the theme file you will notice is the `core` mixin. Angular Material defines a mixin named `core` that includes prerequisite styles for common features used by multiple components, such as [ripples](https://github.com/angular/components/blob/0d3a730631fae77ce4af49b4b9853da2a5fe09f9/src/material/core/_core.scss#L13). The core mixin must be included exactly once for your application, even if you define multiple themes.


#### Defining a theme

Angular Material represents a theme as a SASS map that contains your color and typography choices. Colors are defined through a palette. 

A palette is a collection of colors representing a portion of color space. Each value in this collection is called a hue. In Material Design, each hue in a palette has an identifier number. These identifier numbers include 50, and then each 100 value between 100 and 900. The numbers order hues within a palette from lightest to darkest. Angular Material represents a palette as a SASS map.


##### The `define-palette` function

```scss
$my-app-primary: mat.define-palette(mat.$indigo-palette);
$my-app-accent: mat.define-palette(mat.$pink-palette, A200, A100, A400);
$my-app-warn: mat.define-palette(mat.$red-palette);
```

To construct a theme, 2 palettes are required: `primary` and `accent`, and `warn` palette is optional.

The `define-palette` SASS function accepts a color palette, as well as four optional hue numbers. These four hues represent, in order: the "default" hue, a "lighter" hue, a "darker" hue, and a "text" hue.

Components use these hues to choose the most appropriate color for different parts of themselves. For example, [`MatButton`’s theme](https://github.com/angular/components/blob/0d3a730631fae77ce4af49b4b9853da2a5fe09f9/src/material/button/_button-theme.scss#L64) uses the hues to generate font colors:


```scss
// src/material/button/_button-theme.scss
// content reduced for brevity

// Applies a property to an mat-button element for each of the supported palettes.
@mixin _theme-property($theme, $property, $hue) {
  $primary: map.get($theme, primary);
  $accent: map.get($theme, accent);
  $warn: map.get($theme, warn);
  $background: map.get($theme, background);
  $foreground: map.get($theme, foreground);

  &.mat-primary {
    #{$property}: theming.get-color-from-palette($primary, $hue);
  }
  &.mat-accent {
    #{$property}: theming.get-color-from-palette($accent, $hue);
  }
  &.mat-warn {
    #{$property}: theming.get-color-from-palette($warn, $hue);
  }

  &.mat-primary,
  &.mat-accent,
  &.mat-warn,
  &.mat-button-disabled {
    &.mat-button-disabled {
      $palette: if($property == "color", $foreground, $background);
      #{$property}: theming.get-color-from-palette($palette, disabled-button);
    }
  }
}

@mixin color($config-or-theme) {
  $config: theming.get-color-config($config-or-theme);
  $foreground: map.get($config, foreground);

  .mat-button,
  .mat-icon-button,
  .mat-stroked-button {
    @include _theme-property($config, "color", text);
  }
}
```

In our example, we have used predefined palettes, i.e. `$indigo-palette`, `$pink-palette` and `$red-palette`. You can check out other palettes at the Angular Material GitHub [repo’s file:](https://github.com/angular/components/blob/master/src/material/core/theming/_palette.scss)


```scss
// src/material/core/theming/_palette.scss
// content reduced for brevity

$red-palette: (
  50: #ffebee,
  100: #ffcdd2,
  200: #ef9a9a,
  300: #e57373,
  // ...
  contrast: (
    50: $dark-primary-text,
    100: $dark-primary-text,
    200: $dark-primary-text,
    300: $dark-primary-text,
    // ...
  )
);

$pink-palette: (
  50: #fce4ec,
  100: #f8bbd0,
  200: #f48fb1,
  300: #f06292,
  // ...
  contrast: (
    50: $dark-primary-text,
    100: $dark-primary-text,
    200: $dark-primary-text,
    300: $dark-primary-text,
    // ...
  )
);
```


###### Create your own palette

You can also create your own palettes by defining a SASS map like below:

```scss
$indigo-palette: (
 50: #e8eaf6,
 100: #c5cae9,
 200: #9fa8da,
 300: #7986cb,
 // ... continues to 900
 contrast: (
   50: rgba(black, 0.87),
   100: rgba(black, 0.87),
   200: rgba(black, 0.87),
   300: white,
   // ... continues to 900
 )
);
```


##### The `define-light-theme` function

```scss
$my-app-theme: mat.define-light-theme(
  (
    color: (
      primary: $my-app-primary,
      accent: $my-app-accent,
      warn: $my-app-warn,
    ),
  )
);
```

You can construct a theme by calling either `define-light-theme` or `define-dark-theme` with the result from `define-palette`. The choice of a light versus a dark theme determines the background and foreground colors used throughout the components.


#### Applying a theme to components

```scss
@include mat.all-component-themes($my-app-theme);
```

Angular Material offers a "theme" mixin that emits styles for both color and typography and It’s the `all-component-themes` mixin.

You can check the source file: [`src/material/core/theming/_all-theme.scss`](https://github.com/angular/components/blob/97ec228ada31e55f76a7e0171c715bb0b8d07e9c/src/material/core/theming/_all-theme.scss#L40) to see the mixin `all-component-themes`:

```scss
// src/material/core/theming/_all-theme.scss

@mixin all-component-themes($theme-or-color-config) {
  $dedupe-key: 'angular-material-theme';
  @include theming.private-check-duplicate-theme-styles($theme-or-color-config, $dedupe-key) {
    @include core-theme.theme($theme-or-color-config);
    @include autocomplete-theme.theme($theme-or-color-config);
    @include badge-theme.theme($theme-or-color-config);
    @include bottom-sheet-theme.theme($theme-or-color-config);
    @include button-theme.theme($theme-or-color-config);
    // other material components' themes...
  }
}
```

Additionally, there is a "color" mixin that emits all components' color styles and a "typography" mixin that emits all components’ typography styles. They are `all-component-colors` and `all-component-typographies` mixins.

The `all-component-colors` mixin is present at[`src/material/core/color/_all-color.scss`](https://github.com/angular/components/blob/97ec228ada31e55f76a7e0171c715bb0b8d07e9c/src/material/core/color/_all-color.scss#L5) has:

```scss
// src/material/core/color/_all-color.scss

@mixin all-component-colors($config-or-theme) {
  $config: if(theming.private-is-theme-object($config-or-theme),
      theming.get-color-config($config-or-theme), $config-or-theme);

  @include all-theme.all-component-themes((
    color: $config,
    typography: null,
    density: null,
  ));
}
```

And `all-components-typography` mixin is present at [`src/material/core/typography/_all-typography.scss`](https://github.com/angular/components/blob/97ec228ada31e55f76a7e0171c715bb0b8d07e9c/src/material/core/typography/_all-typography.scss#L42):

```scss
// src/material/core/typography/_all-typography.scss

@mixin all-component-typographies($config-or-theme: null) {
  $config: if(theming.private-is-theme-object($config-or-theme),
      theming.get-typography-config($config-or-theme), $config-or-theme);

  @include badge-theme.typography($config);
  @include typography.typography-hierarchy($config);
  @include autocomplete-theme.typography($config);
  @include bottom-sheet-theme.typography($config);
  @include button-theme.typography($config);
  // other components' typographies
}
```

These mixins emit styles for all 35+ components in Angular Material. This will produce unnecessary CSS, except when your application is using every single component from the library. Let’s look at the `styles` size after the `build` command and then I’ll show you how to reduce it:

![showing styles bundle size](https://i.imgur.com/pJYl4W7.png "showing styles bundle size")

                              
##### Include only used components’ themes

Just like `all-component-colors`, `all-component-typographies` and `all-component-themes`, each Angular Material component has a `color`, a `typography` and a `theme` mixin.

You can checkout `MatButton`’s mixins at [`src/material/button/_button-theme.scss`](https://github.com/angular/components/blob/master/src/material/button/_button-theme.scss):

```scss
// src/material/button/_button-theme.scss
// content reduced for brevity

@mixin color($config-or-theme) {
  $config: theming.get-color-config($config-or-theme);
  $primary: map.get($config, primary);
  $accent: map.get($config, accent);
  $warn: map.get($config, warn);
  // sets up color for buttons
}

@mixin typography($config-or-theme) {
  $config: typography.private-typography-to-2014-config(
      theming.get-typography-config($config-or-theme));
  .mat-button, .mat-raised-button, .mat-icon-button, .mat-stroked-button,
  .mat-flat-button, .mat-fab, .mat-mini-fab {
    font: {
      family: typography-utils.font-family($config, button);
      size: typography-utils.font-size($config, button);
      weight: typography-utils.font-weight($config, button);
    }
  }
}

@mixin theme($theme-or-color-config) {
  $theme: theming.private-legacy-get-theme($theme-or-color-config);
  @include theming.private-check-duplicate-theme-styles($theme, 'mat-button') {
    $color: theming.get-color-config($theme);
    $typography: theming.get-typography-config($theme);

    @if $color != null {
      @include color($color);
    }
    @if $typography != null {
      @include typography($typography);
    }
  }
}
```

We can apply the styles for each of the components used in the application by including each of their theme SASS mixins.

First, we will remove `all-component-themes` from `styles.scss` and instead, add `core-theme`:

```scss
// @include mat.all-component-themes($my-app-theme); <-- removed
@include mat.core-theme($my-app-theme);
```

`core-theme` emits theme-dependent styles for common features used across multiple components, like ripples.

Next, we need to add component related styles. In this example, we are only going to use [`MatButton`](https://material.angular.io/components/button/overview), so we will add `button-theme`:

```scss
@include mat.button-theme($my-app-theme);
```

You can add other components’ `theme`’s in the same way. But, `core-theme` is only needed once per theme. Let’s look at the `styles` size now after build.

![showing styles bundle size after optimizations](https://i.imgur.com/vPVoi24.png "showing styles bundle size after optimizations")

Notice how using only the needed components’ themes reduces the style's size. In our case, it was 72.31 kB earlier and it’s reduced to 23.52 kB, which is almost 58% less.

For better code management, we will move theme related code into `styles/themes/_light.scss`:

```scss
// src/styles/themes/_light.scss

@use "sass:map";
@use "@angular/material" as mat;

$my-app-light-primary: mat.define-palette(mat.$indigo-palette);
$my-app-light-accent: mat.define-palette(mat.$pink-palette, A200, A100, A400);

$my-app-light-warn: mat.define-palette(mat.$red-palette);

$my-app-light-theme: mat.define-light-theme(
  (
    color: (
      primary: $my-app-light-primary,
      accent: $my-app-light-accent,
      warn: $my-app-light-warn,
    ),
  )
);
```

And use the same in `styles.scss`:

```scss
// styles.scss

@use "@angular/material" as mat;

@use "./styles/themes/light";

@include mat.core();

@include mat.core-theme(light.$my-app-light-theme);
@include mat.button-theme(light.$my-app-light-theme);

html,
body {
  height: 100%;
}
body {
  margin: 0;
  font-family: Roboto, "Helvetica Neue", sans-serif;
}
```


#### Output after creating custom theme

Let’s add a `[mat-raised-button]` in application and see how it looks:

```html
<button mat-raised-button color="primary">Raised</button>
<button mat-raised-button color="accent">Accent</button>
<button mat-raised-button color="warn">Warn</button>
```

And the output should look like below:

![output after creating custom theme](https://i.imgur.com/9ghL1oN.gif "output after creating custom theme")


### Using a pre-built theme

When we installed Angular Material, we selected “Custom” in theme selection. If you want any pre-built theme, you can select any theme instead of “Custom”. There are 4 pre-built themes provided:

| Theme                    | Light or dark?     | Palettes (primary, accent, warn)     |
|--------------------------|--------------------|--------------------------------------|
| deeppurple-amber.css     | Light              | deep-purple, amber, red              |
| indigo-pink.css          | Light              | indigo, pink, red                    |
| pink-bluegray.css        | Dark               | pink, bluegray, red                  |
| purple-green.css         | Dark               | purple, green, red                   |

For instance, if you want to use `indigo-pink.css`’s theme, you just need to include that file in the `styles` array of your project’s `angular.json` file:

```json
"styles": [
    "./node_modules/@angular/material/prebuilt-themes/indigo-pink.css",
    // other styles
],
```


### Modify typography

_Typography is a way of arranging type to make text legible, readable, and appealing when displayed. Angular Material's theming system supports customizing the typography settings for the library's components. Additionally, Angular Material provides APIs for applying typography styles to elements in your own application._

When we installed Angular Material through schematics, it set up the font asset for us in `index.html`:

```html
<link href="https://fonts.googleapis.com/css2?family=Roboto:wght@300;400;500&display=swap" rel="stylesheet">
```

And to support `Roboto`, it also added some global styles in `styles.scss`:

```scss
body {
  font-family: Roboto, "Helvetica Neue", sans-serif;
}
```


#### Typography level

In the Material theme, each set of typography is categorised in levels based on which part of the application's structure it corresponds to, such as a header. You can learn more about it at  [typography levels from the 2014 version of the Material Design specification](https://material.io/archive/guidelines/style/typography.html#typography-styles).

| Name             | CSS Class                               | Native Element     | Description                                                                     |
|------------------|-----------------------------------------|--------------------|---------------------------------------------------------------------------------|
| display-4        | `.mat-display-4`                        | None               | 112px, one-off header, usually at the top of the page (e.g. a hero header).     |
| display-3        | `.mat-display-3`                        | None               | 56px, one-off header, usually at the top of the page (e.g. a hero header).      |
| display-2        | `.mat-display-2`                        | None               | 45px, one-off header, usually at the top of the page (e.g. a hero header).      |
| display-1        | `.mat-display-1`                        | None               | 34px, one-off header, usually at the top of the page (e.g. a hero header).      |
| headline         | `.mat-h1` or `.mat-headline`            | `<h1>`             | Section heading corresponding to the `<h1>` tag.                                |
| title            | `.mat-h2` or `.mat-title`               | `<h2>`             | Section heading corresponding to the `<h2>` tag.                                |
| subheading-2     | `.mat-h3` or `.mat-subheading-2`        | `<h3>`             | Section heading corresponding to the `<h3>` tag.                                |
| subheading-1     | `.mat-h4` or `.mat-subheading-1`        | `<h4>`             | Section heading corresponding to the `<h4>` tag.                                |
| --               | `.mat-h5`                               | `<h5>`             | --                                                                              |
| --               | `.mat-h6`                               | `<h6>`             | --                                                                              |
| body-1           | `.mat-body` or `.mat-body-1`            | Body text          | Base body text.                                                                 |
| body-2           | `.mat-body-strong` or `.mat-body-2`     | None               | Bolder body text.                                                               |
| caption          | `.mat-small` or `.mat-caption`          | None               | Smaller body and hint text.                                                     |
| button           | --                                      | --                 | Buttons and anchors.                                                            |
| input            | --                                      | --                 | Form input fields.                                                              |


##### Define a level

You can define a typography level with the `define-typography-config` SASS function. This function accepts, in order, CSS values for `font-size`, `line-height`, `font-weight`, `font-family`, and `letter-spacing`. You can also specify the parameters by name, as demonstrated in the example below.

```scss
@use '@angular/material' as mat;

$my-custom-level: mat.define-typography-level(
  $font-family: Roboto,
  $font-weight: 400,
  $font-size: 1rem,
  $line-height: 1,
  $letter-spacing: normal,
);
```


#### Typography config

Angular Material handles all those levels using **typography config**. Angular Material represents this config as a SASS map.This map contains the styles for each level, keyed by name. You can create a typography config with the `define-typography-config` SASS function. Every parameter for `define-typography-config` is optional; the styles for a level will default to Material Design's baseline if unspecified.

For this example, we will change the typography of headings and we will use [Work Sans](https://fonts.google.com/specimen/Work+Sans) as `font-family`. Let’s see how.


##### Including font assets

First, we will add the font at the bottom of `<head>` in  `index.html`:

```html
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Work+Sans:wght@300;400;500&display=swap">
```


##### Heading font-family SASS variable

Next, create a file `styles/typography/_config.scss` and create a variable in it:

```scss
// src/styles/typography/_config.scss

$heading-font-family: "'Work Sans', sans-serif";
```


##### Create config

Now it’s time to create the config using `define-typography-config` in `styles/typography/_config.scss`:

```scss
$my-app-typography: mat.define-typography-config(
  $display-4: mat.define-typography-level(112px, $font-family: $heading-font-family),
  $display-3: mat.define-typography-level(56px, $font-family: $heading-font-family),
  $display-2: mat.define-typography-level(45px, $font-family: $heading-font-family),
  $display-1: mat.define-typography-level(34px, $font-family: $heading-font-family),
  $headline: mat.define-typography-level(24px, $font-family: $heading-font-family),
  $title: mat.define-typography-level(20px, $font-family: $heading-font-family),
);
```

To customize component typography for the entire application, we will pass the custom typography config to the `core` mixin in `styles.scss`:

```scss
// src/styles.scss

@use "@angular/material" as mat;

@use "./styles/themes/light";
@use "./styles/typography/config" as typography;

@include mat.core(typography.$my-app-typography);

// rest remains same
```

Passing the typography config to core mixin will apply specified values to all Angular Material components. If a config is not specified, `core` will emit the default Material Design typography styles.


##### Theme specific typography

In addition to the `core` mixin, we can specify your typography config when including any `theme` mixin, like below:

```scss
$custom-theme: mat.define-light-theme((
   color: (
     primary: $custom-primary,
     accent: $custom-accent,
   ),
   typography: $custom-typography,
  ));
```

Because the `core` mixin always emits typography styles, specifying a typography config to a `theme` mixin results in duplicate typography CSS. You should only provide a typography config when applying your theme if you need to specify multiple typography styles that are conditionally applied based on your application's behavior.


##### Using typography styles in your application

Angular Material's native elements’ typography works if content is wrapped within the '.mat-typography` CSS class. If you check the `index.html` file, `mat-typography` class is added to the `<body>` tag. It was done when we ran `ng add @angular/material`.

If you don’t want to wrap the whole application in a `mat-typography` class, you can also use individual classes listed in the [levels table](#typography-level).


#### Output after modifying the typography

Let’s temporarily modify the content of `<body>` in `index.html`:

```html
<body>
    <!-- This header will *not* be styled because it is outside `.mat-typography` -->
    <h1>Top header (Material Typography doesn't apply here)</h1>

    <!-- This paragraph will be styled as `body-1` via the `.mat-body` CSS class applied -->
    <p class="mat-body">Introductory text</p>

    <div class="mat-typography">
      <!-- This header will be styled as `title` because it is inside `.mat-typography` -->
      <h2>Inner header</h2>

      <!-- This paragraph will be styled as `body-1` because it is inside `.mat-typography` -->
      <p>Some inner text</p>
      <app-root></app-root>
    </div>
  </body>
```

If you look at the output, you will get an idea how typography works:

![output after modifying typography](https://i.imgur.com/8USPRGx.png "output after modifying typography")

After modifying typography, below is the content of `src/styles/typography/_config.scss`:

```scss
// src/styles/typography/_config.scss

@use "@angular/material" as mat;

$heading-font-family: "'Work Sans', sans-serif";
$my-app-typography: mat.define-typography-config(
  $display-4:
    mat.define-typography-level(112px, $font-family: $heading-font-family),
  $display-3:
    mat.define-typography-level(56px, $font-family: $heading-font-family),
  $display-2:
    mat.define-typography-level(45px, $font-family: $heading-font-family),
  $display-1:
    mat.define-typography-level(34px, $font-family: $heading-font-family),
  $headline:
    mat.define-typography-level(24px, $font-family: $heading-font-family),
  $title: mat.define-typography-level(20px, $font-family: $heading-font-family),
);
```

And below is the content of `style.scss`:

```scss
// src/styles.scss

@use "@angular/material" as mat;

@use "./styles/themes/light";
@use "./styles/typography/config" as typography;

@include mat.core(typography.$my-app-typography);

@include mat.core-theme(light.$my-app-light-theme);
@include mat.button-theme(light.$my-app-light-theme);

html,
body {
  height: 100%;
}
body {
  margin: 0;
  font-family: Roboto, "Helvetica Neue", sans-serif;
}
```


### Create a dark theme

Now we will add a dark theme in the application. Create a new file called `dark.scss` in the `styles/themes` folder with the following content:

```scss
// src/styles/themes/dark.scss

@use "sass:map";
@use "@angular/material" as mat;

@use "../typography/config" as typography;
@use "../components";

$my-app-dark-primary: mat.define-palette(mat.$blue-grey-palette);
$my-app-dark-accent: mat.define-palette(mat.$amber-palette, A200, A100, A400);
$my-app-dark-warn: mat.define-palette(mat.$deep-orange-palette);
$my-app-dark-theme: mat.define-dark-theme(
  (
    color: (
      primary: $my-app-dark-primary,
      accent: $my-app-dark-accent,
      warn: $my-app-dark-warn,
    ),
  )
);

.dark-theme {
  @include mat.core-color($my-app-dark-theme);
  @include mat.button-color($my-app-dark-theme);
}
```

Notice that we are using a class selector `.dark-theme` to render a dark theme.


#### Avoiding duplicated theming styles

While creating `dark-theme`, instead of `core-theme` and `button-theme`, which we used in the original theme, we are using `core-color` and `button-color`. The reason behind that is we only want to change colors in `dark-theme` and every other style should remain the same. If we use `theme` mixins, it would generate all the styles again, which are not required.


#### Changes for background and font color

To complete the theme setup for background and font color, we will need to add class `mat-app-background` to the `<body>` tag in `index.html`:

```html
<body class="mat-typography mat-app-background">
  <app-root></app-root>
</body>
```


#### Lazy load dark theme

For our application, `dark-theme` is an additional theme and can be loaded based on user preferences. So, instead of making it part of the default application, we will lazy load it.

Let’s make changes for that in the project’s `angular.json`:

```json
"styles": [
              "src/styles.scss",
              {
                "input": "src/styles/themes/dark.scss",
                "bundleName": "dark-theme",
                "inject": false
              }
            ],
```

_You can learn more about lazy loading stylesheets at: [How to exclude stylesheets from the bundle and lazy load them in Angular?](https://indepth.dev/tutorials/angular/lazy-load-stylesheets)_

To load the `dark-theme` based on user’s selection, we will simply implement a service called `style-manager.service.ts` and whenever we want to change theme, we will simply call `toggleDarkTheme` from this service:

```typescript
// style-manager.service.ts

import { Injectable } from '@angular/core';

@Injectable({ providedIn: 'root' })
export class StyleManager {
  isDark = false;

  toggleDarkTheme() {
    if (this.isDark) {
      this.removeStyle('dark-theme');
      document.body.classList.remove('dark-theme');
      this.isDark = false;
    } else {
      const href = 'dark-theme.css';
      getLinkElementForKey('dark-theme').setAttribute('href', href);
      document.body.classList.add('dark-theme');
      this.isDark = true;
    }
  }

  removeStyle(key: string) {
    const existingLinkElement = getExistingLinkElementByKey(key);
    if (existingLinkElement) {
      document.head.removeChild(existingLinkElement);
    }
  }
}

function getLinkElementForKey(key: string) {
  return getExistingLinkElementByKey(key) || createLinkElementWithKey(key);
}

function getExistingLinkElementByKey(key: string) {
  return document.head.querySelector(
    `link[rel="stylesheet"].${getClassNameForKey(key)}`
  );
}

function createLinkElementWithKey(key: string) {
  const linkEl = document.createElement('link');
  linkEl.setAttribute('rel', 'stylesheet');
  linkEl.classList.add(getClassNameForKey(key));
  document.head.appendChild(linkEl);
  return linkEl;
}

function getClassNameForKey(key: string) {
  return `style-manager-${key}`;
}
```

_Above is a very opinionated approach, you can change it as per your need._


#### Output after creating a dark-theme

Now, let’s utilize the above service in `app.component.ts`:

```typescript
// src/app/app.component.ts

import { Component } from '@angular/core';
import { StyleManager } from './shared/services/style-manager.service';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.scss'],
})
export class AppComponent {
  title = 'my-app';
  isDark = this.styleManager.isDark;

  constructor(private styleManager: StyleManager) {}

  toggleDarkTheme() {
    this.styleManager.toggleDarkTheme();
    this.isDark = !this.isDark;
  }
}
```

Next, we will add a button to toggle dark and light themes in `app.component.html`:

```html
<!-- src/app/app.component.html -->

<div>
  <h1>Angular Material Theming System: Complete Guide</h1>
  <button
    mat-icon-button
    (click)="toggleDarkTheme()"
    class="theme-toggle"
    aria-label="Toggle Dark Theme"
  >
    <mat-icon>{{ isDark ? "dark_mode" : "light_mode" }}</mat-icon>
  </button>
  <div class="demo-buttons">
    <button mat-raised-button color="primary">Raised</button>
    <button mat-raised-button color="accent">Accent</button>
    <button mat-raised-button color="warn">Warn</button>
  </div>
</div>
```

Let’s look at the output now:

![output after adding dark theme](https://i.imgur.com/IWf1AUt.gif "output after adding dark theme")

Notice that when we change theme, it changes colors and background colors of buttons and text. And also notice that `dark-theme.css` is only included when the user switches to the dark theme.


### Apply Angular Material’s theme to custom component

Let’s assume that there is an `alert` component with the template below:

```html
<!-- alert.html →

<div class="alert" role="alert">
  <h4 class="alert-heading"><mat-icon>thumb_up</mat-icon> Well done!</h4>
  <button mat-icon-button aria-label="Close" class="alert-close">
    <mat-icon>close</mat-icon>
  </button>
  <p class="alert-message">
    You did a great job! To get your reward,
    <a href="#" class="alert-link">click here</a>.
  </p>
  <hr />
  <p class="alert-footer">
    Your reward will be credited to your account within 24 Hour.
  </p>
</div>
```

And it’s styling is as below:

```scss
// alert.scss

.alert {
  position: relative;
  padding: 1rem 1rem;
  margin-bottom: 1rem;
  border: 1px solid transparent;
  border-radius: 0.25rem;

  color: #084298;
  background-color: #cfe2ff;
  border-color: #b6d4fe;

  .alert-link {
    color: #06357a;
  }

  hr {
    margin: 1rem 0;
    color: inherit;
    background-color: currentColor;
    border: 0;
    opacity: 0.25;
    height: 1px;
  }

  .alert-heading {
    color: inherit;
    font-size: calc(1.275rem + 0.3vw);
  }

  .alert-link {
    font-weight: 700;
  }

  mat-icon {
    color: currentColor;
    vertical-align: middle;
    font-size: inherit;
    line-height: inherit;
    height: inherit;
    width: inherit;
  }

  .alert-close {
    position: absolute;
    top: 0;
    right: 0;
    z-index: 2;

    mat-icon {
      color: #000;
      font-size: 24px;
      line-height: 1;
      height: 24px;
      width: 24px;
    }
  }

  .alert-footer {
    font-size: 80%;
    margin-bottom: 0;
  }
}
```

With the above styling it looks like below:
                                  
![alert output](https://i.imgur.com/rBbqdN4.png "alert output")

Now, we want to apply Angular Material’s theme to the `alert` component, so that colors and typography match with the theme.


#### Separating theme styles

Angular Material components each have a SASS file that defines mixins for customizing that component's color and typography. For example, `MatButton` has mixins for `button-color` and `button-typography`. Each mixin emits all color and typography styles for that component, respectively.

We will mirror this structure in `alert` components by defining our own mixins. These mixins should accept an Angular Material theme, from which they can read color and typography values. We will then include these mixins in your application along with Angular Material's own mixins.

This is going to be a step-by-step process.


##### Step 1: Extract theme-based styles to a separate file

To change this file to participate in Angular Material's theming system, we split the styles into two files, with the color and typography styles moved into mixins. By convention, the new file name ends with `-theme`. Additionally, the file starts with an underscore (`_`), indicating that this is a SASS partial file. See the [SASS documentation](https://sass-lang.com/guide#topic-4) for more information about partial files.

```scss
// alert.scss

.alert {
  position: relative;
  padding: 1rem 1rem;
  margin-bottom: 1rem;
  border: 1px solid transparent;
  border-radius: 0.25rem;

  hr {
    margin: 1rem 0;
    color: inherit;
    background-color: currentColor;
    border: 0;
    opacity: 0.25;
    height: 1px;
  }

  .alert-heading {
    color: inherit;
  }

  .alert-link {
    font-weight: 700;
  }

  mat-icon {
    color: currentColor;
    vertical-align: middle;
    font-size: inherit;
    line-height: inherit;
    height: inherit;
    width: inherit;
  }

  .alert-close {
    position: absolute;
    top: 0;
    right: 0;
    z-index: 2;

    mat-icon {
      color: #000;
      font-size: 24px;
      line-height: 1;
      height: 24px;
      width: 24px;
    }
  }

  .alert-footer {
    margin-bottom: 0;
  }
}
```

We will move theme related stuff to `_alert-theme.scss`:

```scss
// _alert-theme.scss

@mixin color($theme) {
  .alert {
    color: #084298;
    background-color: #cfe2ff;
    border-color: #b6d4fe;

    .alert-link {
      color: #06357a;
    }
  }
}

@mixin typography($theme) {
  .alert {
    .alert-heading {
      font-size: calc(
        1.275rem + 0.3vw
      );
    }
    .alert-footer {
      font-size: 80%;
    }
  }
}
```


##### Step 2: Use values from the theme

Now that theme-based styles reside in mixins, we can extract the values we need from the theme passed into the mixins.


###### Reading color values

To read color values from a theme, you can use the `get-color-config` SASS function. This function returns a SASS map containing the theme's primary, accent, and warn palettes, as well as a flag indicating whether dark mode is set. Below is one example:

```scss
@use 'SASS:map';
@use '@angular/material' as mat;

$color-config:    mat.get-color-config($theme);
$primary-palette: map.get($color-config, 'primary');
$accent-palette:  map.get($color-config, 'accent');
$warn-palette:    map.get($color-config, 'warn');
$is-dark-theme:   map.get($color-config, 'is-dark');
```


###### Reading typography values

To read typography values from a theme, you can use the `get-typography-config` SASS function. For example:

```scss
@use '@angular/material' as mat;

$typography-config: mat.get-typography-config($theme);
$my-font-family: mat.font-family($typography-config);
```

Let’s make changes in `_alert-theme.scss`:

```scss
// _alert-theme.scss

@use "sass:map";
@use "@angular/material" as mat;

@mixin color($theme) {
  // Get the color config from the theme.
  $color-config: mat.get-color-config($theme);

  // Get the primary color palette from the color-config.
  $primary-palette: map.get($color-config, "primary");

  .alert {
    color: mat.get-color-from-palette($primary-palette, 700);
    background-color: mat.get-color-from-palette($primary-palette, 100);
    border-color: mat.get-color-from-palette($primary-palette, 300);

    .alert-link {
      color: mat.get-color-from-palette($primary-palette, 900);
    }
  }
}

@mixin typography($typography-config) {
  .alert {
    .alert-heading {
      @include mat.typography-level($typography-config, "title");
    }
    .alert-footer {
      @include mat.typography-level($typography-config, "caption");
    }
  }
}
```


##### Step 3: Add a theme mixin

For convenience, we will add a `theme` mixin that includes both color and typography.

```scss
// _alert-theme.scss

// rest remains same

@mixin theme($theme, $typography-config) {
  $color-config: mat.get-color-config($theme);
  @if $color-config != null {
    @include color($theme);
  }

  @if $typography-config != null {
    @include typography($typography-config);
  }
}
```


##### Step 4: Include the theme mixin in your application

First, we will create `_index.scss` at `src/styles/components` folder, which will hold mixins of all our components. And it will have a theme mixin.

```scss
// src/styles/components/_index.scss

@use "../../app/shared/components/alert/alert-theme" as alert;

@mixin theme($theme, $typography-config) {
  @include alert.theme($theme, $typography-config);
}
```

Next, we will include this `theme` mixin along with the other theme mixins in the application in `styles.scss`:

```scss
// src/styles.scss

@use "@angular/material" as mat;

@use "./styles/themes/light";
@use "./styles/components";
@use "./styles/typography/config" as typography;

@include mat.core(typography.$my-app-typography);

@include mat.core-theme(light.$my-app-light-theme);
@include mat.button-theme(light.$my-app-light-theme);
@include components.theme(
  light.$my-app-light-theme,
  typography.$my-app-typography
);

html,
body {
  height: 100%;
}
body {
  margin: 0;
  font-family: Roboto, "Helvetica Neue", sans-serif;
}
```

We will also include this theme in `src/styles/theme/dark.scss`:

```scss
// src/styles/themes/dark.scss

@use "sass:map";
@use "@angular/material" as mat;

@use "../typography/config" as typography;
@use "../components";

$my-app-dark-primary: mat.define-palette(mat.$blue-grey-palette);
$my-app-dark-accent: mat.define-palette(mat.$amber-palette, A200, A100, A400);
$my-app-dark-warn: mat.define-palette(mat.$deep-orange-palette);
$my-app-dark-theme: mat.define-dark-theme(
  (
    color: (
      primary: $my-app-dark-primary,
      accent: $my-app-dark-accent,
      warn: $my-app-dark-warn,
    ),
  )
);

.dark-theme {
  @include mat.core-color($my-app-dark-theme);
  @include mat.button-color($my-app-dark-theme);
  @include components.theme(
    $my-app-dark-theme,
    typography.$my-app-typography
  );
}
```

Let’s look at the alert now:

![output after applying angular material theme to alert](https://i.imgur.com/ozynt4s.gif "output after applying angular material theme to alert")

It works with light and dark themes. And also the heading font of the “Work Sans” family is applied to the alert's header.


### Customizing Angular Material Component Styles

In this last section, we will learn how to customize styles of Angular Material components. To customize any of the Angular Material component’s styles, we can simply create CSS classes and use the same. For this example, we will customize styling of [`MatButton`](https://material.angular.io/components/button/overview).

Angular Material provides 3 variants: `primary`, `accent` an `warn`. If you inspect and check the generated HTML code for those variants, e.g. `<button mat-button color=”primary”>Primary</button>`, you will find that it adds class `mat-primary` to the button:

![mat-primary class added to MatButton](https://i.imgur.com/KA4sdWP.png "mat-primary class added to MatButton")

For this example, we will add two variants for `MatButton`: `success` and `info`. So, based on the above finding, we will need to add support for the `mat-success` and `mat-info` classes.

Now, we may first think to simply add `color` and `background-color` for the classes `mat-success` and `mat-info`. But, if you check the DOM structure of `MatButton`, it is not just a simple button, it has multiple elements involved inside it to support the ripple effect:

![MatButton DOM structure](https://i.imgur.com/iIf5IZA.png "MatButton DOM structure")

So to properly style the new variants of button, we will need to follow a particular approach. Let’s get started.


#### Step 1: Create variants map for color palettes

Let’s first create `success` and `info` color palettes for `light` and `dark` themes. And we will use it with `components.theme` mixin.

We will create the `variant` [map](https://sass-lang.com/documentation/values/maps) like below and then use each palette from that to create variants of the button:

```scss
$variants: (
    success: $success-palette,
    info: $info-palette
)
```

Let’s first create a variant map for `light` theme in `src/styles/theme/_light.scss` at last:

```scss
// src/styles/themes/_light.scss

// rest remains same

$my-app-light-success: mat.define-palette(mat.$green-palette);
$my-app-light-info: mat.define-palette(mat.$blue-palette);

$my-app-light-variants: (
  success: $my-app-light-success,
  info: $my-app-light-info,
);
```

For `success`, we are using [`$green-palette`](https://github.com/angular/components/blob/97ec228ada31e55f76a7e0171c715bb0b8d07e9c/src/material/core/theming/_palette.scss#L337) and for `info` it’s [`$blue-palette`](https://github.com/angular/components/blob/97ec228ada31e55f76a7e0171c715bb0b8d07e9c/src/material/core/theming/_palette.scss#L205).

Next, we will pass this variant map in `components.theme` mixin as 3rd argument in `src/styles.scss`:

```scss
// src/styles.scss

// rest remains same

@include components.theme(
  light.$my-app-light-theme,
  typography.$my-app-typography,
  light.$my-app-light-variants
);

// rest remains same
```

Next, we will create the same variants map for the `dark` theme in `src/styles/theme/dark.scss` also pass it in `components.theme` mixin:

```scss
// src/styles/themes/dark.scss

// rest remains same

$my-app-dark-success: mat.define-palette(mat.$light-green-palette);
$my-app-dark-info: mat.define-palette(mat.$light-blue-palette);

$my-app-dark-variants: (
  success: $my-app-dark-success,
  info: $my-app-dark-info,
);

.dark-theme {
  @include mat.core-color($my-app-dark-theme);
  @include mat.button-color($my-app-dark-theme);
  @include components.theme(
    $my-app-dark-theme,
    typography.$my-app-typography,
    $my-app-dark-variants
  );
}
```

For the `dark` theme, we are using [`$light-green-palette`](https://github.com/angular/components/blob/97ec228ada31e55f76a7e0171c715bb0b8d07e9c/src/material/core/theming/_palette.scss#L370) and [`$light-blue-palette`](https://github.com/angular/components/blob/97ec228ada31e55f76a7e0171c715bb0b8d07e9c/src/material/core/theming/_palette.scss#L238) for `success` and `info` variants.


#### Step 2: Add variants map in components’ theme mixin

In `components.theme` mixin in `src/styles/components/_index.scss`, there are two arguments as of now: `$theme` and `$typography-config`. We will now add the third argument for variants map:

```scss
// src/styles/components/_index.scss

@use "../../app/shared/components/alert/alert-theme" as alert;

@mixin theme($theme, $typography-config, $variants) {
  @include alert.theme($theme, $typography-config);
}
```

We will see in further steps how to use the `$variants` map.


#### Step 3: Create variants for button

To create `success` and `info` variants for buttons, we are going to follow the same approach from [`src/material/button/_button-theme.scss`](https://github.com/angular/components/blob/master/src/material/button/_button-theme.scss).

If you look at the [button’s theme’s source code](https://github.com/angular/components/blob/master/src/material/button/_button-theme.scss), 5 mixins are needed for button’s styles to work properly:

1. `_focus-overlay-color` - Will apply a focus style to an mat-button element for each of the supported palettes
2. `_ripple-background` - Will apply the background color for a ripple.
3. `_ripple-color` - Will include `_ripple-background` for each variant
4. `_theme-property` - Will apply a property to an `mat-button` element for each of the supported palettes from variants
5. `color` - Will apply theming using all of the above 4 mixins for each [type of `mat-button`](https://material.angular.io/components/button/overview). They are basic, raised, stroked, flat, icon and fab.

Let’s create a file `_variants.scss` at `src/styles/components/button` with the following content:

```scss
// src/styles/components/button/_variants.scss

@use "sass:map";
@use "sass:meta";
@use "@angular/material" as mat;

$_ripple-opacity: 0.1;

// Applies a focus style to an mat-button element for each of the supported palettes.
@mixin _focus-overlay-color($config-or-theme, $variants) {
  $config: mat.get-color-config($config-or-theme);

  @each $variant, $variant-palette in $variants {
    &.mat-#{$variant} .mat-button-focus-overlay {
      background-color: mat.get-color-from-palette($variant-palette);
    }
  }
}

@mixin _ripple-background($palette, $hue, $opacity) {
  $background-color: mat.get-color-from-palette($palette, $hue, $opacity);
  background-color: $background-color;
  @if (meta.type-of($background-color) != color) {
    opacity: $opacity;
  }
}

@mixin _ripple-color($theme, $hue, $opacity, $variants) {
  @each $variant, $variant-palette in $variants {
    &.mat-#{$variant} .mat-ripple-element {
      @include _ripple-background($variant-palette, $hue, $opacity);
    }
  }
}

// Applies a property to an mat-button element for each of the supported palettes.
@mixin _theme-property($theme, $property, $hue, $variants) {
  $background: map.get($theme, background);
  $foreground: map.get($theme, foreground);

  @each $variant, $variant-palette in $variants {
    &.mat-#{$variant} {
      #{$property}: mat.get-color-from-palette($variant-palette, $hue);
    }

    &.mat-#{$variant} {
      &.mat-button-disabled {
        $palette: if($property == "color", $foreground, $background);
        #{$property}: mat.get-color-from-palette($palette, disabled-button);
      }
    }
  }
}

@mixin color($config-or-theme, $variants) {
  $config: mat.get-color-config($config-or-theme);
  $foreground: map.get($config, foreground);
  $background: map.get($config, background);

  .mat-button,
  .mat-icon-button,
  .mat-stroked-button {
    @include _theme-property($config, "color", text, $variants);
    @include _focus-overlay-color($config, $variants);
  }

  .mat-flat-button,
  .mat-raised-button,
  .mat-fab,
  .mat-mini-fab {
    @include _theme-property($config, "color", default-contrast, $variants);
    @include _theme-property($config, "background-color", default, $variants);
    @include _ripple-color(
      $config,
      default-contrast,
      $_ripple-opacity,
      $variants
    );
  }
}
```


#### Step 4: Include button color mixin in components’ theme

It’s time to include `button.color` mixin in `components.theme` at `src/styles/components/_index.scss`:

```scss
// src/styles/components/_index.scss

@use "../../app/shared/components/alert/alert-theme" as alert;
@use "./button/variants" as button-variants;

@mixin theme($theme, $typography-config, $variants) {
  @include alert.theme($theme, $typography-config);
  @include button-variants.color($theme, $variants);
}
```

That’s it! Now you can add `MatButton` with `color=”success”` and `color=”info”`. Let’s add them in `app.component.html` at the end:

```html
<!-- src/app/app.component.html -->

<!-- rest remains same ->

<!-- success buttons -->

<button mat-button color="success">Success</button>
<button mat-stroked-button color="success">Success Stroked</button>
<button mat-flat-button color="success">Success Flat</button>
<button mat-raised-button color="success">Success Raised</button>
<button mat-icon-button color="success">
  <mat-icon>check_circle</mat-icon>
</button>
<button mat-fab color="success">
  <mat-icon>check_circle</mat-icon>
</button>
<button mat-mini-fab color="success">
  <mat-icon>check_circle</mat-icon>
</button>

<!-- info buttons -->

<button mat-button color="info">Info</button>
<button mat-stroked-button color="info">Info Stroked</button>
<button mat-flat-button color="info">Info Flat</button>
<button mat-raised-button color="info">Info Raised</button>
<button mat-icon-button color="info">
  <mat-icon>info</mat-icon>
</button>
<button mat-fab color="info">
  <mat-icon>info</mat-icon>
</button>
<button mat-mini-fab color="info">
  <mat-icon>info</mat-icon>
</button>
```

Let’s take a look at output now:

![output after adding variants for MatButton](https://i.imgur.com/wyLKtuW.gif "output after adding variants for MatButton")


## Updating old project to latest version of Angular Material

If your current project uses Angular Material older than version 12 and wants to update to version 13, then follow this section, else you can jump to the [summary](#summary).

For this example, we are going to take code from my series of “[Custom Theme for Angular Material Components Series](https://indepth.dev/posts/1320/custom-theme-for-angular-material-components-series-part-1-create-a-theme)”. The code is available at [indepth-theming-material-components](https://github.com/shhdharmen/indepth-theming-material-components).

If you run `ng version` inside the project's folder, you will notice that version `10.1` is used. And we want to upgrade it to version `13`.


### Angular Update Guide

We are going to follow guidelines from the [Angular Update Guide](https://update.angular.io/). Angular CLI does not support migrating across multiple major versions at once. So we will migrate each major version individually.

Open up the terminal in the project's folder and run the commands below. After each command you will have to commit your changes, otherwise Angular CLI will not allow you to proceed further.

While running any of the commands below, if you face any error like `Could not resolve dependency` or `Conflicting peer dependency`, do the following:

1. Revert the changes of `package.json`
2. Install dependencies again with `npm i`
3. Run the update command with `--force`


#### Version 10 to 11


##### Update Angular to version 11

```bash
npx @angular/cli@11 update @angular/core@11 @angular/cli@11
```


##### Update Angular Material to version 11

```bash
npx @angular/cli@11 update @angular/material@11
```

With this, we have updated the project to version 11. Check once by running `npm start`. Now, we will upgrade the project to version 12.


#### Version 11 to 12


##### Update Angular to version 12


```bash
npx @angular/cli@12 update @angular/core@12 @angular/cli@12
```


##### Update Angular Material to version 12

```bash
npx @angular/cli@12 update @angular/material@12
```


##### Changes of version 12

With the above command, you will see many changes, let’s understand what’s changed.


###### Migration from `@import` to `@use`

The first major change you will notice is migration from `@import` to `@use`. So in all `.scss` files, below `@import`


```scss
@import "~@angular/material/theming";
```

is changed to below `@use`:

```scss
@use "~@angular/material" as mat;
```

The `@use` rule loads mixins, functions, and variables from other SASS stylesheets, and combines CSS from multiple stylesheets together. Stylesheets loaded by `@use` are called "modules".

The SASS team discourages the continued use of the `@import` rule. SASS will [gradually phase it out](https://github.com/sass/sass/blob/master/accepted/module-system.md#timeline) over the next few years, and eventually remove it from the language entirely


###### API refactors

To adhere to the above-mentioned module system, many APIs are also reworked. And they have been refactored for better developer experience. For example, `mat-get-color-config` is changed to `mat.get-color-config`. `mat-color` is changed to `mat.get-color-from-palette`.


##### Fix errors after update

Now if you try to run the project, it will throw errors. Let’s resolve those errors one by one.


###### Value isn’t a valid CSS value

The first error you will see is at line 7 of `sidenav.component.scss-theme.scss`:


```bash
7 │   $config: mat-get-color-config($config-or-theme);
  │                                 ^^^^^^^^^^^^^^^^
```

To fix that, we will change `mat-get-color-config` to `mat.get-color-config`. And make the same change in `dialog.component.scss-theme.scss`:

```scss
$config: mat.get-color-config($config-or-theme);
```


###### Undefined mixin

The next error you will see is at line 28:

```bash
28 │         @include _mat-toolbar-color($val);
   │         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
```

Above error is coming because within Angular Material version 12, components’ color mixins are refactored. And we can’t simply use the mixin any more. So, instead of using `MatToolbar`’s color mixin, we will use it’s SASS code. So change above line with below 2 lines in both, `sidenav.component.scss-theme.scss` and `dialog.component.scss-theme.scss` files:

```scss
background-color: mat.get-color-from-palette($val);
color: mat.get-color-from-palette($val, default-contrast);
```

Now your project should run fine.
                                                                                                      

##### Adhere to latest SASS changes

As per the latest SASS changes, [`map`](https://sass-lang.com/documentation/modules/map) module functions should be used in the new module system. For that, first we will use the `SASS:map` module using the `@use` rule:

```scss
@use "sass:map";
```

And then, simply change all `map-get` to `map.get` in both, `sidenav.component.scss-theme.scss` and `dialog.component.scss-theme.scss` files:

```scss
$primary: map.get($config, primary);
$accent: map.get($config, accent);
$warn: map.get($config, warn);
$foreground: map.get($config, foreground);
$background: map.get($config, background);
```


#### Version 12 to 13


##### Update Angular to version 13

```bash
npx @angular/cli@13 update @angular/core@13 @angular/cli@13
```


##### Update Angular Material to version 12

```bash
npx @angular/cli@13 update @angular/material@13
```


###### Removal of tilde

After the above command, except dependencies, one major change you will notice in all `.scss` files is the removal of `~` (tilde) from `@use "~@angular/material" as mat;`.

The reason behind that is [SASS-loader](https://webpack.js.org/loaders/sass-loader) has deprecated the usage of `~` and it’s recommended that it’s removed from code.

**Why remove it?**

The loader will first try to resolve `@use` as a relative path. If it cannot be resolved, then the loader will try to resolve `@use` inside `node_modules`.


## Summary

In this article, we first learned about what Angular Material Theming is and it is based on [Google's Material Design](https://material.io/design/material-theming/overview.html) specification. And then we understood that with Angular Material version 12, `@import` rule migrated to `@use` and SASS APIs were refactored for better developer experience.

We started with a blank project and added Angular Material. Next, we understood the `core` mixin, `define-palette` function, palettes and `define-light-theme` function and we created a custom theme. And then we applied our custom theme to first all components using `all-components-theme` and at last we optimized it to use only `core-theme` and `button-theme` and reduced final styles size.

We also learned how to use a pre-built theme by adding the theme's stylesheet path in the `styles` array of `angular.json`. For example, we can add `./node_modules/@angular/material/prebuilt-themes/indigo-pink.css` to use the `indigo-pink` theme in our application.

Then we started with typography. We first understood typography levels and how to create one using `define-typography-level`. Next, we learned that Angular Material handles all those levels using typography config, and Angular Material represents this configuration as a SASS map. We created a custom config using `define-typography-config` and applied it to `core` mixin so that custom typography is applied to the whole application.

Next we created a dark theme in a separate file `themes/dark-theme.scss`. Then we used only color mixins, i.e. `core-color` and `button-color`, and not theme mixing to avoid duplicate style generation. And at last, we made changes in `angular.json` so that dark theme is loaded on demand only when needed.

Then we followed a step-by-step process to add support for Angular Material’s Theming system to custom components.

And at last, we learned about how to customise Angular Material’s button component, i.e. [`MatButton`](https://material.angular.io/components/button/overview). In this, we mainly followed the approach from its source code and we added two new variants to it: `success` and `info`.

The project which we created in this article is available on GitHub repo at [angular-material-theming-system-complete-guide](https://github.com/shhdharmen/angular-material-theming-system-complete-guide).

With the new system, we also looked at how to update older Angular Material versions to the latest one by taking examples from one of the old projects.


## Credits

While writing this article I took references from [Angular Material Guides](https://material.angular.io/guides).
