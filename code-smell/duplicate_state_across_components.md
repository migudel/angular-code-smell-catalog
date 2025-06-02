# Duplicate State Across Components

## Description

This code smell occurs when multiple components maintain their own local copies of the same state or data, instead of relying on a single source of truth. While duplicating state may seem like a quick solution, it becomes increasingly unsustainable as the application grows, leading to inconsistencies, redundancy, and scattered logic.

Common examples include duplicating user data, session status, active filters, or search results across components. This approach violates the DRY (Don't Repeat Yourself) principle and complicates system maintenance.

The most appropriate way to address this issue is to adopt a centralized and reactive state management strategy. The main alternatives are:

- **RxJS**: Suitable for small or medium-sized applications. Using `BehaviorSubject` in injectable services enables lightweight, encapsulated, and reactive stores without relying on external libraries.
- **NgRx**: Recommended for large-scale applications with complex business logic and multiple sources of state. Implements Redux with a unidirectional data flow, action traceability, and support for asynchronous effects.
- **Akita**: A modern and more ergonomic alternative to NgRx, focused on developer productivity. Particularly useful for projects needing a powerful solution with a smaller learning curve.

## Why This Is a Code Smell

- **State desynchronization**: Components may display inconsistent or outdated data.
- **Increased complexity**: Extra logic is required to keep duplicated states in sync.
- **Harder maintenance**: Any update to the shared state must be manually replicated in all copies.
- **Poor scalability**: As the application grows, so does the likelihood of state-related bugs.
- **Violates the single source of truth principle**: State should have one authoritative source.

---

## Non-Compliant Code Example

```ts
@Component({...})
export class ComponentA {
  user = { name: 'Ana', loggedIn: true };
}

@Component({...})
export class ComponentB {
  // Duplicated and independent state
  user = { name: 'Ana', loggedIn: true }; 
}
```

---

## Compliant Code Example

### RxJS

```ts
// user-store.service.ts
import { Injectable } from '@angular/core';
import { BehaviorSubject } from 'rxjs';
import { User } from './user.model';

@Injectable({ providedIn: 'root' })
export class UserStore {
  private userSubject = new BehaviorSubject<User | null>(null);
  readonly user$ = this.userSubject.asObservable();

  setUser(user: User) {
    this.userSubject.next(user);
  }

  getUserSnapshot(): User | null {
    return this.userSubject.getValue();
  }
}
```

```ts
// component-a.component.ts
import { Component } from '@angular/core';
import { UserStore } from './user-store.service';

@Component({...})
export class ComponentA {
  constructor(private userStore: UserStore) {}

  login() {
    this.userStore.setUser({ name: 'Ana', loggedIn: true });
  }
}
```

```ts
// component-b.component.ts
import { Component } from '@angular/core';
import { Observable } from 'rxjs';
import { User } from './user.model';
import { UserStore } from './user-store.service';

@Component({
  template: `
    <div *ngIf="user$ | async as user">
      Welcome, {{ user.name }}!
    </div>
  `
})
export class ComponentB {
  user$ = this.userStore.user$;

  constructor(private userStore: UserStore) {}
}
```

### [NgRx][use_ngrx]

```ts
// user.actions.ts
import { createAction, props } from '@ngrx/store';
import { User } from './user.model';

export const login = createAction('[User] Login', props<{ user: User }>());
```

```ts
// user.reducer.ts
import { createReducer, on } from '@ngrx/store';
import { login } from './user.actions';
import { User } from './user.model';

export interface UserState {
  user: User | null;
}

export const initialState: UserState = { user: null };

export const userReducer = createReducer(
  initialState,
  on(login, (state, { user }) => ({ ...state, user }))
);
```

```ts
// user.selectors.ts
import { createSelector, createFeatureSelector } from '@ngrx/store';
import { UserState } from './user.reducer';

export const selectUserState = createFeatureSelector<UserState>('user');

export const selectUser = createSelector(
  selectUserState,
  (state: UserState) => state.user
);
```

```ts
// component-a.component.ts
import { Component } from '@angular/core';
import { Store } from '@ngrx/store';
import { login } from './user.actions';
import { User } from './user.model';

@Component({...})
export class ComponentA {
  constructor(private store: Store) {}

  login() {
    const user: User = { name: 'Ana', loggedIn: true };
    this.store.dispatch(login({ user }));
  }
}
```

```ts
// component-b.component.ts
import { Component } from '@angular/core';
import { Observable } from 'rxjs';
import { Store } from '@ngrx/store';
import { selectUser } from './user.selectors';
import { User } from './user.model';

@Component({
  template: `
    <div *ngIf="user$ | async as user">
      Welcome, {{ user.name }}!
    </div>
  `
})
export class ComponentB {
  user$: Observable<User | null>;

  constructor(private store: Store) {
    this.user$ = this.store.select(selectUser);
  }
}
```

### [Akita][use_akita]

```ts
// user.store.ts
import { Injectable } from '@angular/core';
import { EntityState, EntityStore, StoreConfig } from '@datorama/akita';
import { User } from './user.model';

export interface UserState extends EntityUser<User> {}

@Injectable({ providedIn: 'root' })
@StoreConfig({ name: 'user' })
export class UserStore extends EntityStore<UserState, User> {
  constructor() {
    super();
  }
}
```

```ts
// user.query.ts
import { Injectable } from '@angular/core';
import { QueryEntity } from '@datorama/akita';
import { UserStore, UserState } from './user.store';

@Injectable({ providedIn: 'root' })
export class UserQuery extends QueryEntity<UserState> {
  constructor(protected store: UserStore) {
    super(store);
  }
}
```

```ts
// user.service.ts
import { Injectable } from '@angular/core';
import { UserStore } from './user.store';
import { UserQuery } from './user.query';
import { User } from './user.model';

@Injectable({ providedIn: 'root' })
export class UserService {
  constructor(private userStore: UserStore, private userQuery: UserQuery) {}

  login(user: User) {
    this.userStore.setLoading(true);
    setTimeout(() => {
      this.userStore.set([user]);
      this.userStore.setLoading(false);
    }, 1000);
  }
}
```

```ts
// component-a.component.ts
import { Component } from '@angular/core';
import { UserStore } from './user.store';

@Component({...})
export class ComponentA {
  constructor(private userService: UserService) {}

  login() {
    this.userService.login({ id: 1, name: 'Ana', loggedIn: true });
  }
}
```

```ts
// component-b.component.ts
import { Component } from '@angular/core';
import { UserQuery } from './user.query';
import { Observable } from 'rxjs';
import { User } from './user.model';

@Component({
  template: `
    <div *ngIf="user$ | async as user">
      Welcome, {{ user.name }}!
    </div>
  `
})
export class ComponentB {
  user$: Observable<User | undefined>;

  constructor(private userQuery: UserQuery) {
    this.user$ = this.userQuery.selectEntity(1);
  }
}
```

---

## Sources

- [https://www.freecodecamp.org/news/best-practices-for-a-clean-and-performant-angular-application-288e7b39eb6f/](https://www.freecodecamp.org/news/best-practices-for-a-clean-and-performant-angular-application-288e7b39eb6f/) – "Bigger picture -> state management" section
- [https://www.sourceallies.com/2020/11/state-management-anti-patterns/](https://www.sourceallies.com/2020/11/state-management-anti-patterns/) – Sections 1 and 2
- [https://roshancloudarchitect.me/identifying-and-eliminating-code-smells-in-angular-micro-frontends-advanced-techniques-for-6f07a781f93d](https://roshancloudarchitect.me/identifying-and-eliminating-code-smells-in-angular-micro-frontends-advanced-techniques-for-6f07a781f93d) – Sections 3 and 6

[use_ngrx]: https://medium.com/@igorm573/state-management-with-ngrx-in-angular-66ddc61cdf14
[use_akita]: https://medium.com/@ShantKhayalian/state-management-in-angular-ngrx-vs-akita-e31d81a2ec87
