# Nested Subscriptions

> [!Note]
> There are already two ESLint plugins focused on RxJS best practices. Specifically, this code smell is addressed by the following rules:
>
> - [`rxjs/no-nested-subscribe`](https://github.com/cartant/eslint-plugin-rxjs/blob/main/docs/rules/no-nested-subscribe.md)
> - [`rxjs-x/no-nested-subscribe`](https://github.com/JasonWeinzierl/eslint-plugin-rxjs-x/blob/main/docs/rules/no-nested-subscribe.md)

## Description

**Nested subscriptions** occur when one observable is subscribed to inside the callback of another subscription in Angular code. This results in a deeply nested, pyramid-shaped structure that is difficult to read, maintain, and test. It also disregards the powerful composition model of reactive programming with RxJS.

## Why This Is a Code Smell

- **Reduced readability and maintainability**: Nesting subscriptions leads to callback pyramids, making control flow difficult to follow and refactor.
- **Poor error and lifecycle handling**: Errors and unsubscriptions are harder to manage when subscriptions are embedded within others.
- **Violation of reactive principles**: Ignores RxJS’s declarative and composable model, undermining its idiomatic usage.
- **Increased coupling**: Tight binding of async operations reduces modularity and increases fragility.
- **Limited data stream reuse**: Nested flows are often rigid and hard to test or reuse across components.
- **Greater risk of subtle bugs**: Nested subscriptions increase the likelihood of duplicated side effects, missed unsubscriptions, or execution order issues.

---

## Non-Compliant Code Example

```ts
// Consuming the first subscription (data dependency)
this.userService.getUser().subscribe(user => {
  this.orderService.getOrders(user.id).subscribe(orders => {
    this.orders = orders;
  });
});

// Combination of the 2 subscriptions
firstObservable$.pipe(
  take(1)
)
.subscribe(firstValue => {
  secondObservable$.pipe(
    take(1)
  )
  .subscribe(secondValue => {
    console.log(`Combined values are: ${firstValue} & ${secondValue}`);
  });
});
```

---

## Compliant Code Example

```ts
// Consuming the first subscription (data dependency)
this.userService.getUser().pipe(
  switchMap(user => this.orderService.getOrders(user.id))
).subscribe(orders => {
  this.orders = orders;
});

// Combination of the 2 subscriptions
firstObservable$.pipe(
  withLatestFrom(secondObservable$),
  first()
)
.subscribe(([firstValue, secondValue]) => {
  console.log(`Combined values are: ${firstValue} & ${secondValue}`);
});
```

---

## Sources

- [https://alex-klaus.com/angular-code-review/](https://alex-klaus.com/angular-code-review/) (*Nested subscribes* inside section 3: *Neglected RxJs subscriptions*)
- [https://www.freecodecamp.org/news/best-practices-for-a-clean-and-performant-angular-application-288e7b39eb6f/](https://www.freecodecamp.org/news/best-practices-for-a-clean-and-performant-angular-application-288e7b39eb6f/) (Section 9: *Avoid having subscriptions inside subscriptions*)
- [https://blog.brecht.io/rxjs-best-practices-in-angular/](https://blog.brecht.io/rxjs-best-practices-in-angular/) (Section 6: *Avoiding nested subscribes*)
- [https://www.slideshare.net/slideshow/rxjs-best-bad-practices-for-angular-developers/233392471](https://www.slideshare.net/slideshow/rxjs-best-bad-practices-for-angular-developers/233392471) (Section 6: *Subscribe in subscribe*)
- [https://www.thinktecture.com/angular/rxjs-antipattern-1-nested-subs/](https://www.thinktecture.com/angular/rxjs-antipattern-1-nested-subs/)
- [https://www.sourceallies.com/2020/11/state-management-anti-patterns/](https://www.sourceallies.com/2020/11/state-management-anti-patterns/) (Section 4)
- [https://zydesoft.com/must-know-clean-code-principles-in-angular/](https://zydesoft.com/must-know-clean-code-principles-in-angular/) (Section 6: *Restrain owning subscriptions inside subscriptions*)
