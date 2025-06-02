# Give Streams to Child Components

## Description

This code smell occurs when an `Observable` or `Subject` is passed directly to a child component via `@Input()`, instead of providing already-consumed data (e.g., via `async` pipe) or handling subscriptions entirely in the parent component.

While technically valid, passing streams directly increases the child component's dependency on RxJS. It forces the child to manage subscriptions, intermediate state, or transformations—responsibilities that should remain in higher-level components.

In Angular, child components should receive **ready-to-display data**, while observables and reactive logic should remain encapsulated in the parent or container component that controls the data flow.

## Why This Is a Code Smell

- **Breaks encapsulation**: the child must know how to handle observables, which violates abstraction.
- **Increases coupling**: the parent dictates how the child must consume the data.
- **Reduces reusability**: the child component cannot be easily reused without passing streams.
- **Makes testing harder**: tests must simulate observables rather than using simple mock data.
- **Violates one-way data flow**: child components should remain passive data consumers, not reactive processors.

---

## Non-Compliant Code Example

```ts
@Component({ 
  template: '<child-component [users]="users$"></child-component>'
})
export class ParentComponent {
  users$ = this.userService.getUsers();
}
```

```ts
@Component({ 
  selector: 'child-component'
})
export class ChildComponent {
  @Input() users!: Observable<User[]>;

  ngOnInit(): void {
    this.users.subscribe(data => {
      // Process data internally
    });
  }
}
```

---

## Compliant Code Example

```ts
@Component({ 
  template: '<child-component [users]="users$ | async"></child-component>'
})
export class ParentComponent {
  users$ = this.userService.getUsers();
}
```

```ts
@Component({ 
  selector: 'child-component'
})
export class ChildComponent {
  @Input() users: User[] = [];
}
```

If the same stream needs to be passed to multiple components, encapsulate the subscription like this:

```ts
@Component({
  template: `
    <ng-container *ngIf="users$ | async as users">
      <child-component [users]="users"></child-component>
      <child-component-2 [users]="users"></child-component-2>
    </ng-container>
  `
})
export class ContainerComponent {
  users$ = this.userService.getUsers();
}
```

This ensures that only a **single subscription** is made and the result is reused across multiple inputs.

---

## Sources

- [https://blog.brecht.io/rxjs-best-practices-in-angular/](https://blog.brecht.io/rxjs-best-practices-in-angular/) – Section 8 (*Don’t pass streams to components directly*)
- [https://www.slideshare.net/slideshow/rxjs-best-bad-practices-for-angular-developers/233392471](https://www.slideshare.net/slideshow/rxjs-best-bad-practices-for-angular-developers/233392471) – Section 5 (*Don't pass streams to components*)
