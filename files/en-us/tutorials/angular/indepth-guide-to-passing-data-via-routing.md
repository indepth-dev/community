---
title: InDepth guide to passing data via routing - Angular Tutorials | indepth.dev
author_name: Maciej Wójcik
author_link: https://twitter.com/maciej_wwojcik
discussion_link: https://github.com/indepth-dev/community/discussions/276
display_name: InDepth guide to passing data via routing
---

# **InDepth guide to passing data via routing**

## What you will learn

- How to send and receive route data.
    - static data (`RouterConfig` property)
    - dynamic data (`NavigationExtras`)
- How does the Angular navigation feature works and how it populate the data in browser history.


## Intro

The concept of passing data through components in web applications is one of the most popular aspects of our daily programming routine. In Angular, we can implement such a mechanism in many ways. The most popular ones are listed below:

- via `Input` / `Output` properties
- via a `Service`
- via `Routing`

The first two ways are usually well known and used, but often we tend to forget about the third option which is as powerful as the other ones. The great advantage of using the routing to pass the data is that it requires only a slight change in the existing navigation code, without a need of implementing a new service or binding template properties in many files. 

So if you are not yet using routing for passing data or are not yet familiar with this concept, you should check this guide and start using this Angular built-in feature in your codebase.

If you are looking for something more in-depth, we will also investigate how the Angular routing works, and how we can use this knowledge to receive data.

## Passing data

First, we should briefly discuss when it makes sense to use routing to pass data, because as with every other way to pass the data it has some advantages and disadvantages. 

There are two main use cases for choosing the routing feature to pass the data:

1. When we want to customize a component based on the route
2. When we want to pass some information to a component after redirection (usually some temporary data, based on what a user did in the previous route) 

### Use cases

To best imagine the use cases, that we are going to implement in this guide we will focus on specific, and real-world features for web applications. We will implement the following features:

1. A component that will behave and look different depending on which route it will be placed
2. Set of components to create a stepper view to create some complex business object in steps

We will implement an application to create and display profiles of people.

The profile will be displayed via a component placed in two routes:

- `full` - in a full-size layout
- `short` - in a smaller layout, containing less information

The profile view will work as follows:

