---
title: Authentication using the Amazon Cognito to an Angular application - Angular Tutorials | indepth.dev
author_name: Rodrigo Kamada
author_link: https://twitter.com/rodrigokamada
discussion_link:  https://github.com/indepth-dev/community/discussions/246
display_name: Authentication using the Amazon Cognito to an Angular application
---



# Authentication using the Amazon Cognito to an Angular application



## Introduction

[Angular](https://angular.io/) is a development platform for building WEB, mobile and desktop applications using HTML, CSS and TypeScript (JavaScript). Currently, Angular is at version 13 and Google is the main maintainer of the project.

[Amazon Cognito](https://aws.amazon.com/cognito/) is a simple and secure authentication service that supports user sign in, sign up and control in a WEB or mobile application.



## Prerequisites


Before you start, you need to install and configure the tools:

* [git](https://git-scm.com/)
* [Node.js and npm](https://nodejs.org/)
* [Angular CLI](https://angular.io/cli)
* IDE (e.g. [Visual Studio Code](https://code.visualstudio.com/))



## Getting started


### Create and configure the account on the Amazon Cognito

**1.** Let's create the account. Access the site [https://aws.amazon.com/cognito/](https://aws.amazon.com/cognito/) and click on the button *Sign up now*.

![Amazon Cognito - Home page](https://res.cloudinary.com/rodrigokamada/image/upload/v1642283628/Blog/angular-cognito/cognito-step1.png)

**2.** Click on the button *Create a new AWS account*.

![Amazon Cognito - Sign in](https://res.cloudinary.com/rodrigokamada/image/upload/v1642283894/Blog/angular-cognito/cognito-step2.png)

**3.** Fill in the fields *Email Address*, *Password*, *Confirm password*, *AWS account name*, *Security check* and click on the button *Continue (step 1 of 5)*.

![Amazon Cognito - Sign up for AWS (step 1 of 5)](https://res.cloudinary.com/rodrigokamada/image/upload/v1642288229/Blog/angular-cognito/cognito-step3.png)

**4.** Click on the option *Personal - for your own projects*, fill in the fields *Full Name*, *Phone Number*, *Country or Region*, *Address*, *City*, *State, Province or Region*, *Postal Code*, click on the checkbox *I have read and agree to the terms of the AWS Customer Agreement.* and click on the button *Continue (step 2 of 5)*.

![Amazon Cognito - Sign up for AWS (step 2 of 5)](https://res.cloudinary.com/rodrigokamada/image/upload/v1642298952/Blog/angular-cognito/cognito-step4.png)

**5.** Fill in the fields *Credit card number*, *Expiration date*, *Cardholder's name*, click on the optiond *Use my contact address*, *CPF*, fill in the fields *Tax Registration Number*, *Date of birth*, *Postal Code*, *Neighbothood*, *Tax address* and click on the button *Verify and Continue (step 3 of 5)*.

![Amazon Cognito - Sign up for AWS (step 3 of 5)](https://res.cloudinary.com/rodrigokamada/image/upload/v1642348218/Blog/angular-cognito/cognito-step5.png)

**Note:**

* Some fields may be different for your country.

**6.** Click on the option *Text message (SMS)*, select an option in the field *Country or region code*, fill in the fields *Mobile phone number*, *Type the characters as shown above* and click on the button *Send SMS (step 4 of 5)*.

![Amazon Cognito - Sign up for AWS (step 4 of 5)](https://res.cloudinary.com/rodrigokamada/image/upload/v1642368061/Blog/angular-cognito/cognito-step6.png)

**7.** Fill in the field *Verify code* and click on the button *Continue (step 4 of 5)*.

![Amazon Cognito - Sign up for AWS (continue step 4 of 5)](https://res.cloudinary.com/rodrigokamada/image/upload/v1642368525/Blog/angular-cognito/cognito-step7.png)

**8.** Click on the option *Basic support - Free* and click on the button *Complete sign up*.

![Amazon Cognito - Sign up for AWS (complete sign up)](https://res.cloudinary.com/rodrigokamada/image/upload/v1642368861/Blog/angular-cognito/cognito-step8.png)

**9.** Click on the button *Go to the AWS Management Console*.

![Amazon Cognito - Congratulations](https://res.cloudinary.com/rodrigokamada/image/upload/v1642368989/Blog/angular-cognito/cognito-step9.png)

**10.** Click on the option *Root user*, fill in the field *Root user email address* and click on the button *Next*.

![Amazon Cognito - Sign in](https://res.cloudinary.com/rodrigokamada/image/upload/v1642369063/Blog/angular-cognito/cognito-step10.png)

**11.** Fill in the field *Security check* and click on the button *Submit*.

![Amazon Cognito - Security check](https://res.cloudinary.com/rodrigokamada/image/upload/v1642369401/Blog/angular-cognito/cognito-step11.png)

**12.** Fill in the field *Password* and click on the button *Sign in*.

![Amazon Cognito - Root user sign in](https://res.cloudinary.com/rodrigokamada/image/upload/v1642370125/Blog/angular-cognito/cognito-step12.png)

**13.** Click on the button *Switch to the new Console Home*.

![Amazon Cognito - New AWS Console Home](https://res.cloudinary.com/rodrigokamada/image/upload/v1642370137/Blog/angular-cognito/cognito-step13.png)

**14.** Click on the menu *Security, Identity & Compliance* and on the submenu *Cognito*.

![Amazon Cognito - Menu Cognito](https://res.cloudinary.com/rodrigokamada/image/upload/v1642418597/Blog/angular-cognito/cognito-step14.png)

**15.** Click on the link *Try out the new interface*.

![Amazon Cognito - Amazon Cognito old](https://res.cloudinary.com/rodrigokamada/image/upload/v1642419871/Blog/angular-cognito/cognito-step15.png)

**16.** Click on the button *Create user pool*.

![Amazon Cognito - Amazon Cognito new](https://res.cloudinary.com/rodrigokamada/image/upload/v1642420048/Blog/angular-cognito/cognito-step16.png)

**17.** Click on the option *Email* and click on the button *Next*.

![Amazon Cognito - Configure sign-in experience](https://res.cloudinary.com/rodrigokamada/image/upload/v1642420365/Blog/angular-cognito/cognito-step17.png)

**18.** Click on the options *Cognito defaults*, *No MFA*, *Enable self-service account recovery - Recommended*, *Email only* and click on the button *Next*.

![Amazon Cognito - Configure security requirements](https://res.cloudinary.com/rodrigokamada/image/upload/v1642421044/Blog/angular-cognito/cognito-step18.png)

**19.** Click on the options *Enable self-registration*, *Allow Cognito to automatically send messages to verify and confirm - Recommended*, *Send email message, verify email address* and click on the button *Next*.

![Amazon Cognito - Configure sign-up experience](https://res.cloudinary.com/rodrigokamada/image/upload/v1642425488/Blog/angular-cognito/cognito-step19.png)

**20.** Click on the option *Send email with Cognito* and click on the button *Next*.

![Amazon Cognito - Configure message delivery](https://res.cloudinary.com/rodrigokamada/image/upload/v1642425858/Blog/angular-cognito/cognito-step20.png)

**21.** Fill in the field *User pool name*, click on the option *Public client*, fill in the field *App client name*, click on the option *Don't generate a client secret* and click on the button *Sign in*.

![Amazon Cognito - Integrate your app](https://res.cloudinary.com/rodrigokamada/image/upload/v1642427958/Blog/angular-cognito/cognito-step21.png)

**22.** Click on the button *Create user pool*.

![Amazon Cognito - Review and create](https://res.cloudinary.com/rodrigokamada/image/upload/v1642428754/Blog/angular-cognito/cognito-step22.png)

**23.** Click on the link *angular-cognito* with the user pool name.

![Amazon Cognito - User pools](https://res.cloudinary.com/rodrigokamada/image/upload/v1642473465/Blog/angular-cognito/cognito-step23.png)

**24.** Copy the user pool ID displayed, in my case, the ID `us-east-2_pptCj2gqV` was displayed because this ID will be configured in the Angular application and click on the link *App integration*.

![Amazon Cognito - User pool overview](https://res.cloudinary.com/rodrigokamada/image/upload/v1642474448/Blog/angular-cognito/cognito-step24.png)

**25.** Copy the client ID displayed, in my case, the ID `1452opnjll0ldmocs201b1oimu` was displayed because this ID will be configured in the Angular application.

![Amazon Cognito - App integration](https://res.cloudinary.com/rodrigokamada/image/upload/v1642474595/Blog/angular-cognito/cognito-step25.png)

**26.** Ready! Account created, user pool ID and client ID generated.



### Create the Angular application


**1.** Let's create the application with the Angular base structure using the `@angular/cli` with the route file and the SCSS style format.

```powershell
ng new angular-cognito --routing true --style scss
CREATE angular-cognito/README.md (1060 bytes)
CREATE angular-cognito/.editorconfig (274 bytes)
CREATE angular-cognito/.gitignore (548 bytes)
CREATE angular-cognito/angular.json (3261 bytes)
CREATE angular-cognito/package.json (1079 bytes)
CREATE angular-cognito/tsconfig.json (863 bytes)
CREATE angular-cognito/.browserslistrc (600 bytes)
CREATE angular-cognito/karma.conf.js (1432 bytes)
CREATE angular-cognito/tsconfig.app.json (287 bytes)
CREATE angular-cognito/tsconfig.spec.json (333 bytes)
CREATE angular-cognito/.vscode/extensions.json (130 bytes)
CREATE angular-cognito/.vscode/launch.json (474 bytes)
CREATE angular-cognito/.vscode/tasks.json (938 bytes)
CREATE angular-cognito/src/favicon.ico (948 bytes)
CREATE angular-cognito/src/index.html (300 bytes)
CREATE angular-cognito/src/main.ts (372 bytes)
CREATE angular-cognito/src/polyfills.ts (2338 bytes)
CREATE angular-cognito/src/styles.scss (80 bytes)
CREATE angular-cognito/src/test.ts (745 bytes)
CREATE angular-cognito/src/assets/.gitkeep (0 bytes)
CREATE angular-cognito/src/environments/environment.prod.ts (51 bytes)
CREATE angular-cognito/src/environments/environment.ts (658 bytes)
CREATE angular-cognito/src/app/app-routing.module.ts (245 bytes)
CREATE angular-cognito/src/app/app.module.ts (393 bytes)
CREATE angular-cognito/src/app/app.component.scss (0 bytes)
CREATE angular-cognito/src/app/app.component.html (23364 bytes)
CREATE angular-cognito/src/app/app.component.spec.ts (1100 bytes)
CREATE angular-cognito/src/app/app.component.ts (220 bytes)
✔ Packages installed successfully.
    Successfully initialized git.
```

**2.** Install and configure the Bootstrap CSS framework. Do steps 2 and 3 of the post *[Adding the Bootstrap CSS framework to an Angular application](https://github.com/rodrigokamada/angular-bootstrap)*.

**3.** Configure the variable `cognito.userPoolId` with the Amazon Cognito User Pool ID and the variable `cognito.userPoolWebClientId` with the Amazon Cognito WEB Client ID in the `src/environments/environment.ts` and `src/environments/environment.prod.ts` files as below.

```typescript
cognito: {
  userPoolId: 'us-east-2_pptCj2gqV',
  userPoolWebClientId: '1452opnjll0ldmocs201b1oimu',
},
```

**4.** Install the `aws-amplify` library.

```powershell
npm install aws-amplify
```

**5.** Change the `src/polyfills.ts` file. Add the global declaration as below. This configuration is required starting with Angular version 6.

```typescript
(window as any).global = window;
```

**6.** Create the `CognitoService` service.

```powershell
ng generate service cognito --skip-tests=true
CREATE src/app/cognito.service.ts (136 bytes)
```

**7.** Change the `src/app/cognito.service.ts` file and add the lines as below.

```typescript
import { Injectable } from '@angular/core';
import { BehaviorSubject } from 'rxjs';
import Amplify, { Auth } from 'aws-amplify';

import { environment } from '../environments/environment';

export interface IUser {
  email: string;
  password: string;
  showPassword: boolean;
  code: string;
  name: string;
}

@Injectable({
  providedIn: 'root',
})
export class CognitoService {

  private authenticationSubject: BehaviorSubject<any>;

  constructor() {
    Amplify.configure({
      Auth: environment.cognito,
    });

    this.authenticationSubject = new BehaviorSubject<boolean>(false);
  }

  public signUp(user: IUser): Promise<any> {
    return Auth.signUp({
      username: user.email,
      password: user.password,
    });
  }

  public confirmSignUp(user: IUser): Promise<any> {
    return Auth.confirmSignUp(user.email, user.code);
  }

  public signIn(user: IUser): Promise<any> {
    return Auth.signIn(user.email, user.password)
    .then(() => {
      this.authenticationSubject.next(true);
    });
  }

  public signOut(): Promise<any> {
    return Auth.signOut()
    .then(() => {
      this.authenticationSubject.next(false);
    });
  }

  public isAuthenticated(): boolean {
    return this.authenticationSubject.value;
  }

  public getUser(): Promise<any> {
    return Auth.currentUserInfo();
  }

  public updateUser(user: IUser): Promise<any> {
    return Auth.currentUserPoolUser()
    .then((cognitoUser: any) => {
      return Auth.updateUserAttributes(cognitoUser, user);
    });
  }

}
```

**8.** Create the `SignUpComponent` component.

```powershell
ng generate component sign-up --skip-tests=true
CREATE src/app/sign-up/sign-up.component.scss (0 bytes)
CREATE src/app/sign-up/sign-up.component.html (22 bytes)
CREATE src/app/sign-up/sign-up.component.ts (279 bytes)
UPDATE src/app/app.module.ts (638 bytes)
```

**9.** Change the `src/app/sign-up/sign-up.component.ts` file. Import the `Router` and `CognitoService` services and create the `signUp` and `confirmSignUp` methods as below.

```typescript
import { Component } from '@angular/core';
import { Router } from '@angular/router';

import { IUser, CognitoService } from '../cognito.service';

@Component({
  selector: 'app-sign-up',
  templateUrl: './sign-up.component.html',
  styleUrls: ['./sign-up.component.scss'],
})
export class SignUpComponent {

  loading: boolean;
  isConfirm: boolean;
  user: IUser;

  constructor(private router: Router,
              private cognitoService: CognitoService) {
    this.loading = false;
    this.isConfirm = false;
    this.user = {} as IUser;
  }

  public signUp(): void {
    this.loading = true;
    this.cognitoService.signUp(this.user)
    .then(() => {
      this.loading = false;
      this.isConfirm = true;
    }).catch(() => {
      this.loading = false;
    });
  }

  public confirmSignUp(): void {
    this.loading = true;
    this.cognitoService.confirmSignUp(this.user)
    .then(() => {
      this.router.navigate(['/signIn']);
    }).catch(() => {
      this.loading = false;
    });
  }

}
```

**10.** Change the `src/app/sign-up/sign-up.component.html` file. Add the lines as below.

```html
<div class="row justify-content-center my-5">
  <div class="col-4">
    <div class="card">
      <div class="card-body" *ngIf="!isConfirm">
        <div class="row">
          <div class="col mb-2">
            <label for="email" class="form-label">Email:</label>
            <input type="email" id="email" name="email" #email="ngModel" [(ngModel)]="user.email" class="form-control form-control-sm">
          </div>
        </div>
        <div class="row">
          <div class="col mb-2">
            <label for="password" class="form-label">Password:</label>
            <div class="input-group input-group-sm">
              <input [type]="user.showPassword ? 'text' : 'password'" id="password" name="password" #password="ngModel" [(ngModel)]="user.password" class="form-control form-control-sm">
              <button type="button" class="btn btn-outline-secondary" (click)="user.showPassword = !user.showPassword">
                <i class="bi" [ngClass]="{'bi-eye-fill': !user.showPassword, 'bi-eye-slash-fill': user.showPassword}"></i>
              </button>
            </div>
          </div>
        </div>
        <div class="row">
          <div class="col d-grid">
            <button type="button" (click)="signUp()" class="btn btn-sm btn-success" [disabled]="loading">
              <span class="spinner-border spinner-border-sm" role="status" aria-hidden="true" *ngIf="loading"></span>
              Sign up
            </button>
          </div>
        </div>
      </div>
      <div class="card-body" *ngIf="isConfirm">
        <div class="row">
          <div class="col mb-2">
            <label for="code" class="form-label">Code:</label>
            <input type="text" id="code" name="code" #code="ngModel" [(ngModel)]="user.code" class="form-control form-control-sm">
          </div>
        </div>
        <div class="row">
          <div class="col d-grid">
            <button type="button" (click)="confirmSignUp()" class="btn btn-sm btn-success" [disabled]="loading">
              <span class="spinner-border spinner-border-sm" role="status" aria-hidden="true" *ngIf="loading"></span>
              Confirm
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</div>
```

**11.** Create the `SignInComponent` component.

```powershell
ng generate component sign-in --skip-tests=true
CREATE src/app/sign-in/sign-in.component.scss (0 bytes)
CREATE src/app/sign-in/sign-in.component.html (22 bytes)
CREATE src/app/sign-in/sign-in.component.ts (279 bytes)
UPDATE src/app/app.module.ts (490 bytes)
```

**12.** Change the `src/app/sign-in/sign-in.component.ts` file. Import the `Router` and `CognitoService` service and create the `signIn` method as below.

```typescript
import { Component } from '@angular/core';
import { Router } from '@angular/router';

import { IUser, CognitoService } from '../cognito.service';

@Component({
  selector: 'app-sign-in',
  templateUrl: './sign-in.component.html',
  styleUrls: ['./sign-in.component.scss'],
})
export class SignInComponent {

  loading: boolean;
  user: IUser;

  constructor(private router: Router,
              private cognitoService: CognitoService) {
    this.loading = false;
    this.user = {} as IUser;
  }

  public signIn(): void {
    this.loading = true;
    this.cognitoService.signIn(this.user)
    .then(() => {
      this.router.navigate(['/profile']);
    }).catch(() => {
      this.loading = false;
    });
  }

}
```

**13.** Change the `src/app/sign-in/sign-in.component.html` file. Add the lines as below.

```html
<div class="row justify-content-center my-5">
  <div class="col-4">
    <div class="card">
      <div class="card-body">
        <div class="row">
          <div class="col mb-2">
            <label for="email" class="form-label">Email:</label>
            <input type="email" id="email" name="email" #email="ngModel" [(ngModel)]="user.email" class="form-control form-control-sm">
          </div>
        </div>
        <div class="row">
          <div class="col mb-2">
            <label for="password" class="form-label">Password:</label>
            <div class="input-group input-group-sm">
              <input [type]="user.showPassword ? 'text' : 'password'" id="password" name="password" #password="ngModel" [(ngModel)]="user.password" class="form-control form-control-sm">
              <button type="button" class="btn btn-outline-secondary" (click)="user.showPassword = !user.showPassword">
                <i class="bi" [ngClass]="{'bi-eye-fill': !user.showPassword, 'bi-eye-slash-fill': user.showPassword}"></i>
              </button>
            </div>
          </div>
        </div>
        <div class="row">
          <div class="col d-grid">
            <button type="button" (click)="signIn()" class="btn btn-sm btn-success" [disabled]="loading">
              <span class="spinner-border spinner-border-sm" role="status" aria-hidden="true" *ngIf="loading"></span>
              Sign in
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</div>
```

**14.** Create the `ProfileComponent` component.

```powershell
ng generate component profile --skip-tests=true
CREATE src/app/profile/profile.component.scss (0 bytes)
CREATE src/app/profile/profile.component.html (22 bytes)
CREATE src/app/profile/profile.component.ts (280 bytes)
UPDATE src/app/app.module.ts (726 bytes)
```

**15.** Change the `src/app/profile/profile.component.ts` file. Import the `CognitoService` service and create the `update` method as below.

```typescript
import { Component, OnInit } from '@angular/core';

import { IUser, CognitoService } from '../cognito.service';

@Component({
  selector: 'app-profile',
  templateUrl: './profile.component.html',
  styleUrls: ['./profile.component.scss'],
})
export class ProfileComponent implements OnInit {

  loading: boolean;
  user: IUser;

  constructor(private cognitoService: CognitoService) {
    this.loading = false;
    this.user = {} as IUser;
  }

  public ngOnInit(): void {
    this.cognitoService.getUser()
    .then((user: any) => {
      this.user = user.attributes;
    });
  }

  public update(): void {
    this.loading = true;

    this.cognitoService.updateUser(this.user)
    .then(() => {
      this.loading = false;
    }).catch(() => {
      this.loading = false;
    });
  }

}
```

**16.** Change the `src/app/profile/profile.component.html` file. Add the lines as below.

```html
<div class="row justify-content-center my-5">
  <div class="col-4">
    <div class="row">
      <div class="col mb-2">
        <label for="email" class="form-label">Email:</label>
        <input type="email" id="email" name="email" #email="ngModel" [ngModel]="user.email" disabled class="form-control form-control-sm">
      </div>
    </div>
    <div class="row">
      <div class="col mb-2">
        <label for="name" class="form-label">Name:</label>
        <input type="text" id="name" name="name" #name="ngModel" [(ngModel)]="user.name" class="form-control form-control-sm">
      </div>
    </div>
    <div class="row">
      <div class="col d-grid">
        <button type="button" (click)="update()" class="btn btn-sm btn-dark" [disabled]="loading">
          <span class="spinner-border spinner-border-sm" role="status" aria-hidden="true" *ngIf="loading"></span>
          Update
        </button>
      </div>
    </div>
  </div>
</div>
```

**17.** Change the `src/app/app.component.ts` file. Import the `Router` and `CognitoService` services and create the `isAuthenticated` and `signOut` methods as below.

```typescript
import { Component } from '@angular/core';
import { Router } from '@angular/router';

import { CognitoService } from './cognito.service';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.scss'],
})
export class AppComponent {

  constructor(private router: Router,
              private cognitoService: CognitoService) {
  }

  public isAuthenticated(): boolean {
    return this.cognitoService.isAuthenticated();
  }

  public signOut(): void {
    this.cognitoService.signOut()
    .then(() => {
      this.router.navigate(['/signIn']);
    });
  }

}
```

**18.** Change the `src/app/app.component.html` file and add the menu as below.

```html
<nav class="navbar navbar-expand-sm navbar-light bg-light">
  <div class="container-fluid">
    <a class="navbar-brand" href="#">Angular Amazon Cognito</a>

    <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation">
      <span class="navbar-toggler-icon"></span>
    </button>

    <div id="navbarContent" class="collapse navbar-collapse">
      <ul class="navbar-nav me-auto mb-2 mb-lg-0">
        <li class="nav-item">
          <a class="nav-link" routerLink="/signUp" routerLinkActive="active" *ngIf="!isAuthenticated()">Sign up</a>
        </li>
        <li class="nav-item">
          <a class="nav-link" routerLink="/signIn" routerLinkActive="active" *ngIf="!isAuthenticated()">Sign in</a>
        </li>
        <li class="nav-item">
          <a class="nav-link" routerLink="/profile" routerLinkActive="active" *ngIf="isAuthenticated()">Profile</a>
        </li>
        <li class="nav-item">
          <a class="nav-link" routerLink="" (click)="signOut()" *ngIf="isAuthenticated()">Sign out</a>
        </li>
      </ul>
    </div>
  </div>
</nav>

<router-outlet></router-outlet>
```

**19.** Change the `src/app/app-routing.module.ts` file and add the routes as below.

```typescript
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';

import { ProfileComponent } from './profile/profile.component';
import { SignInComponent } from './sign-in/sign-in.component';
import { SignUpComponent } from './sign-up/sign-up.component';

const routes: Routes = [
  {
    path: '',
    redirectTo: 'signIn',
    pathMatch: 'full',
  },
  {
    path: 'profile',
    component: ProfileComponent,
  },
  {
    path: 'signIn',
    component: SignInComponent,
  },
  {
    path: 'signUp',
    component: SignUpComponent,
  },
  {
    path: '**',
    redirectTo: 'signIn',
  },
];

@NgModule({
  imports: [
    RouterModule.forRoot(routes),
  ],
  exports: [
    RouterModule,
  ],
})
export class AppRoutingModule {
}
```

**20.** Change the `src/app/app.module.ts` file. Import the `FormsModule` module, the `ProfileComponent`, `SignInComponent`, `SignUpComponent` components as below.

```typescript
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { FormsModule } from '@angular/forms';

import { AppRoutingModule } from './app-routing.module';

import { AppComponent } from './app.component';
import { ProfileComponent } from './profile/profile.component';
import { SignInComponent } from './sign-in/sign-in.component';
import { SignUpComponent } from './sign-up/sign-up.component';

@NgModule({
  declarations: [
    AppComponent,
    ProfileComponent,
    SignInComponent,
    SignUpComponent,
  ],
  imports: [
    BrowserModule,
    FormsModule,
    AppRoutingModule,
  ],
  providers: [
  ],
  bootstrap: [
    AppComponent,
  ],
})
export class AppModule {
}
```

**21.** Run the application with the command below.

```powershell
npm start

> angular-cognito@1.0.0 start
> ng serve

✔ Browser application bundle generation complete.

Initial Chunk Files   | Names         |  Raw Size
vendor.js             | vendor        |   9.98 MB | 
styles.css, styles.js | styles        | 486.91 kB | 
polyfills.js          | polyfills     | 339.28 kB | 
scripts.js            | scripts       |  76.33 kB | 
main.js               | main          |  47.13 kB | 
runtime.js            | runtime       |   6.87 kB | 

                      | Initial Total |  10.91 MB

Build at: 2022-01-18T16:33:13.971Z - Hash: ce7a03498c95d4f5 - Time: 25230ms

** Angular Live Development Server is listening on localhost:4200, open your browser on http://localhost:4200/ **


✔ Compiled successfully.
```

**Note:**

* It may display some warnings when running the application. 

**22.** Ready! Access the URL `http://localhost:4200/` and check if the application is working. See the application working on [GitHub Pages](https://rodrigokamada.github.io/angular-amazon-cognito/) and [Stackblitz](https://stackblitz.com/edit/angular13-amazon-cognito).

![Angular Amazon Cognito](https://res.cloudinary.com/rodrigokamada/image/upload/v1642523631/Blog/angular-cognito/angular-cognito.png)



### Testing the application

**1.** Let's test the application. Access the URL `http://localhost:4200/`, fill in the field *Email*, *Password* and click on the button *Sign up*.

![Application - Sign up](https://res.cloudinary.com/rodrigokamada/image/upload/v1642530554/Blog/angular-cognito/application-step1.png)

**2.** Check that the user was created in Amazon Cognito.

![Application - User unconfirmed](https://res.cloudinary.com/rodrigokamada/image/upload/v1642532413/Blog/angular-cognito/application-step2.png)

**3.** Open the email with the subject *Your verification code*  and copy the code generated, in my case, the code `308386` was generated.

![Application - Your verification code](https://res.cloudinary.com/rodrigokamada/image/upload/v1642539595/Blog/angular-cognito/application-step3.png)

**4.** Fill in the field *Code* with the code copied and click on the button *Confirm*.

![Application - Confirm code](https://res.cloudinary.com/rodrigokamada/image/upload/v1642539853/Blog/angular-cognito/application-step4.png)

**5.** Check that the user was confirmed in Amazon Cognito.

![Application - User confirmed](https://res.cloudinary.com/rodrigokamada/image/upload/v1642540005/Blog/angular-cognito/application-step5.png)

**6.** Fill in the field *Email*, *Password* and click on the button *Sign in*.

![Application - Sign in](https://res.cloudinary.com/rodrigokamada/image/upload/v1642540122/Blog/angular-cognito/application-step6.png)

**7.** Fill in the field *Name* and click on the button *Update*.

![Application - Update user](https://res.cloudinary.com/rodrigokamada/image/upload/v1642540298/Blog/angular-cognito/application-step7.png)

**8.** Click on the user link created in Amazon Cognito.

![Application - Users](https://res.cloudinary.com/rodrigokamada/image/upload/v1642540443/Blog/angular-cognito/application-step8.png)

**9.** Check that the user name was updated in Amazon Cognito.

![Application - User updated](https://res.cloudinary.com/rodrigokamada/image/upload/v1642540632/Blog/angular-cognito/application-step9.png)

**10.** Ready! We test the user sign in, sign up and update.



The application repository is available at [https://github.com/rodrigokamada/angular-amazon-cognito](https://github.com/rodrigokamada/angular-amazon-cognito).

This tutorial was posted on my [blog](https://rodrigo.kamada.com.br/blog/autenticacao-usando-o-amazon-cognito-em-uma-aplicacao-angular) in portuguese.
