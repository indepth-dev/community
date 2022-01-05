---
title: The simple way to reload data using RxJS
author_name: Oleh Baranovskyi
author_link: https://obaranovskyi.medium.com/
discussion_link: https://github.com/indepth-dev/community/discussions/244
display_name: The simple way to reload data using RxJS
---

# **The simple way to reload data using RxJS**

Most of the time, we have to load data from the server. To perform the action client usually sends requests along with predefined data. Such data usually takes from the route, browser storage, or from attributes in the case when it's a component. To load the user details we need to have userId, to load card details we need to have cardId, and so on. But what if you already load the data and you just need to reload without passing the same predefined data again and again. Sounds like a trivial task, right?

It depends.

- If we want to stay reactive and write declarative code.
- If we don't want to create new variables which responsibility will be to keep predefined data only for data reloading.
- If we want to deal with reusable code.
- If we want to avoid boilerplate as much as possible.
- If the code should stay simple.

_Then I would stick with no._

I've noticed that every project that I was working on had two and more distinct solutions. Our mission will be to develop data reload pattern using the RxJS library and do it well.

If you're interested in the approach we came up with, stay tuned!

## Initial code without any data reload implementation

```typescript
import { Observable, of, ReplaySubject } from 'rxjs';
import { switchMap } from 'rxjs/operators';

function Identity<T>(value: T): T {
  return value;
}

interface User {
  id: number;
  name: string;
}

class UserMockWebService {
  readonly users: User[] = [
    { id: 1, name: 'John' },
    { id: 2, name: 'Liza' },
    { id: 3, name: 'Suzy' }
  ];

  getUserById(id: number): Observable<User> {
    const user = this.users.find((user: User) => {
      return user.id === id;
    });

    return of(user) as Observable<User>;
  }
}
```
- `Identity` - is going to be responsible for returning the value that it got.
- `User` - model which we would manipulate with.
- `UserMockWebService` - mock service that we would use as a web service.

```typescript
class UserService {
  private idRplSubj = new ReplaySubject<number>(1);

  userObs$: Observable<User> = this.idRplSubj
    .pipe(
      switchMap((userId: number) => {
        return this.userWebService.getUserById(userId);
      })
    );

    constructor(private userWebService: UserMockWebService) {}

    setId(id: number): void {
      this.idRplSubj.next(id);
    }
}


// Demo
const userWebService = new UserMockWebService();
const userService = new UserService(userWebService);

userService.userObs$.subscribe(console.log);

userService.setId(2);
userService.setId(3);
```

`UserService` - is responsible for providing data which in turn gathered from the server whenever `userId` changes.

At the end of the code, you can see the demo section, which will output into the following:
```
{ id: 2, name: "Liza" }
{ id: 3, name: "Suzi" }
```

## Initial data reload - the draft 

Now we're ready to implement data reload functionality. Let's meet the `scan` operator.

```typescript
class UserService {
  private reloadSubj = new Subject<void>();
  private idRplSubj = new ReplaySubject<number>(1);

  userObs$: Observable<User> = 
    merge(
      this.idRplSubj,
      this.reloadSubj
    )
    .pipe(
      scan((oldValue, currentValue) => {
        if(!oldValue && !currentValue) 
          throw new Error(`Reload can't run before initial load`);

        return currentValue || oldValue;
      }),
      switchMap((userId: number) => {
        return this.userWebService.getUserById(userId);
      })
    );

    constructor(private userWebService: UserMockWebService) {}

    setId(id: number): void {
      this.idRplSubj.next(id);
    }

    reload(): void {
      this.reloadSubj.next();
    }
}
view raw
```
Let's take a look at it in action:

```typescript
const userWebService = new UserMockWebService();
const userService = new UserService(userWebService);

userService.userObs$.subscribe(console.log);

userService.setId(2);
userService.setId(3);
userService.reload();
userService.setId(1);
```

```
{ id: 2, name: "Liza" }
{ id: 3, name: "Suzi" }
{ id: 3, name: "Suzi" }
{ id: 1, name: "John" }
```
Everything works! However, here we implemented it for loading users, but we need to load not only the users but the card details as well. In this case, we would need to include the same `scan` code boilerplate again and again and that is not what we would want to do. Let's try to make the code more generic.

## Adding custom `reload` operator

We need to extract the `scan` operator to avoid code duplication:

```typescript
function reload(selector: Function = Identity) {
  return scan((oldValue, currentValue) => {
    if(!oldValue && !currentValue) 
      throw new Error(`Reload can't run before initial load`);

    return selector(currentValue || oldValue);
  });
}
```

Now, instead of the scan we can use one line reload operator and it will take only one line:

```typescript
class UserService {
  private reloadSubj = new Subject<void>();
  private idRplSubj = new ReplaySubject<number>(1);

  userObs$: Observable<User> = 
    merge(
      this.idRplSubj,
      this.reloadSubj
    )
    .pipe(
      reload(),
      switchMap((userId: number) => {
        return this.userWebService.getUserById(userId);
      })
    );

    constructor(private userWebService: UserMockWebService) {}

    setId(id: number): void {
      this.idRplSubj.next(id);
    }

    reload(): void {
      this.reloadSubj.next();
    }
}
```

It's definitely looking better! However, there is still a code boilerplate that we need to remember. We need to have `merge()` with `this.reload$` and `reload()` operator in the pipe whenever we need to reload the data. The good news is that there is a solution.

## Adding the `combineReload` factory function


In order to avoid boilerplate code from the previous example, we are going to use the factory function called `combineReload()`. This function will encapsulate everything for us. Here is how it looks:

```typescript
function combineReload<T>(
  value$: Observable<T>,
  reload$: Observable<void>,
  selector: Function = Identity
): Observable<T> {
  return merge(value$, reload$).pipe(
    reload(selector),
    map((value: any) => value as T)
  );
}
```

Now we can remove the `reload()` operator and just use our `combineReload()` factory function.

```typescript
class UserService {
  private reloadSubj = new Subject<void>();
  private idRplSubj = new ReplaySubject<number>(1);

  userObs$: Observable<User> = 
    combineReload(
      this.idRplSubj,
      this.reloadSubj
    )
    .pipe(
      switchMap((userId: number) => {
        return this.userWebService.getUserById(userId);
      })
    );

  constructor(private userWebService: UserMockWebService) {}

  setId(id: number): void {
    this.idRplSubj.next(id);
  }

  reload(): void {
    this.reloadSubj.next();
  }
}
```

Now it looks clean and neat. Moreover, it's reusable and simple to use!

