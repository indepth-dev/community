---
title: How to implement global loaders using HTTP Interceptors - Angular Tutorials | indepth.dev
author_name: Maria Zayed
author_link: https://twitter.com/mariawzayed
discussion_link: https://github.com/indepth-dev/community/discussions/218
display_name: How to implement global loaders using HTTP Interceptors
---

# **How to implement global loaders using HTTP Interceptors**
Web applications are by nature asynchronous and have to communicate to the server. Each request-response pair takes time, so we use visual elements called loaders to let the user know about the process taking place. In this article, I’ll walk you through how to use an [HTTP interceptor](https://indepth.dev/posts/1118/insiders-guide-into-interceptors-and-httpclient-mechanics-in-angular) to catch all HTTP requests and responses to show or hide global or local loaders.

## Project Setup
Let’s set up a brand-new project and create the necessary components by running the following commands:
* **ng new global-loader** - to create a project named global-loader
* **ng g c components/loader** - to create a loader component
* **ng g s services/loader** - to create a loader service
* **ng g interceptor interceptors/loader** - to create a loader interceptor
  
In the end, you should have a folder structure like this:

![1](https://user-images.githubusercontent.com/27064594/143764761-37bb960c-f7ca-4357-8511-06e2db8cfbe5.PNG)

 Due to the fact we will use angular material for the spinner, let’s install it, using this command `ng add @angular/material`. 

When the install is finished, you can import the spinner in the `app.module.ts`. You can learn more about the spinner module from the official [docs](https://material.angular.io/components/progress-spinner/overview)
  
```javascript
// app.module.ts

...
import { MatProgressSpinnerModule } from '@angular/material/progress-spinner';

@NgModule({
  declarations: [
     ...
  ],
  imports: [
    ...
     MatProgressSpinnerModule,
  ],
  ...
 })
export class AppModule {
}
```

An important part of the angular material setup is to also import the theme you want to use. Don’t skip this step, as it can cause problems with your loader visibility. I have decided to use the indigo-pink theme by importing it in `styles.css` (the global styles file)

```javascript
@import '~@angular/material/prebuilt-themes/indigo-pink.css';
```

Now we should be able to use the angular material spinner in our `loader.component.html`.

```html
<!-- loader.component.html -->

<div class="overlay">
  <mat-progress-spinner class="spinner"
                        mode="indeterminate">
  </mat-progress-spinner>
</div>
```

```scss
// loader.component.scss

.overlay {
 position: fixed;
 display: block;
 width: 100%;
 height: 100%;
 top: 0;
 left: 0;
 background-color: rgba(74, 74, 74, .8);
 z-index: 99999;
}

.spinner {
 position: absolute;
 top: 50%;
 left: 50%;
 transform: translate(-50%, -50%);
}
```

Now we can test it by importing the loader component selector in `app.component.html`. Remove everything in `app.component.html` and add the `<app-loader></app-loader>` tag.

### Using the loader service
The idea behind the loader service is to expose a Subject with the name `isVisible`, to whom we can emit true/false values from the interceptor and listen for changes from our loader component.

In case you are not familiar with the Subject, it’s a special type of Observable that can act both as Observable and Observer at the same time. You can learn more about it [here](https://indepth.dev/reference/rxjs/subjects) or from the official [docs](https://rxjs.dev/guide/subject)

In the `loader.service.ts` add the following code

```javascript
import { Injectable } from '@angular/core';
import { Subject } from 'rxjs';

@Injectable({
  providedIn: 'root',
})
export class LoaderService {

  isLoading = new Subject<boolean>();

  constructor() {
  }

  show() {
     this.isLoading.next(true);
  }

  hide() {
     this.isLoading.next(false);
  }
}
```

Basically, we are creating a new variable isLoading with type Subject that expects boolean data and two methods (show and hide) that push data to the Subject.

### Using the interceptor
In the `loader.interceptor.ts` file add the following code

```javascript
import { Injectable } from '@angular/core';
import {
  HttpRequest,
  HttpHandler,
  HttpEvent,
  HttpInterceptor,
} from '@angular/common/http';
import { Observable } from 'rxjs';
import { LoaderService } from '../services/loader.service';
import { finalize } from 'rxjs/operators';

@Injectable()
export class LoaderInterceptor implements HttpInterceptor {

  constructor(private loaderService: LoaderService) {
  }

  intercept(request: HttpRequest<unknown>, next: HttpHandler): Observable<HttpEvent<unknown>> {
     this.loaderService.show();

     return next.handle(request).pipe(
           finalize(() => this.loaderService.hide()),
     );
  }
}
```

This interceptor will change the Subject’s value to true when a request starts (to show the loader) and change the Subject’s value to false (to hide the loader) when the request is finished.
The cool thing about finalize is that it calls our callback function on both success and error responses. This way, we can be sure that our application will not end up with a non-stop spinning loader. You can check the official docs to learn more about [finalize](https://rxjs.dev/api/operators/finalize).

Now, when our interceptor is ready, we just have to tell our angular application to use it. Like this

```javascript
// app.module.ts

...
import {
  HTTP_INTERCEPTORS,
  HttpClientModule,
} from '@angular/common/http';
import { LoaderInterceptor } from './interceptors/loader.interceptor';

@NgModule({
  declarations: [
    ...
  ],
  imports: [
    ...
  ],
  providers: [
     {
        provide: HTTP_INTERCEPTORS,
        useClass: LoaderInterceptor,
        multi: true,
     },
  ],
  ...
})
export class AppModule {
}
```

#### Modify the loader component to use the loader service
As we don’t want to show the spinner non-stop, we will put the whole loader.component.html in `*ngIf` and use the loader service to get the correct state:

```javascript
// loader.component.ts

import { Component } from '@angular/core';
import { LoaderService } from '../../services/loader.service';
import { Subject } from 'rxjs';

@Component({
  selector: 'app-loader',
  templateUrl: './loader.component.html',
  styleUrls: [ './loader.component.scss' ],
})
export class LoaderComponent {
  isLoading: Subject<boolean> = this.loaderService.isLoading;

  constructor(private loaderService: LoaderService) {
  }
}
```

```html
<!-- loader.component.html -->

<div *ngIf="isLoading | async" class="overlay">
  <mat-progress-spinner class="spinner"
                        mode="indeterminate">
  </mat-progress-spinner>
</div>
```

We have to note that this can also be achieved with the traditional subscribe/unsubscribe approach, but I prefer to use the async pipe as it looks cleaner and you shouldn’t care about memory leaks if you miss unsubscribing.

Now, let’s test our solution with a real API call, but first, we should import the `HttpClientModule` in our `app.module.ts` and include it in the imports array.

```html
<!-- app.component.html -->

<app-loader></app-loader>

<button (click)="getData()">Get Data</button>

<ul *ngIf="users.length">
  <li *ngFor="let user of users">{{ user }}</li>
</ul>
```

```javascript
// app.component.ts

import { Component } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { map } from 'rxjs/operators';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: [ './app.component.scss' ],
})
export class AppComponent {

  url = 'https://jsonplaceholder.typicode.com/users';
  users: string[] = [];

  constructor(private httpClient: HttpClient) {
  }

  getData() {
     this.httpClient.get(this.url).pipe(map(res => {
              const users: any[] = [];
              res.forEach(obj => {
                 users.push(obj['name']);
              });
              return users;
           }))
           .subscribe(data => {
              this.users = data;
           });
  }
}
```

In real projects, you should remove the `getData()` logic to a separate service, but for the sake of the demo, we implemented it in the `app.component.ts`.
Now, you should see the loader after clicking on the button, and it will hide after finishing the request.

## Conclusion
Setting up a loader centrally in an interceptor is a smart way to improve the user experience because it shows up whenever there are active requests.
Moreover, it improves the developer’s code quality by creating reusable and scalable code since there are an expected set of steps that need to be performed for every request that doesn’t have to be duplicated.

## References

[Insiders guide into interceptors and httpclient mechanics in angular](https://indepth.dev/posts/1118/insiders-guide-into-interceptors-and-httpclient-mechanics-in-angular)

[Top 10 ways to use interceptors in angular](https://indepth.dev/posts/1051/top-10-ways-to-use-interceptors-in-angular)

[Http intercepting requests-and responses](https://angular.io/guide/http#intercepting-requests-and-responses)

[How to implement automatic token insertion in requests using HTTP interceptor](https://indepth.dev/tutorials/angular/authentication-token-interceptor)
