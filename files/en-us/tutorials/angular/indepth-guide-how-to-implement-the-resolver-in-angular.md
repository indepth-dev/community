---
title: InDepth guide on how to implement the Resolver in Angular - Angular Tutorials | indepth.dev
author_name: Varvara Sandakova
author_link: https://twitter.com/varvarasandako1
discussion_link: https://github.com/indepth-dev/community/discussions/284
display_name: InDepth guide on how to implement the Resolver in Angular
---
# **InDepth guide on how to implement the Resolver in Angular**

## Use cases/problems
- How to preload data during the page navigation

## What you will learn
- How to implement the Resolver.
- The execution order of multiple Resolvers.
- How to get access to the resolved data in the component.
- What will happen if your Resolver returns the error?

## Intro
The most popular task in our daily routine as front-end developers is fetching data from the backend. The potential place where we can load data depends on the task we want to solve. In Angular there are several places where we can load data:

- App Initializer - application start.
- Resolver - the transition between routes.
- Lifecycle hooks - component initialization.
- User interaction actions - loading data by clicking the button.

In this tutorial, we will look at the second case in the list - preloading data while transitioning between routes. We will learn how to preload data during page navigation and briefly review how to get access ****to the resolved data in the component. In addition to this, I want to look in depth at the execution process of the Resolver:  successful route navigation, canceling navigation and returning errors from the Resolver.

Let's start.

### Use case:

1. Let's imagine the task where you need to implement the page with a navigation list of three users.

