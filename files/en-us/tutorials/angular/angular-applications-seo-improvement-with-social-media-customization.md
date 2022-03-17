---
title: Angular applications SEO improvement with social media customization - Angular Tutorials | indepth.dev
author_name: Maria Zayed
author_link: https://twitter.com/mariawzayed
discussion_link:  https://github.com/indepth-dev/community/discussions/259
display_name: Angular applications SEO improvement with social media customization
---

# Introduction
[Scully](https://scully.io/) is a static site generator for Angular projects which utilizes all of Angular's power but still gives all the SEO benefits that static sites offer.

In this article, we will understand how the Google search algorithm works, then how we can improve the [SEO](https://yoast.com/what-is-seo/) in Angular apps using Scully, then how we can customize Angular apps for social media, then an optional section on how we could deploy a static site in Firebase.

## Understanding Google Search
Google follows three basic steps to generate results from web pages:

### Crawling
Once Google discovers a page URL, it visits or crawls, the page to find out what's on it (using bots). Google renders the page and analyzes both the text and non-text content and overall visual layout to decide where it should appear in search results. The better that Google can understand your site, the better they can match it to people who are looking for your content.

### Indexing
After a page is discovered, Google tries to understand what the page is about. This process is called indexing. Google analyzes the content of the page, catalogs images and video files embedded on the page, and otherwise tries to understand the page. This information is stored in the Google index, a huge database stored in many computers.

### Serving (and Ranking)
When a user types a query, Google tries to find the most relevant answer from its index based on many factors. Google tries to determine the highest quality answers, and factor in other considerations that will provide the best user experience and most appropriate answer, by considering things such as the user's location, language, and device (desktop or phone).

For example, searching for "bicycle repair shops" would show different answers to a user in Paris than it would to a user in Hong Kong. Google doesn't accept payment to rank pages higher, and ranking is done programmatically.

# Project Setup
To learn how to pre-render content, we will create a new Angular project. You can clone the starter code from [here](https://github.com/mariazayed/ng-boost-seo/tree/start-code), or you can copy/paste the following code.

The final code is also available [here](https://github.com/mariazayed/ng-boost-seo/tree/master).

In summary, we will create a simple project with 3 routes 
* `/`
* `/about`
* `/contact`

First, install the Angular CLI globally (if you don’t have it) with the following command
`
npm install -g @angular/cli
`

Then, create a new project named `ng-boost-seo` using this command:

`
ng new ng-boost-seo
`

Then, create the needed components using the following commands:

```
ng generate component components/home
ng generate component components/about
ng generate component components/contact
```

Then, create the main navigation for changing the routes.

```html
<!-- app.component.html -->

<nav>
  <ul>
     <li>
        <a routerLink="/">Home</a>
     </li>
     <li>
        <a routerLink="/about">About</a>
     </li>
     <li>
        <a routerLink="/contact">Contact</a>
     </li>
  </ul>
</nav>

<router-outlet></router-outlet>
```

Then add the HTML contents for the `home`, `contact`, and `about` components respectively.

```html
<!-- components/home.components.html -->

<h1>I'm the home component</h1>

<ul>
  <li>Learn how Google ranks your site</li>
  <li>Learn how to pre-render html content with Scully</li>
  <li>Learn how to add Search Engine meta tags</li>
  <li>Learn how to add Open Graph meta tags for social media</li>
  <li>Learn how to deploy your boosted app to Firebase</li>
</ul>
```

```html
<!-- components/contact.component.html -->

<h1>I'm the contact component</h1>

<p>
  Keep in touch with us
</p>
```

```html
<!-- components/about.component.html -->

<h1>I'm the about component</h1>

<p>
  Make sure your site ranks high on Google so you get more visits and your business grows.
</p>
```

Then, configure the routes in `app.routing.module.ts` file like this:

```ts
// app-routing.module.ts

const routes: Routes = [
  {
     path: '',
     component: HomeComponent,
     pathMatch: 'full',
  },
  {
     path: 'about',
     component: AboutComponent,
  },
  {
     path: 'contact',
     component: ContactComponent,
  },
  {
     path: '**',
     redirectTo: '',
  },
];

@NgModule({
  imports: [ RouterModule.forRoot(routes) ],
  exports: [ RouterModule ],
})

export class AppRoutingModule {
}
```

# Improving SEO in Angular apps
Now that we know the three main steps Google uses to position search results, let's see what changes can be done to Angular apps to rank higher.

First, we run the starter code and use browser’s `View page resource` option.

![4](https://user-images.githubusercontent.com/27064594/156120138-efec0936-16c4-48a0-a6dc-b7ff835561cc.png)

We should get the following result

![5](https://user-images.githubusercontent.com/27064594/156120191-47cd6e42-a0d3-4858-9d58-4f1f374a2031.PNG)

Not much content for Google when visiting our page, right? We only have `<app-root></app-root>` with any content inside, and some scripts at the bottom.

Angular apps are SPAs (Single Page Applications) and the content inside `<app-root></app-root>` is rendered at runtime. In other words, the browser runs the Angular bundled JavaScript files and then, renders the HTML content.

Relying on Google bots executing our JavaScript to finally see the content is not the best way. So, the objective is to pre-render each route’s HTML content, so bots can instantly analyze it, without the need of executing any JavaScript.

## Scully to the rescue
This task of pre-rendering the HTML content can be done using [Scully](https://scully.io/), which is a static site generator that analyzes the route structure of the compiled Angular application and generates a static version of each page.

Before making any change, check the software requirements
* Angular versions: v8.x.x or higher
* Node.js: 10 or higher.
* Chromium: Scully uses Chromium. Therefore, your Operating System, as well as its administrator rights must allow its installation and execution.

Make sure you meet all those requirements when building the app on your machine. Our demo app already has an `app-routing.module.ts` file, which is a pre-requisite for Scully. If you don’t have it in your project, you can create one with the following Angular CLI command:

`
ng generate module app-routing --flat --module=app
`

Now, it’s time to add Scully (you can see the final code for this section from [here](https://github.com/mariazayed/ng-boost-seo/tree/add-scully))

`
ng add @scullyio/init
`

And then choose `Scully platform server` option

![2](https://user-images.githubusercontent.com/27064594/156120916-c10ae1de-fbdf-4b35-9793-c5053635534b.PNG)

After installation. the following files will be updated/created.

![3](https://user-images.githubusercontent.com/27064594/156121072-b9f80b6d-47ae-40f1-b4e0-1ffce347a5c0.PNG)

**NOTE:** After installation, if you were serving the app during the installation, you need to restart the server with `ng serve`

After adding Scully, we have a config file named `scully.<projectName>.config.ts`, where the `projectName` is the name of our Angular project. For our demo app, config file looks like this:

```ts
// scully.ng-boost-seo.config.ts

import { ScullyConfig } from '@scullyio/scully';

export const config: ScullyConfig = {
 projectRoot: "./src",
 projectName: "ng-boost-seo",
 outDir: './dist/static',
 routes: {}
};
```

Even with this basic config, we are now ready to build our Angular app using Scully for the first time!

**NOTE:** It is important to know that any routes in the Angular project that contain route parameters will not be pre-rendered until we modify the above config to account for those parameters.

Before Scully can run, we need to build our Angular project. Let’s make sure we output the build files into a folder inside the `/dist` folder. For this demo app, the `angular.json` file has this default config:

```ts
// angular.json

...
"build": {
 "builder": "@angular-devkit/build-angular:browser",
 "options": {
   "outputPath": "dist/ng-boost-seo",
...
```

So, the build result of the Angular application will be added to a folder named `ng-boost-seo` inside the `dist` folder.

Let’s now build our Angular app running this command:

`
ng build
`

Now that the Angular project was built, Scully can do its work. We now run Scully, with the following command:

`
npm run scully
`

**We did it!** We have turned our Angular app into a pre-rendered static site.

The Scully-built version of the project is located in the `/dist/static` folder. It contains all the static pages in the project. In our app, it has three `index.html` files, one for each route:

Each `index.html` file contains the pre-rendered HTML that corresponds to each route.

![6](https://user-images.githubusercontent.com/27064594/156121348-3ae3fdd1-3253-4063-968d-a0e4f6c435ff.PNG)

**NOTE:** If the application has 100 routes, there should be 100 `index.html` files in the `dist/static` folder.

The name of the folders inside `/dist/static` has the name of the routes. So if you have a route called `/news` in your app, there will be a folder named `/news`, which holds an `index.html` file.

These `index.html` files are jamstack-packed with HTML and CSS. This result tells us that the Scully was built successfully and that our app is now pre-rendered.

The build is now ready. Scully provides us with a server so that we can test out our [jamstack](https://jamstack.org/what-is-jamstack/) site. To launch Scully's test server, let’s run the following command:

`
npm run scully:serve
`

This is the prompted result:

![7](https://user-images.githubusercontent.com/27064594/156121578-85de06b4-e7c3-4842-9e09-f5c4f466c8b6.PNG)

This command actually launches two servers. The first one hosts the results of the Scully build (serving the files inside the `/static` folder), and the second server hosts the results of `ng build` (serving the files inside `/ng-boost-seo`). This allows us to test both versions of our built app.

We now access `http://localhost:1668/`, where the `/static` folder is served. We can actually test again the source of the application using the `view page source` and we’ll see the following content:

![8](https://user-images.githubusercontent.com/27064594/156121677-6267448e-e1bc-447e-9c0c-0f5826bb41d0.PNG)

Now we see that there is some content inside the `<app-root></app-root>` tag. That’s what we needed!

What happens if we now go to `/about` using the navigation link, and inspect sources again with `view source code`? The `about` page looks as follows:

![9](https://user-images.githubusercontent.com/27064594/156121767-d574cf56-e87e-4c5f-bcad-f39db489dc23.PNG)

So, every route has its content pre-rendered. This will improve the page SEO because the content is now visible to bots.

## Improving the page with HTML tags
HTML tags lets bots properly understand what’s our web page content about and index it properly. 

Now it’s time to make some changes to some files in our Angular app (you can see the final code for this section from [here](https://github.com/mariazayed/ng-boost-seo/tree/html-tags-improvement))

### Title & description
The most important tag for SEO is the title. It is kind of the label of our content. The other one is the description meta tag, which is useful for users to decide if they want to visit the page. 

Let’s add the following code to the `index.html` file:

```html
<!-- index.html -->

<!doctype html>
<html lang="en">
     ...
     <title>How to Boost Angular Apps SEO</title>
     <meta content="A guide to boost your Angular app SEO and still have all the benefits of SPAs"
           name="description"/>
     ...
</html>
```

Let’s see an example of how these tags affect a site’s look on Google search. Let’s search for inDepthDev on Google:

![10](https://user-images.githubusercontent.com/27064594/156122012-30b103f2-a00f-4b89-b245-43fd8c3efdff.PNG)

The title is marked in brown, and the description is in blue. This information is pretty useful for users, right?

### Use meaningful headers

Let's change the `<h1>` tag content in the `home.component.html`.

```html
<!-- components/home.components.html -->

<!-- Change this -->
<h1>I'm the home component</h1>

<!-- To this -->
<h1>How to boost your Angular App SEO</h1>
```

This change will help Google determine segments of content and create featured rich snippets.

### Add Microdata
Microdata, is a set of tags, introduced with HTML5. [Schema.org](https://schema.org/) provides a collection of shared vocabularies. Webmasters can use them to mark up their pages in ways that can be understood by the major search engines such as Google, Microsoft, Yandex, and Yahoo.

Let’s change our `index.html` file and add the following meta tags:

```html
<!-- index.html -->

...
     <meta content="How to boost Angular Apps SEO"
           itemprop="name"/>
     <meta content="A guide to boost your Angular app SEO and still have all the benefits of SPAs"
           itemprop="description"/>
     <meta content="IMAGE URL"
           itemprop="image"/>
...
```

In the above code snippet, we added three `<meta />` tags, each one has a `content` and `itemprop`.
`iItemprop` is the property that we add to provide the search engine with more information on what our website is about. Schema.org provides shared vocabularies that webmasters can use. For `itemprop` we mainly have four properties: `name`, `description`, `url`, and `image`.

**Note:** the URL for the image is up to you because it will depend on where you host it. You can leave this URL blank and fill it in later on. Check [this](https://schema.org/docs/gs.html#microdata_why) for more information.

### Facebook and Whatsapp customization

Let’s customize how our site will be shared on Facebook and WhatsApp using [Open Graph](https://ogp.me/) meta tags which are snippets of code that control how URLs are displayed when shared on social media. In the case of Facebook and WhatsApp, the tags in the `index.html` file should be like this:

```html
<!-- index.html -->

...
<meta content="en_US"
     property="og:locale"/>
<meta content="WEBSITE WRL"
     property="og:url"/>
<meta content="website"
     property="og:type"/>
<meta content="How to Boost Angular Apps SEO"
     property="og:title"/>
<meta content="A guide to boost your Angular app SEO and still have all the benefits of SPAs"
     property="og:description"/>
<meta content="IMAGE URL"
     property="og:image"/>
<meta content="How to Boost Angular Apps SEO"
     property="og:site_name"/>
...
```

For the above code snippet, there are four required properties for every page
- `og:title` - The title of our object as it should appear within the graph.
- `og:type` - The type of our object.
- `og:image` - An image URL which should represent our object within the graph.
- `og:url` - URL where our app will be served.

### Twitter customization

Now let's customize how our site will be shared on Twitter and let’s add these tags to `index.html’

```html
<!-- index.html -->

...
<meta content="summary"
     name="twitter:card"/>
<meta content="How to boost Angular Apps SEO"
     name="twitter:title"/>
<meta content="A guide to boost your Angular app SEO and still have all the benefits of SPAs"
     name="twitter:description"/>
<meta content="WEBSITE WRL"
     name="twitter:url"/>
<meta content="IMAGE URL"
     name="twitter:image"/>
...
```

For more information on how to use Twitter tags, check this [doc](https://developer.twitter.com/en/docs/twitter-for-websites/cards/overview/markup).

Now let’s build our application again to deploy it, using these commands:

`
ng build
npm run scully
`

# Deploy to Firebase (optional)

Now, our app is ready to go live. First of all, let’s create a project on Firebase from [here](https://firebase.google.com/), and then click on `Go to console`, on the top right corner:

![11](https://user-images.githubusercontent.com/27064594/156124177-c2f9cc3e-c15a-49ca-aaa0-6d5dc9ae83b8.PNG)

Now, click on `add project`

![12](https://user-images.githubusercontent.com/27064594/156124213-c19841ce-df11-4e6f-9c88-73607bee1b0e.PNG)

And then, it’s just a matter of following the steps they provide to add a project, which is super straightforward.

Now, that the project has been successfully created, let’s go back to our code editor and open the terminal at the root level of our app folder.

To host our site with Firebase Hosting, we need the Firebase CLI. Run the following command to install the CLI or update to the latest CLI version.

`
npm install -g firebase-tools
`

Then we need to sign in to Google by running:

`
firebase login
`

Then run the following command to initiate our project:

`
firebase init
`

Then choose the hosting option

![13](https://user-images.githubusercontent.com/27064594/156124266-3621ecb4-76f9-47d3-86fe-d4bad5ee8d0e.PNG)

Then let’s link our Angular app to a Firebase project, by choosing to use an existing project and pick up the Firebase project you’ve created at the beginning

![14](https://user-images.githubusercontent.com/27064594/156124312-7bb7d4d6-6734-469d-a2e5-5d9cf6406415.PNG)

Then we’ll be asked to set the `public` folder, which is the folder that has our deployed files.

![15](https://user-images.githubusercontent.com/27064594/156124349-aca4d888-bf1c-422d-bf20-1a7c73d81372.PNG)

Type `dist/static` to set it as a public directory

![16](https://user-images.githubusercontent.com/27064594/156124422-54b48854-4fb9-483e-917c-067f152c5f6c.PNG)

Now, we'll be asked how we want to configure the Firebase server when incoming requests come.

![17](https://user-images.githubusercontent.com/27064594/156124466-4d3e2302-d590-44ef-bfab-d85c8b814005.PNG)

Choose `N` for No, because we don't want the server to always serve the same `index.html` for all the routes requested.

Now, Firebase will try to rewrite the `404.html` file located at the root level of our `/dist/static` folder.

![18](https://user-images.githubusercontent.com/27064594/156124530-f1dc1942-8bd6-4a3f-82b7-6393a85a81db.PNG)

Let's answer `N` for No.

They will also try to rewrite our `index.html` file located at the root level of our `/dist/static` folder.

![19](https://user-images.githubusercontent.com/27064594/156124582-642a76f3-410c-472e-9dd5-d70341f42d1c.PNG)

Let's answer `N` for No.

Now, our firebase project is configured.

![20](https://user-images.githubusercontent.com/27064594/156124618-90a7ec79-031b-4d34-8f25-3c9e9b3d1b29.PNG)


It’s time to deploy our project by running this command:

`
firebase deploy
`

![21](https://user-images.githubusercontent.com/27064594/156124656-e156bae3-2688-4936-9261-035a2a973bfc.PNG)

**We made it!** Now, the deployed Angular app will have the amazing user experience that a SPA can give, and it will also have good SEO.

# Further Reading
* [How Google Search Works (for beginners)](https://developers.google.com/search/docs/beginner/how-search-works)
* [Schema Markup 2022- SEO Best Practices](https://makewebbetter.com/blog/schema-markup-seo-best-practices/)
* [Understand how structured data works](https://developers.google.com/search/docs/advanced/structured-data/intro-structured-data)
* [Open Graph Meta Tags: Everything You Need to Know](https://ahrefs.com/blog/open-graph-meta-tags/)
* [A Guide to Share for Webmasters (for Facebook & Whatsapp)](https://developers.facebook.com/docs/sharing/webmasters/)

# Conclusion

SEO can help you improve your rankings in search engine results. This has the potential to make a huge impact on every company’s most important goals, like increasing the leads and sales.

In this article, we understood how to improve the SEO in Angular apps using Scully, and how to customize the site that will be shared on Facebook, Whatsapp, and Twitter. As well as, how to deploy static sites to Firebase.