![https://i.imgur.com/Za1Kzyc.gif](https://i.imgur.com/Za1Kzyc.gif)

In case you wondering about the routing configuration, here is a sneak peek:

```tsx
const routes: Routes = [
  { path: 'full', component: ViewProfileComponent },
  { path: 'short', component: ViewProfileComponent },
];
```

Now, the profile creation. The profile object will be created in other components in two steps:

- `create-profile/type` - for a user to choose an appropriate profile type as the first step of the data acquisition process
- `create-profile/data` - for a user to fill all the data in a form, dependent on the profile type

The profile creation will work as follows:

![https://i.imgur.com/OJ1IzUC.gif](https://i.imgur.com/OJ1IzUC.gif)

As in previous use case, here is a proposition for the routing configuration:

```tsx
const routes: Routes = [
  {
    path: 'create-profile', 
		component: CreateProfileComponent, 
		children: [
      { path: 'type', component: ProfileTypeComponent },
      { path: 'data', component: ProfileFormComponent },
    ]
  }
];
```

## The profile component

Angular offers an interesting property in `Route` definition, which we use when defining the routing modules. It is called `data` and it is a simple object of key-value pairs. So basically we can use this object to store anything we want, from simple values to objects, arrays, or even functions. 

The following snippet presents the basic syntax of this feature: 

```tsx
const routes: Routes = [
  { 
		path: '',
    component: MainComponent,
    data: { key: 'value as string' },
  },
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule {
}
```

Now once the data is set in, we should be interested in reading this data in our components. Angular stores the data assigned to the route and makes it accessible via the `ActivateRoute` object. As usually when it comes to getting values from `ActivateRoute` we can do it in two ways:

- Observable way - via subscribing to `ActivateRoute` property changes
- Snapshot way - via accessing the `ActivateRouteSnapshot` and then getting property value from the current route state

An observable way is presented below: 

```tsx
@Component({...})
export MainComponent{

	constructor(
		private route: ActivatedRoute,
  ) { }

  ngOnInit() {
		this.route.data.subscribe((data) => {
			const value = data['key']
    });
  }
}
```

We can easily subscribe to `data` changes via exposed Observable. However if the value we expect seems static, or at least we don’t care about changes we can use the `ActivateRouteSnapshot` to focus on the current snapshot and don’t use Observables for that task. The snapshot implementation is the following:

```tsx
@Component({...})
export MainComponent{

	constructor(
		private route: ActivatedRoute,
  ) { }

  ngOnInit() {
		const value = this.route.snapshot.data['key'];
  }
}
```

Now let’s get back to our use case. We want to have a component to display profile information. Additionally, we want this component to work in two layout modes - a full one and a smaller one which displays less information. The component will be placed on different routes, which will define the component’s view mode. 

Let us start with the routing definition for this case - two routes with our component:

```tsx
const routes: Routes = [
  { path: 'full', component: ViewProfileComponent },
  { path: 'short', component: ViewProfileComponent },
];
```

This is a basic definition, but we need to extend it to let our component know, how it should display the profile: 

```tsx
const routes: Routes = [
  { path: 'full', component: ViewProfileComponent, data: { fullView: true } },
  { path: 'short', component: ViewProfileComponent , data: { fullView: false }},
];
```

We assigned an object to the `data` property in each of the routes. Inside the object, there is a single boolean property, called `fullView` to inform the component in which mode it is used. 

The routing configuration is ready, and the only thing left is the component implementation. Regarding the component code, we can split the problem into two parts:

- understanding in which mode the component is used and somehow storing that information
- reacting to the mode in the component’s template and presenting different views based on the mode

The second problem is not really in the topic of this guide so I will just present an example template later. For now, let us focus on the component’s class code.

We defined the `data` object in the routing as a static boolean value, which is either `true` or `false` and does not change over time. That means we don’t have to use a more complicated - Observable way of receiving the data, we can simply use the current snapshot state to get the value we are interested in. The following snippet presents such implementation:

```tsx
@Component({...})
export ViewProfileComponent {

	fullMode: boolean;

	constructor(
		private route: ActivatedRoute,
  ) { }

  ngOnInit() {
		this.fullMode = this.route.snapshot.data['fullView'];
  }
}
```

We created a component’s property called `fullMode` and then we assigned its value based on the data that comes from the snapshot of the current route. We can use this property to apply layout and styling changes using various techniques. In this guide we proposed to use an `NgIf` directives to display or hide particular parts of the template, as follows:

```html
<section>
  <img *ngIf="fullView" [src]="profile.image">
  <article>
    <p>{{profile.name}} {{profile.surname}}</p>
    <p *ngIf="fullView">{{profile.address}}</p>
  </article>
</section>
```

Remember, that you can use anything you want, like `HostBinding`, or `ngClass` and `ngStyle` techniques to make your template dynamic. 

The component implementation is completed. There is only one thing worth mentioning - that with using this approach to pass data via routing, we don’t have to do anything new to the code that actually does the redirection to this route. So the following code still works as expected:

```html
<a routerLink="full">redirect to full version</a>
<a routerLink="short">redirect to short version</a>
```

Same goes with using the `Router` itself:

```tsx
this.router.navigate(['full']);
this.router.navigate(['short']);
```

The information about the mode is stored in the route configuration, and we don’t have to reapply it anywhere. That’s all regarding the route’s `data` property.

## The profile creation steps

There is another feature implemented in Angular for passing data via routing. It is different than the previous one, mainly because of the moment, when the actual data transfer happens. In the previous situation, data was set in routing configuration on application init. 

This feature works in the moment of actual redirection and it can be used to pass dynamic data that is usually created at runtime, based on the current state and user action. 

In Angular, we can use different methods to redirect to a certain route, for instance: 

- using `router.navigate()` method
- using `routerLink` directive placed on HTML element

In the beginning, we will go through the implementation using the `router.navigate()` method, and at the end, we will reimplement our code using `routerLink`. 

Most basic usage of the navigate function is as follows:

```tsx
this.router.navigate(['path', 'to', 'route']);
```

It takes as the first argument array of URL fragments, which constructs the target URL. What’s important for our situation, is that this function takes also a second, optional argument. It is called extras and it has a property called `state`. The state is again, a simple key-value pairs object that can be passed to any navigation. The feature is presented in the following snippet:

```tsx
this.router.navigate(
      ['path', 'to', 'route'],
      { state: { key: 'value as string' }}
    );
```

Please note that the actual value passed to the state is generated at the moment of the actual redirection so the developer can easily pass dynamically created values.

Let us start working on our use case and implement this way of passing the data. We will also learn how to receive data from the state. We want to implement a set of components that will result in creating a new profile. We want to do it in two steps:

- choose profile type - it can be a `Person`, `Company`, or a `Group`
- fill profile form to create the `Profile` object

We will start by defining the routing, to clearly state how the components will be connected:

```tsx
const routes: Routes = [
  {
    path: 'create-profile', 
		component: CreateProfileComponent, 
		children: [
      { path: 'type', component: ProfileTypeComponent },
      { path: 'data', component: ProfileFormComponent },
    ]
  }
];
```

So we are going to have two components:

- `ProfileTypeComponent` for selecting a profile type
- `ProfileFormComponent` for filling the data

What is important for us, `ProfileTypeComponent` will have to pass the information to `ProfileFormComponent` about what type of profile the user wants to create. These two components are not in any direct parent-child relation, so we cannot use the easiest `Input`/`Output` way. We are going to use the navigation state to pass the information about the selected profile type when redirecting to the next step of the profile creation process. 

Let us start with `ProfileTypeComponent` - when the user selects a profile type in the template we are going to call the component’s function to redirect to the next step. This can look as follows:

```tsx
@Component({...})
export class ProfileTypeComponent {

  constructor(
    private router: Router,
  ) {
  }

  goToNextStep() {
    this.router.navigate(['create-profile', 'data']);
  }
}
```

However we need to inform the next component about chosen type of profile. We will get the chosen value as function argument, and pass it via the navigation state, as presented below:

```tsx
goToNextStep(profileType: ProfileType) {
    const state = { profileType };
    this.router.navigate(['create-profile', 'data'], { state });
}
```

The template is not that important in the case of this guide, so I won’t spend much time on it - but in case you were wondering how the function gets called, the template may look like the below:

```html
<h1>Please choose profile type</h1>

<button (click)="goToNextStep(ProfileType.Person)"> Person </button>
<button (click)="goToNextStep(ProfileType.Company)"> Company </button>
<button (click)="goToNextStep(ProfileType.Group)"> Group </button>
```

We can assume the data will be sent correctly via the state property. Now it’s a turn to implement the mechanism of receiving the data in the `ProfileFormComponent`.

 

The `ProfileFormComponent` should receive information about the selected profile type in the step before, and gather appropriate input data from the user. For instance, we can assume that for `Company` we want to get the company’s name, and for a `Person`- its name and surname.

Angular offers another interesting `Router` feature which is a function called `getCurrentNavigation`. It returns a current `Navigation` object, only when the navigation takes place. This object gives us access to the navigation state. Let us implement the component logic as follows:

```tsx
@Component({...})
export class ProfileFormComponent {

  profileType: ProfileType;

	constructor(
		private router: Router,
	) {
		const state = this.router.getCurrentNavigation().extras.state;
		this.profileType = state['profileType'];
  }
}

```

As you see, we are getting the state from the current navigation and then assigning the `profileType` value to the component’s property so we can use it to implement the form.  

Implementing the template should be done with respect to the profile type. So to let the user fill in only the data required for the chosen profile type. The complete process of passing the data through routing is done at this point. 

There is a small extension to this topic regarding the actual redirection function. In the example above we have used the `router.navigate()` method to navigate. What if we would like to use the `routerLink` directive in the template and don’t implement any TypeScript code? Well, it’s easy, take a look at the following snippet:

```tsx
<h1>Please choose profile type</h1>

<a routerLink="../data" [state]="{profileType: ProfileType.Person}">Personal</a>
<a routerLink="../data" [state]="{profileType: ProfileType.Group}">Group</a>
<a routerLink="../data" [state]="{profileType: ProfileType.Company}">Company</a>
```

The `routerLink` directive exposes `state` input which we can use to specify the state associated with that link.

## In-depth bits

### Angular Router

There are some interesting details regarding the state object, how the navigation works under the hood, and why, in the example above, we have accessed state in the component’s `constructor` scope, and not in the `ngOnInit`. 

Navigation in Angular is based heavily on the `Router` class which is responsible for handling all navigation requests, with that `Router` needs to not only change correctly the URL, but also keep track of the transitions, and information about them, and work closely with the browser to support browser `History` and `Location`.

When we call the `router.navigate()` function, the `Router` registers this as a new transition, puts it in the queue, and starts processing it as soon as possible. While processing the transition, the `Router` stores information about the current process and exposes it via a `router.getCurrentNavigation()` - that is precisely why we can call this function in the component’s constructor and receive information about the current transition, which stores the state object. 

This is very important because if you try to put our code in the `ngOnInit`, instead of the `constructor` of the component, you will receive `undefined` instead of the state! It works that way because once you are in `ngOnInit` the transition is completed and there is no current navigation that we could observe. Although there is a way to get `state` data later than the component’s construction. 

![How Angular Router works under the hood](https://i.imgur.com/9Or1pws.png)

How Angular Router works under the hood

So what happens later in the transition process from above? After storing current navigation, the `Router` does a bunch of stuff to support various navigation options and finally checks if the navigation can be allowed using Guards. Once everything is ready for the actual navigation, the `Router` loads components and then updates `Location` and `History`. This results in an updated URL but also with updated browser history by the new record and this is a second important moment for our case. You can check more about the browser API [here](https://developer.mozilla.org/en-US/docs/Web/API/History/replaceState).

We can use this knowledge to receive state, not only in the component’s `constructor` but on `ngOnInit` or anytime later. We can do it using the `history` global object as presented below:

```tsx
@Component({...})
export class ProfileFormComponent implements OnInit {

  profileType: ProfileType;

	ngOnInit() {
    const state = history.state;
    this.profileType = state['profileType'];
	}
}

```

If you are interested more about how Angular Router works, here are links to great articles by Nate Lapinski:

- [Introduction article](https://admin.indepth.dev/the-three-pillars-of-angular-routing-angular-router-series-introduction/)
- [Router Navigation Cycle article](https://admin.indepth.dev/angular-router-series-pillar-2-understanding-the-routers-navigation-cycle/)

### Data not only for components

Going back for a while to a static `data` in routing definition. Although we were working with components to receive the data, this way also works great as a technique for customizing other Angular elements. We can use the data in Guards and Resolvers!

There is a very popular concept which we will use as an example. Let us imagine we have various roles within the application and user must met some role criteria to enter the route. The routes in this case can look something like that:

```tsx
const routes:Routes = [
  {
    path: '',
    component: HomeComponent,
    data: { roles: [] },
    canActivate: RoleGuard
  },
  {
    path: 'admin',
    component: AdminComponent,
    data: { roles: ['admin'] },
    canActivate: RoleGuard
  },
  {
    path: 'user',
    component: UserComponent,
    data: { roles: ['user', 'admin'] },
    canActivate: RoleGuard
  },
];
```

Each route has `RoleGuard` assigned which check is user has a role that is required to enter the route. This way the Guard is implemented in very generic manner and a single Guard class can handle every route. The customization happens in the `data` property which keeps information about required roles. The Guard can read that and decide whether return false or true. Guard implementation can look as follows:

```tsx
export class RoleGuard implements CanActivate {

  constructor(private user: UserService) {
  }

  canActivate(route: ActivatedRouteSnapshot, state: RouterStateSnapshot): boolean {
    const requiredRoles = route.data['roles'];
    return this.user.isAuthorized(requiredRoles);
  }
}
```

This is a great technique to write reusable classes, that can be customized depending on the actual route.

## Summary

Angular Routing is a very powerful feature and we can use it not only to navigate between various views in our application but also to pass the data between the components. We can choose between two options to pass the data:

- the `data` property of the `Route` - we can assign it in the routing configuration
- `state` property in `NavigationExtras` - we can assign value in a moment of the actual navigation

The first approach is usually very good for customizing components based on the route or passing any other static kind of data. The state approach allows us to pass dynamic data, which can change in every navigation and usually contains data specific to user action.

**Live app:**

[Here](https://github.com/MaciejWWojcik/InDepth-guide-routing-data) you may find the app with complete code for this guide. Enjoy!