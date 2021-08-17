---
title: How to exclude stylesheets from the bundle and lazy load them in Angular? - Angular Tutorials | indepth.dev
author_name: Dharmen Shah
author_link: https://twitter.com/shhdharmen
discussion_link: https://github.com/indepth-dev/community/discussions/148
---

# How to exclude stylesheets from the bundle and lazy load them in Angular?

Let’s learn how we can load stylesheets only when needed without making them part of the application bundle.

We will try to understand this by taking example of multiple themes support.

## Multiple theme files

Let’s assume that apart from main _styles.css_ file, you have 2 theme files present:

1. src/styles/themes/theme-light.css
2. src/styles/themes/theme-dark.css

Next, you would have them in `angular.json`’s styles option:


```json
"styles": [
    "src/styles.css",
    "src/styles/themes/theme-light.css",
    "src/styles/themes/theme-dark.css"
  ]
```

And lastly, you would handle loading of a particular theme based on the users’ choice or their preferences.

Everything works great, but both of your theme files are part of your application bundle all the time.

## Exclude theme files

Let’s make a minor change in angular.json to exclude theme files from bundle:

```json
"styles": [
    "src/styles.css",
    {
      "input": "src/styles/themes/theme-light.css",
      "inject": false,
      "bundleName": "theme-light"
    },
    {
      "input": "src/styles/themes/theme-dark.css",
      "inject": false,
      "bundleName": "theme-dark"
    }
  ]
```

Two new options to learn here:

1. **`inject`**: Setting this false will not include the file from “input” path in bundle
2. **`bundleName`**: A separate bundle will be created containing the stylesheet from “input” path

Now if you try to build the project, it will create separate files and the output will look something like below:

![output of npm run build command](https://user-images.githubusercontent.com/6831283/128328453-e6b500d3-a509-4899-8560-3babe6b881b1.png)

Notice that `theme-light.css` and `theme-dark.css` are part of **Lazy Chunk Files**. Lazy chunk files speed up the application load time, because they are loaded on demand.

## Lazy load theme files

We managed to exclude them from the bundle and they are externally available. Now the question arises how to load these theme files?

One way to load them is to use their bundle directly with a link tag:

```html
<link
  rel="stylesheet"
  href="theme-dark.css"
  media="(prefers-color-scheme: dark)" />
<link
  rel="stylesheet"
  href="them-light.css"
  media="(prefers-color-scheme: light)"
/>
```

You will maybe need to tweak the document base URL using `base` tag to successfully load them.

## Conclusion

We learnt that we can exclude stylesheets by simply setting `inject` false in workspace configuration file, i.e. `angular.json`. And to load them on demand, we will use the `bundleName` option.

The advantage of excluding stylesheets from the bundle is reduced bundle size, which will in turn improve the initial load time of application and finally user experience will be better.