![](https://i.imgur.com/eb5obkA.png)

By clicking on each user you want to open a new page with a **preloaded** table with a contact list for the current user.

Please note that in our case we want to display a table with **an already loaded** contact list (the contact list of the current user is ready before component initialization) and we don’t want to proceed with navigation if the data load fails.

So, we are going to load data by implementing the Resolver. Before the implementation of resolvers, we need perfectly understand the use case and compare all pros and cons. When we load a big amount of data the user experience suffers a lot -  resolvers wait for the asynchronous tasks to finish, before the routing continues. The resolved data is available synchronously when the component starts. That means, the user can wait for several seconds and nothing happened. You can find more information about handling application navigation in this [article](https://indepth.dev/posts/1023/the-three-pillars-of-angular-routing-angular-router-series-introduction).

Before we start, let me briefly refresh what is the function of the resolver.  If you’re already familiar with this concept, skip directly to the [implementation](https://github.com/VarvaraSandakova/resolver-in-angular).

## A brief overview of resolver

Resolver is just a simple middleware in the angular routing process which loads data before the navigation takes place. In other words, the router is waiting for data to be available first before the component defined for the route is rendered.

Angular's router supports as many resolvers per route as you want. The resolver will wait for the async call to complete and only after processing the async call, it will proceed with navigation to the corresponding URL. The route won't be activated until ALL resolves are complete/fulfilled.   Thus, if you want to do something (service call), even before the component is initialized, the resolver is the way to go.

Once users click on the link, the following takes place:
1.  Execute Resolver and load data

1. Activate the route by instantiating the component
2. The loaded data will be available for the component in the activated route data property.

## How to

Let’s get to the practical part now

1. The first step would be the implementation of the resolver to have the possibility for loading contacts before navigation to the page:

```tsx
import {Injectable} from "@angular/core";
import {ActivatedRouteSnapshot, Resolve} from "@angular/router";
import {UserContacts, UserService} from "./user.service";
import {delay, Observable, of} from "rxjs";

export interface UserContacts {
    name: string;
    telephone:number;
}

const userContactsMock = [
    {name: 'Anna', telephone: 123},
    {name: 'Jim', telephone: 345},
    {name: 'Joana', telephone: 678},
    {name: 'Tim', telephone: 876},
    {name: 'Katty', telephone: 432}
]

@Injectable({
    providedIn: 'root',
})

export class ContactsResolver implements Resolve<UserContacts[]> {
    constructor() {}

    resolve(route: ActivatedRouteSnapshot): Observable<UserContacts[]> {
        // We just mock user information in the resolver for avoiding complications.
        // Usually you use services in Angular for that
        //By the way, we used a delay operator to simulate asynchronous HTTP requests from the service.
        return of(userContactsMock).pipe(delay(1000));
    }
}
```
According to the Angular [documentation](https://angular.io/api/router/Resolve), we implement a resolver using the Resolve interface.

```tsx
interface Resolve <T> {
    resolve(route: ActivatedRouteSnapshot, state: RouterStateSnapshot): Observable<T> | Promise<T> | T
}

```
Interface that classes can implement to be a data provider. A data provider class can be used with the router to resolve data during navigation. The interface defines a `resolve()`
method that invokes right after the [ResolveStart](https://angular.io/api/router/ResolveStart) router event. 
The router waits for the data to be resolved before the route is finally activated. According to our example, 
`ContactsResolver` implements interface Resolve with type `UserContacts[]`.

2. Now we can set up the routing module to include the resolver - app.route.ts.

```tsx
import { AppComponent } from './app.component';
import { ContactsComponent } from './contacts/contacts.component';
import {UserComponent} from "./user/user.component";

const routes: Routes = [
  {
    path: '',
    pathMatch: 'full',
    component: UserComponent
  },
  {
    path: 'user/:id',
    component: ContactsComponent,
    resolve: { contacts: ContactsResolver }
  }
];
...
})
export class AppModule { }

```

In the above code, we add `ContactsResolver` to the route related to `ContactsComponent` . Do not forget to add your services and resolver to the providers for `AppModule`.

3. Finally, we are going to have the access to resolved data in the `ContactsComponent` - app-contacts.component.ts

```tsx
import { Component, OnInit } from '@angular/core';
import {ActivatedRoute} from "@angular/router";
import {UserContacts} from "../user.service";

@Component({
  selector: 'app-contacts',
  templateUrl: './contacts.component.html',
  styleUrls: ['./contacts.component.scss']
})
export class ContactsComponent implements OnInit {
  public contacts: UserContacts[] = [];

  constructor(private route: ActivatedRoute) {}

  ngOnInit(): void {
    this.contacts = this.route.snapshot.data['contacts'];
  }
}

```
As you can see that we inject `ActivatedRoute` to the constructor of the `ContactsComponent`.

Then we get preload data from the snapshot property of the ActivatedRoute and store it contacts property. In this case, you need to use `this.route.data` observable you can write:

```tsx
this.contacts$ = this.route.data.pipe(map(data => data.contacts));

```

The result would be:

![](https://i.imgur.com/wyD7Svv.png)

As a result, we get what we wanted -  open a new page with **preloaded** contacts for the current user by clicking on the user. But “waiting” time is still there, so the data isn’t really preloaded in the sense that you don’t have to wait on click, but rather that it’s available before the component is initialized.

In case, when you switch between two pages from `/user/1` to `/user/2` (as in our example) you can define your own reusing strategy by implementing a class [BaseRouteReuseStrategy](https://angular.io/api/router/RouteReuseStrategy). Route reuse strategy help to avoid destroying the corresponding component during the navigation process,. So, imagine you would try:

1. Open the page `/user/1`
2. Then switch to the page with all users `/users`
3. Switch to the page`/user/2`

In this scenario, Angular destroys `ContactsComponent` (which refers to `/user/:id` route ) when navigating to `/users` and would be recreated when navigating to `/user/2`.

Let`s try to implement our own route-reuse-strategy to avoid destroying a component  `ContactsComponent` when you know that you will need it again and soon.

1. Create a class `CustomRouteReuseStrategy` implementing the abstract class `RouteReuseStrategy` .

```tsx
import { RouteReuseStrategy, ActivatedRouteSnapshot, DetachedRouteHandle } from '@angular/router';

export class CustomRouteReuseStrategy implements RouteReuseStrategy {
 private acceptedRoutes: string[] = ["user/:id"];
  private storedRoutes = new Map<string, DetachedRouteHandle>();

// determines if this route (and its subtree) should be detached to be reused later.
  shouldDetach(route: ActivatedRouteSnapshot): boolean {
	// check to see if the route's path is in our acceptedRoutes array
  return this.acceptedRoutes.indexOf(route.routeConfig.path) > -1)
  }

// stores the detached route.
  store(route: ActivatedRouteSnapshot, handle: DetachedRouteHandle): void {
    this.storedRoutes.set(route.routeConfig.path, handle);
  }

// determines if this route (and its subtree) should be reattached.
  shouldAttach(route: ActivatedRouteSnapshot): boolean {
    return !!route.routeConfig && !!this.storedRoutes.get(route.routeConfig.path);
  }

// retrieves the previously stored route.
  retrieve(route: ActivatedRouteSnapshot): DetachedRouteHandle {
    return this.storedRoutes.get(route.routeConfig.path);
  }

// determines if a route should be reused.
  shouldReuseRoute(future: ActivatedRouteSnapshot, curr: ActivatedRouteSnapshot): boolean {
    return future.routeConfig === curr.routeConfig;
  }
}
```
2. Provide `CustomRouteReuseStrategy`  in our module:

```tsx
import { NgModule } from '@angular/core';
import { RouteReuseStrategy } from '@angular/router';
import { CustomRouteReuseStrategy } from './custom-route-reuse-strategy';

@NgModule({
 ...
  providers: [{
    provide: RouteReuseStrategy,
    useClass: CustomRouteReuseStrategy,
  }],
})
export class AppModule { }
```

## **In-depth bits**

In this section I’d like to spend time on the common patterns with resolvers that cause confusion.
Particularly, let’s take a look at what happens when:

- navigation is canceled;
- multiple resolvers are defined for the route;
- route is reused;

For the most curious, we’ll also take a look at the or**der of the execution guards and resolvers**

There are important things that you need to know about the **execution process** of resolvers:

### - Canceling navigation

Let's assume that the `ContactsResolver` returns an error:

```tsx
@Injectable({
	providedIn: 'root',
})

export class ContactsResolver implements Resolve<any> {
	constructor(private userService: UserService,
		private router: Router) {
	}

resolve( route: ActivatedRouteSnapshot, state: RouterStateSnapshot): Observable<{ user: User }> {
    // this.router.navigateByUrl('/error-page');
    return throwError(error);
	}
    }
}
```
Now take a minute and think about What will happen next if UserResolver returns the error?

a) UserResolver executes but the page with user contacts will not render/navigate.

b) UserResolver executes and navigates to the user contacts page.

The correct answer b - UserResolver executes but the page with user contacts will not render/navigate.
It happened because Observable from the service returns the error(in another case it could be never complete) then you won’t be able to navigate to the page. 
In other words, the route will not resolve and your page will not render.

To understand more details you can activate tracing support by passing a flag when it’s added to the app routes, like so:

```tsx
@NgModule({
	imports: [RouterModule.forRoot(routes, { enableTracing: true })],
	exports: [ RouterModule ]
})

export class AppRoutingModule {}
```

### - Execution multiple Resolvers

In case, when we need to load a different part of data in one component we can define more than one resolver in the route of the component:

```tsx
 
const routes: Routes = [
  {
    path: '',
    pathMatch: 'full',
    component: UserComponent
  }, 
  { path: 'user/:id',
    component: ContactsComponent, 
    resolve: {
        contact: ContactsResolver,
        account: AccountResolver
        // any other resolver
    }
  }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }

```

As a result, the route won't be activated until ALL resolves are complete. Resolvers are resolved in parallel.

If `Contact` and `Account` are supposed to be resolved in series they should be a single `UserResolver`:
```tsx
class UserResolver implements Resolve<{ foo: any, bar: any }> {
  constructor(
    protected contact: ContactsResolver,
    protected account: AccountResolver
  ) {}

  async resolve(route): Promise<any> {
    const foo = await this.contact.resolve(route);
    const bar = await this.account.resolve(route);

    return { contact, account };
  }
}
```

So, we need to ensure that all data will resolve 100%.

### - Order of the execution guards and resolvers

For investigation of the order of execution guards and resolvers let's create an `AuthUserGuard`:

```tsx
import { Injectable } from "@angular/core";
import {
    ActivatedRouteSnapshot,
    CanActivate,
    Router,
    RouterStateSnapshot
} from "@angular/router";
import { AuthService } from "./auth.service";
  
@Injectable()
export class AuthUserGuard implements CanActivate {
    constructor(
        private authService: AuthService,
        private router: Router) { }
    
    canActivate(
        route: ActivatedRouteSnapshot,
        state: RouterStateSnapshot): boolean | Promise<boolean> {
        
        // lets imagine that this.authService.getAuthStatus() return true if user is authentificatied
        let isAuthenticated = this.authService.getAuthStatus();	
        if (!isAuthenticated) {
            this.router.navigate(['/login']);
        }
        return isAuthenticated;
    }
}
```

We just created a `AuthUserGuard` that implements `CanActivate` interface. ( The deeper investigation of the CanActivate guard is described [here](https://indepth.dev/posts/1211/the-difference-between-the-canactivate-and-canactivatechild-guards). ) In the `isAuthenticated` variable, we stored received the status (true or false) from some `authService`.

So, if a user is authenticated `canActivate` method returns true, if a user is not authenticated we are redirect the user to the login page using `this.router.navigate(['/login']);`

After the implementation of the `AuthUserGuard` we can configure `AppRoutingModule`:
```tsx
// imports

const routes: Routes = [
	{
	    path: '',
	    pathMatch: 'full',
	    component: UserComponent
	},
 {
		 path: 'user/:id'
		 canActivate: [AuthUserGuard],
		 component: ContactsComponent,
		 resolve: {contacts: ContactsResolver}
		 children: [
                // another page with its own Guard and Resolver
		  {
		    path: 'user-details',
		    canActivate: [UserDetailsGuard],
		    component: DetailsComponent,
		    resolve: {details: DetailsResolver}
		   }
		 ]
	}
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }

```
We will add a children array for the route with `ContactsComponent`

where we define a new `path`, add `UserDetailsGuard` and a new resolver `DetailsResolver`.

In general, the order of execution of the resolvers and guards would be - first guards, and second - resolvers. So, to prove that, let’s see the following example:

`AuthUserGuard, UserDetailsGuard, ContactsResolver, DetailsResolver.`

Important to mention when both guard and resolvers are specified, the resolvers are not executed until all guards have run and succeeded. For example, consider the following route configuration:

- In case, when you need one resolver for each child component, you can share the resolver between them:
```tsx

// imports

const routes: Routes = [
  { path: '', 
      resolve: {
        contacts: ContactsResolver
      }, 
      children: [
        { path: 'user_1', component: UserOneComponent},
        { path: 'user_2', component: UserTwoComponent},
        { path: 'user_3', component: UserThreeComponent},
    ]}
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

This case is pretty popular for components with similar functionality. You can create one `ContactsResolver` and get preloading contacts for each component.

## Conclusion

In this tutorial, we learned how to fetch data during the page navigation in Angular using Resolvers. The data is fetched before Angular’s router initializes the component defined for the router. We also consider the execution process of the Resolver and investigate the question of canceling navigation.

So, before thinking about the implementation of Resolver we need to consider the main requirements of the task. Here’s what you need to keep in mind deciding to use Resolvers:

- You should create a Resolver when the data is critical for the component view.
- You should use resolvers for operations that are predictably fast – so your users don't need to wait.
- If the Resolver is not complete, you will not be able to see your component rendered. As a result, the user won’t be able to navigate to the page. If you return null from the resolver, the router will cancel the navigation.
- Do not ignore errors in the Resolver. In case if service returns the error or  Observable is never complete then you won’t be able to navigate to the page.
- The same resolver can be used to provide data to different components.