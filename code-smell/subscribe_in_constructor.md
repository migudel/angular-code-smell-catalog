# Subscribe in Constructor

## Description

This code smell occurs when a subscription is made directly inside the constructor of a component, instead of using the `ngOnInit` lifecycle hook. While this may seem straightforward, it breaks Angular’s lifecycle conventions and can lead to issues in maintainability, testability, and resource management.

Best practices recommend keeping constructors clean and free from logic. Any initialization involving service calls, subscriptions, or side effects should be handled within lifecycle hooks such as `ngOnInit`.

> [!note]
> If the subscription is only needed for rendering data, consider removing it entirely and using the `async` pipe instead.
> See also: [Manual subscriptions](manual_subscriptions.md)

## Why This Is a Code Smell

- **Violates the single responsibility principle**: The constructor should only handle basic dependency injection and not contain operational logic.
- **Hinders testability**: Automatically triggering subscriptions on instantiation introduces side effects that complicate unit tests.
- **Lifecycle misalignment**: Bypassing Angular’s lifecycle hooks can result in errors when interacting with elements that are not yet initialized.
- **Increased risk of memory leaks**: Subscriptions made in constructors are more prone to being forgotten or mismanaged, especially without a structured cleanup strategy like `ngOnDestroy`.

---

## Non-Compliant Code Example

```ts
@Component({ ... })
export class UserComponent {
  user: User | null = null;

  constructor(private userService: UserService) {
    this.userService.getUser().subscribe(user => {
      this.user = user;
    });
  }
}
```

---

## Compliant Code Example

```ts
@Component({ ... })
export class UserComponent implements OnInit, OnDestroy {
  user: User | null = null;
  private userSubscription: Subscription | null = null;

  constructor(private userService: UserService) {}

  ngOnInit(): void {
    this.userSubscription = this.userService.getUser().subscribe(user => {
      this.user = user;
    });
  }

  ngOnDestroy(): void {
    this.userSubscription?.unsubscribe();
  }
}
```

---

## Sources

- [https://www.slideshare.net/slideshow/rxjs-best-bad-practices-for-angular-developers/233392471](https://www.slideshare.net/slideshow/rxjs-best-bad-practices-for-angular-developers/233392471) (Section 10)
