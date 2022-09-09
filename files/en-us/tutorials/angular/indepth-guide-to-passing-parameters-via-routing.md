---
title: InDepth guide to passing parameters via routing - Angular Tutorials | indepth.dev
author_name: Maciej Wójcik
author_link: https://twitter.com/maciej_wwojcik
discussion_link: https://github.com/indepth-dev/community/discussions/285
display_name: InDepth guide to passing data via routing
---

# **InDepth guide to passing parameters via routing**

## What you will learn

- What are the different ways to pass data via routing - route params, query params, matrix params
- How to send and receive route and query params
- What is the RouterLink directive and how to use it

## Intro

The case when we need to pass simple data through components is quite frequent in web development. Persisting this data - so the application presents the same view after the browser refresh, or after sending a link to the application to another browser is really important to us.

We can’t store such data in the Angular Service, because once the app is reloaded we will lose all the data, same goes when we open the link in another browser. We could use the browser's local storage for the first case, but still, it won’t work for the case when we open the link in another browser. The only thing that we can really use for such cases is storing the necessary data in the URL.

There are many different ways of storing information in the URL, I’m sure we all have seen URLs as follows:

- `www.app.com/home/dashboard?user=”user-id”`
- `www.app.com/home/dashboard/user-id/tasks`
- `www.app.com/home/dashboard/user-id;profile=premium/tasks`
- `www.app.com/home/dashboard/user-id/tasks?profile=premium`

There are a lot of various methods, use cases, and ways to persist the data. In this guide we will go through the most popular features available in Angular - route required params and query params. In the In-depth bits section of the guide, we will also learn about optional params aka matrix params, and how the `routerLink` directive works.

