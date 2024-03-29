---
title: Angular - handle HTTP errors using interceptors - Angular Tutorials | indepth.dev
author_name: Maria Zayed
author_link: https://twitter.com/mariawzayed
discussion_link:  https://github.com/indepth-dev/community/discussions/203
display_name: Angular - handle HTTP errors using interceptors
---

# **Angular - handle HTTP errors using interceptors**
HTTP errors are common in Angular while making an HTTP request, it’s a good practice to handle these errors without hampering the user’s experience. In this article, we’ll discuss three ways of dealing with such cases.

First things first, let’s understand the error interceptor.

## What is the Error Interceptor?
An error interceptor is a special type of interceptor which is used for handling errors that arise while making an HTTP request. The error may come in either the client-side (browser) or it may be an error from the server-side when the request fails due to any reason. You can learn more about the internal implementation of interceptors in [Insider’s guide into interceptors and HttpClient mechanics in Angular.](https://indepth.dev/posts/1118/insiders-guide-into-interceptors-and-httpclient-mechanics-in-angular)

Let’s see the error interceptor in practice.

## Project Setup
First of all, let’s create a new project using the *ng new <project-name>* command (I named the project http-interceptor).
After creating the project successfully, let's create the needed components & services. Run the following commands:

* ng g c components/demo-component (to create a new component named demo)
* ng g s services/api (to create an api service)
* ng g interceptor interceptors/error-catching (to create an error-catching interceptor)
  After running the above commands you should get a project structure like this.
  

  ![2](https://user-images.githubusercontent.com/27064594/141654079-a5575cdc-cc4f-4830-9a88-2671059aeb41.PNG)
  

Clean the `app.component.html `and add the `<app-demo-component></app-demo-component>` tag in the `demo-component.component.html` add this piece of code:
  
```javascript
<button (click)="getData()">Get Data</button>
<div *ngIf="data">{{data}}</div>
```

Simple enough right! Nothing fancy happening here. Before starting with the fun part, let’s create a function to send an HTTP request.

```javascript
// demo-component.component.ts

import {Component, OnInit} from '@angular/core';
import {ApiService} from "../../services/api.service";

@Component({
selector: 'app-demo-component',
templateUrl: './demo-component.component.html',
styleUrls: ['./demo-component.component.scss']
})
export class DemoComponentComponent implements OnInit {

data = ""

constructor(private apiService: ApiService) {
}

ngOnInit(): void {
}

getData() {
this.apiService.getData().subscribe(res => {
this.data = JSON.stringify(res)
    })
  }
}
```

```javascript
// api.service.ts

import {Injectable} from '@angular/core';
import {HttpClient} from "@angular/common/http";
import {Observable} from "rxjs";

@Injectable({
  providedIn: 'root'
})
export class ApiService {
  url = 'https://jsonplaceholder.typicode.com/todos/10';

  constructor(private http: HttpClient) {
  }

  getData(): Observable<any> {
     return this.http.get(this.url);
  }
}
```

At this point, it’s a typical HTTP request. Now, let’s start with the fun part.

## Use HTTP Interceptor
Now, let’s dive deep into the interesting part. First, let’s take a look at the `error-catching.interceptor.ts` file.

Notice that it implements the HttpInterceptor interface, which forces us to implement the intercept function which identifies and handles a given HTTP request.

What about the intercept function parameters?
* req – The outgoing request object to handle.
* next – The next interceptor in the chain, or the backend if no interceptors remain in the chain.
* Returns: an observable of the event stream.
  
Now let’s add our touch. Take a look at the next code:
  
```javascript
import {Injectable} from '@angular/core';
import {HttpErrorResponse, HttpEvent, HttpHandler, HttpInterceptor, HttpRequest} from '@angular/common/http';
import {Observable, throwError} from 'rxjs';
import {catchError} from "rxjs/operators";

@Injectable()
export class ErrorCatchingInterceptor implements HttpInterceptor {

  constructor() {
  }

  intercept(request: HttpRequest<unknown>, next: HttpHandler): Observable<HttpEvent<unknown>> {
     return next.handle(request)
           .pipe(
                 catchError((error: HttpErrorResponse) => {
                    let errorMsg = '';
                    if (error.error instanceof ErrorEvent) {
                       console.log('This is client side error');
                       errorMsg = `Error: ${error.error.message}`;
                    } else {
                       console.log('This is server side error');
                       errorMsg = `Error Code: ${error.status},  Message: ${error.message}`;
                    }
                    console.log(errorMsg);
                    return throwError(errorMsg);
                 })
           )
  }
}
```

What is happening here? I’ll explain.

Inside the `pipe()` operator of rxjs we are catching any error and checking for the source of it (whether it’s from the client-side or from the server-side). Then we are returning the error through the `throwError()` operator of rxjs.

Now, let’s use the interceptor file in our project. We will add it to the providers array in the `app.module.ts`.

```javascript
@NgModule({
  ...
        providers: [
  {
     provide: HTTP_INTERCEPTORS,
     useClass: ErrorCatchingInterceptor,
     multi: true
  }
],
...
})

export class AppModule {
}
```

Let’s make sure that everything is working as expected.

What is the expected behavior? 

The expected behavior is every request/response should pass through the interceptor.

For the sake of the demo, we will modify the `error-catching.interceptor.ts` file, and add two `console.log()`

```javascript
import {Injectable} from '@angular/core';
import {HttpErrorResponse, HttpEvent, HttpHandler, HttpInterceptor, HttpRequest} from '@angular/common/http';
import {Observable, throwError} from 'rxjs';
import {catchError, map} from "rxjs/operators";

@Injectable()
export class ErrorCatchingInterceptor implements HttpInterceptor {

constructor() {
}

intercept(request: HttpRequest<unknown>, next: HttpHandler): Observable<HttpEvent<unknown>> {
console.log("Passed through the interceptor in request");

     return next.handle(request)
           .pipe(
                 map(res => {
                    console.log("Passed through the interceptor in response");
                    return res
                 }),
                 catchError((error: HttpErrorResponse) => {
                    let errorMsg = '';
                    if (error.error instanceof ErrorEvent) {
                       console.log('This is client side error');
                       errorMsg = `Error: ${error.error.message}`;
                    } else {
                       console.log('This is server side error');
                       errorMsg = `Error Code: ${error.status},  Message: ${error.message}`;
                    }
                    console.log(errorMsg);
                    return throwError(errorMsg);
                 })
           )
}
}
```

You should see the following output:

![3](https://user-images.githubusercontent.com/27064594/141654269-2689e1e6-3753-41ea-ab5a-4d59e2c5b54d.PNG)

Now, let’s break the URL to make the request fail.

```javascript
url = 'https://jsonplaceholder.typicode.com/tod';
```

Now, after running the code with a broken URL you should see the following output

![4](https://user-images.githubusercontent.com/27064594/141654298-d96a40b7-6cd9-4dd5-a896-d8f75af23f2e.PNG)

## Conclusion
HttpInterceptors is a great tool for modern applications, it has great benefits. Following are a few of them:
* Code reusability — since we are expected to perform a certain common set of steps against a majority of the HTTP requests and responses, we can write the logic in one place and enable the request and response flow to use that logic. This way, instead of replicating the same logic individually on a service level, we write that once and hence it produces a lot less redundant code which is also a clean practice to follow.
* Scalability — this point is, kind of, linked to the first point in a way that it provides an extension to the idea stated earlier. If the code is reusable then it automatically makes sense that it will be a lot easier to scale the code and its design.
* Better service layer — since we can handle the related activities using interceptors, we do not need to pollute the service layer with handling that other stuff. This way, the service layer will be clean, to the point, and will only do what it is responsible for doing.

## References

[How to split http interceptors between multiple backends](https://indepth.dev/posts/1455/how-to-split-http-interceptors-between-multiple-backends)

[Insiders guide into interceptors and httpclient mechanics in angular](https://indepth.dev/posts/1118/insiders-guide-into-interceptors-and-httpclient-mechanics-in-angular)

[Top 10 ways to use interceptors in angular](https://indepth.dev/posts/1051/top-10-ways-to-use-interceptors-in-angular)

[Http intercepting requests-and responses](https://angular.io/guide/http#intercepting-requests-and-responses)

[How to implement automatic token insertion in requests using HTTP interceptor](https://indepth.dev/tutorials/angular/authentication-token-interceptor)