Keep in mind that this way of passing the data through routing is recommended for simple information, like IDs, flags, and a short indication of user choices. For passing complex objects, we most likely should use different methods, for instance, routing data described in this [guide](https://indepth.dev/tutorials/angular/indepth-guide-to-passing-data-via-routing).

## Required parameters

We will start from the most popular kind of params that can be passed via routing - the required parameters. The following URL is an example of the usage of the required parameters:

```tsx
localhost: 4200 / profile / 152;
```

The route from above can point to a component that displays a profile and to do that, it needs to know which profile to show. We could use various techniques to pass the complete profile object from the view where a user selects to which profile wants to redirect. However, that’s not the point of our guide. We want to make sure, that after refreshing the browser or passing the link to someone else, the application will show the same profile.

Here is what the concept could look like:

![https://s4.gifyu.com/images/ezgif.com-gif-maker20b1fec6d7d30b73.gif](https://s4.gifyu.com/images/ezgif.com-gif-maker20b1fec6d7d30b73.gif)

In such cases, it’s usually preferred to use the required parameter which can uniquely reference the object we want to get. In our example, we can use profile id `152` to fetch the complete profile object using that specific id.

What is important, our profile component doesn’t make much sense without the profile id, that’s why using the required parameter for that kind of job is great - we won’t be in a situation the profile component is active, and there is no id.

### Route definition

How to define routing so that it reflects our required parameters? We have to define the route as usual and prefix any route elements with a colon, to indicate that this is a required parameter of the following name. Take a look at the following route definition in the routing module:

```tsx
const routes: Routes = [{ path: "profile/:id", component: ProfileComponent }];
```

The above example is pretty straightforward, but keep in mind that we may also build complex routes like this:

```tsx
const routes: Routes = [
  { path: "profile/:id/details/:view", component: ProfileComponent },
];
```

Such parameters in the route definition are called tokens, which then are used to create dynamic slots which will be accessible given the token name. So to correctly met requirements for the route definition such as:

```tsx
  { path: 'profile/:id', component: ProfileComponent },
```

We need to open the following URL in the browser:

```tsx
localhost: 4200 / profile / 152;
```

Which will then use `152` as the value of parameter `id`. Keep in mind that navigating to a partial route, such as `[localhost:4200/profile](http://localhost:4200/profile)` won’t open that route, because it lacks the second element.

### Navigating

Angular provides a very easy way of building the route we want to navigate. We will start from the core, so the `Router`. The `Router` provides a function that does the actual navigation, and it is called `navigate`. It takes one required argument - the array of commands. The array consists of URL fragments, which will construct the target URL. It can hold static strings or dynamic content. In our case we could use it as follows, to navigate to our `ProfileComponent`.

```tsx
@Component({...})
export class AppComponent {

  constructor(private router: Router) { }

  onSomething(profileId: number) {
		this.router.navigate(['profile', profileId]);
	}

}
```

It is really that simple! If we have more elements we put every on of them in the array (in the correct order).

There is also another way to navigate and it’s even easier to use. `RouterLink` is an incredibly useful directive provided by Angular. It exposes various routing-related properties once assigned to the HTML element. We will go through the directive’s implementation more deeply in the “In-depth bits” section. Right now let’s focus on the most basic use case for the `RouterLink` — the ability to navigate, without injecting `Router` in our component. Here is how we can achieve the same result as in the code snippet above, but declaring everything in the template:

```html
<a [routerLink]="['profile', profileId]">Navigate</a>
```

That’s all we need to navigate. Quite simple, right?

### Receiving parameters

Alright, we know how to define routing and navigate to route with required parameters - now it’s time to receive data and use it. Following our example of component that displays a profile. We are going to receive the profile id passed through routing as a required parameter and get the complete profile object, based on that id.

We can assume at this point, that in the application we have a service that handles fetching profile data for us, based on the provided id. So the only thing we need to do is to invoke the method with the correct id.

Params related to the route of a component are stored in the `ActivatedRoute` object, that we can inject and use in our components. So for the following route definition:

```tsx
const routes: Routes = [{ path: "profile/:id", component: ProfileComponent }];
```

There is a couple of ways to get the required parameter `id` from the `ActivatedRoute`:

- `params` property - to get Observable of the required properties scoped to the route
  ```tsx
  this.route.params.subscribe(...);
  ```
- `paramsMap` property - to get Observable that contains a map of the properties
  ```tsx
  this.route.paramMap.subscribe(...);
  ```
- `snapshot` property - to access the current route snapshot and get static values, instead of the Observable (both `params` and `paramsMap` can be used)

  ```tsx
  this.route.snapshot.paramMap...

  this.route.snapshot.params...
  ```

We may wonder, why we should care about the Observable data when we can simply access static values using the `snapshot`. By default, Angular tries to reuse the components in memory and not recreate them if that’s not necessarily. So for instance, when we are navigating from here:

```tsx
localhost: 4200 / profile / 152;
```

to here:

```tsx
localhost: 4200 / profile / 263;
```

We are targeting the same component - the `ProfileComponent`. In this case, it won’t be destroyed during the transition and created from scratch. This means the `constructor` or even the `ngOnInit` won’t be called “again”! Because of this, we won’t be able to check the newest snapshot, because we will not know when to check for it.

That’s when the observable way becomes handy - we can subscribe for any param changes. So even though the component instance is reused, we will still get the new param, and update the UI accordingly.

This mechanism is handled by `RouteReuseStrategy`. By default, it uses `[BaseRouteReuseStrategy](https://angular.io/api/router/BaseRouteReuseStrategy)` implementation — but we can override it and write our own if we want.

For our guide, we will use the Observable with a Map. So in order to receive the `id` parameter in the component we simply have to get it from the `paramMap`, like this:

```tsx
@Component({...})
export class ProfileComponent implements OnInit {

  constructor(private route: ActivatedRoute) {}

  ngOnInit(): void {
    this.route.paramMap.subscribe((params) => {
      const id = params.get('id');
    });
  }

}
```

The map has two useful methods available for us:

- `has(parameterName)` - which checks whether the map contains a given parameter
- `get(parameterName)` - which retrieves the parameter value

Now we can use the id to get complete profile data using some service. The exact implementation may vary depending on your specific needs, and it can look for instance as follows:

```tsx
@Component({...})
export class ProfileComponent implements OnInit {

  profile;

  constructor(
    private route: ActivatedRoute,
    private profile: ProfileService,
  ) {
  }

  ngOnInit(): void {
    this.route.paramMap.subscribe((params) => {
      const id = params.get('id');
      this.profile = this.profiles.getPerson(id);
    });
  }

}
```

We can then use the `profile` property of the component to display data in the template.

## Query params

Query parameters are not required, and are not scoped to a single route fragment - query params are available for the entire URL and can be read from any component. The example of a URL with query params is presented below:

```tsx
localhost:4200/profile/152?lastVisited=215
```

So it’s all about the fragment after the`?` sign. In this example, there is only one parameter called `lastVisited` with a value of `215`.

We use query params for any optional parameters and we can preserve them so no matter to which route we will navigate next the query parameters will stay. This means query parameters can store information between many routes.

In this guide, we will implement a feature to display information about previously displayed profile. It can be used to redirect to the previous one if a user wants to go back.

The concept of the last visited profile feature is presented in the gif below:

![https://s4.gifyu.com/images/ezgif.com-gif-maker-293547bb85971550b.gif](https://s4.gifyu.com/images/ezgif.com-gif-maker-293547bb85971550b.gif)

Although in our guide we will use query params to store a single value, keep in mind that we can assign multiple query params and each of them can store not only a single string, but also an array or even an object if we handle the encoding - more about that in the “In-depth bits” section.

### Route definition

Query params are not related to any specific route, but URL in general. They are also completely optional. With that in mind, there is no possibility to include them in the route definition - because of their dynamic nature. So the route definition stays the same!

### Navigating

Similarly to the required parameters, we can use the `Router` implicitly or the `RouterLink` directive. Let’s start with the `Router`. We will use the same `navigate` method as in the section above. However now, we will include a second (optional) argument when calling the method. The second argument contains an object with many options that modifies `Router` navigation strategy. The options can include any query parameters we want to add to the URL. It works as follows:

```tsx
this.router.navigate(["profile", id], {
  queryParams: {
    lastVisited: "last-visited-id-here-...",
  },
});
```

The `queryParams` object contains key-value pairs, where the key is a query param name.

We can use a similar technique to navigate with query params using the `RouterLink` directive. The directive exposes additional input property called `queryParams` which expects the same object that we assigned in `router.navigate` method. It can be implemented as follows:

```html
<a
  [routerLink]="['profile', id]"
  [queryParams]="{ lastVisited: 'last-visited-id-here-...' }"
>
  Navigate
</a>
```

### Receiving parameters

Reading query params works in the very same way as with required parameters - we just have to use a different set of properties to access the data. We can use:

- `queryParams` property - to get Observable of the query params shared by all routes
  ```tsx
  this.route.queryParams.subscribe(...);
  ```
- `queryParamMap` property - to get Observable that contains a map of the query params
  ```tsx
  this.route.queryParamMap.subscribe(...);
  ```
- `snapshot` property - to access the current route snapshot and get static values, instead of the Observable (both `queryParams` and `queryParamMap` can be used)

  ```tsx
  this.route.snapshot.queryParams...

  this.route.snapshot.queryParamMap...
  ```

Once again we are going to use the Observable way with the map. To access the `lastVisited` query param we can use `queryParamMap` as follows:

```tsx
@Component({...})
export class ProfileComponent implements OnInit {

  constructor(private route: ActivatedRoute) {}

  ngOnInit(): void {
    this.route.queryParamMap.subscribe((queryParams) => {
      const lastVisitedId = queryParams.get('lastVisited');
    });
  }

}
```

We can use the id to get the complete profile as in the required parameters section and display what we want in the template.

## In-depth bits

In this section, we will go deeply into Angular features and go through details of the `RouterLink` directive, other kinds of routing params, and how to serialize the data.

### RouterLink directive

You may ask, why we use some directive to make a redirection when we could simply use what HTML provides us:

```tsx
<a href="profile/125"> Navigate </a>
```

We often don’t want to use `href` attribute implicitly because the result of the user click, will be the full reload of the entire page. We will lose all the application state, and user input data, and the page will blink resulting in a very bad user experience. The advantage of Angular `Router` is that we can make the actual navigation without reloading the app. That’s why it is recommended to use `RouterLink`.

How the `RouterLink` works? The directive does many useful things, but in this guide, we will cover the simple navigation case. `RouterLink` receives URL fragments passed to `routerLink` input property:

```tsx
<a [routerLink]="['profile', id]">  Navigate </a>
```

The implementation is pretty flexible so you don’t have to pass the URL fragments as an array, if the route is quite simple you may use a plain string, as follows:

```tsx
<a routerLink="profile/125"> Navigate </a>
```

No matter how you pass the fragments, the directive will store them as a `commands` array. The `RouterLink` uses `HostListener` to listen to all `click` events on that element. Once the user clicks on the element, `HostListener` will trigger `onClick` function which executes the Angular navigation mechanism using the well-known `Router` class, stored commands to build the target URL, and any other optional options that we set.

So in other words, we can say that the `RouterLink` directive is a wrapper for `Router` that we can use directly in the template.

### Matrix params

The most popular kind of params are required and query params, but there is also the third kind - optional params. The optional params are often called matrix params. That’s because they use the matrix URL notation to separate params in a single route fragment. That’s right - there can be multiple optional params assigned to a route.

For instance, let’s say we want in some cases to pass not only a profile id (required parameter) but also the type of the profile. The URL will look as follows:

```tsx
localhost: 4200 / profile / 152;
type = person;
```

It may look similar to query params at glance, but keep in mind the optional params can be assigned to the specific URL fragment, while query params are always present at the end, and available in every route scope. Consider such URL:

```tsx
localhost:4200/profile/152;type=person/tasks/task/51?lastVisited=1
```

Information about the last visited id is general information available in all the scopes because it is a query param. However type of the profile is an optional param stored against only a single scope of the route. Let’s take a look at the route definition that follows the proposed idea:

```tsx
const routes: Routes = [
  {
    path: "profile/:id",
    component: ProfileComponent,
    children: [
      {
        path: "tasks",
        component: TasksComponent,
        children: [{ path: "task/:id", component: TaskComponent }],
      },
    ],
  },
];
```

Only `ProfileComponent` will have access to `type` matrix parameter - because it belongs to only this scope - the same as the required parameter `id`.

How we can navigate and pass matrix params? It’s quite easy - in the commands array, we have to put them in the object, and the object will be converted into matrix params. We can use the `Router` itself or `RouterLink` - the syntax is the same:

```tsx
this.router.navigate(["profile", id, { type: "person" }]);
```

The `{type: 'person'}` will become the matrix param at this point.

Optional parameters can be accessed the same way as the required parameters. We can use `paramMap` or `params` to access them from `ActivatedRoute`.

### Serializing data in URL

Sometimes we may need to store something bigger than just a short string of something’s id. We usually use query params for storing such data. Let’s for instance think about a basic e-commerce site with a page full of products and filters. The filters choice should be persisted in the URL, in case you want to pass the link to someone else, or store it as a bookmark. Storing filter choice as query params is a great idea - but we need to make this right.

Let’s assume that the choice of the filter can be represented by an object that looks as follows:

```tsx
const filters = {
  page: 1,
  priceRange: [5, 10],
  brands: ['Angular'],
  ...
}
```

When we put this object simply in the router navigation call, the URL won’t be produced correctly, because at some point Angular will convert our filters Object to string saying `Object Object`.

What we have to do is to serialize the object into a string and pass that as query params. Then in the component, we can simply deserialize it to read the object.

We need to be careful to use only URL-safe characters — that’s why we will use [encodeURIComponent](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/encodeURIComponent#try_it).

Navigation implementation:

```tsx
const filters = {
  page: 1,
  priceRange: [5, 10],
  brands: ['Angular'],
  ...
}

this.router.navigate(['products'], {
  queryParams: {
    filters: encodeURIComponent(JSON.stringify(filters))
});
```

This will convert our object to stringified version. Reading such data is easy as well, we just need to deserialize it, as follows:

```tsx
@Component({...})
export class ProductsComponent implements OnInit {

  constructor(private route: ActivatedRoute) {}

  ngOnInit(): void {
    this.route.queryParamMap.subscribe((queryParams) => {
      const serializedFilters = queryParams.get('filters');
			const filters = JSON.parse(decodeURIComponent(serializedFilters));
    });
  }

}
```

## Summary

There are many methods to pass data in Angular. Using route parameters we can persist information not only for the life of the user session but beyond - after the refresh, in browser bookmarks, or when sharing a link to anyone else. Route parameters stored in the URL make it uniquely coupled with the current app state, and with the correct implementation, we can be sure that each time we access the URL we will see the same view.

**Semantic HTML & accessibility:**

Tiny note at the end - I encourage you to use `<a>` elements to register navigation when possible, and don’t use other elements like `<button>` for instance with click function that only calls `router.navigate()`. For such cases consider using `<a>` with `RouterLink` directive which will result in readable HTML, great user experience, and improved accessibility in your application.

## **Live app**

[Here](https://github.com/MaciejWWojcik/InDepth-guide-routing-data) you may find the app with the complete code for this guide. Enjoy!
