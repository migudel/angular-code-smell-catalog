# Manual Subscriptions

## Description

This code smell occurs when subscriptions are managed manually inside Angular components using `.subscribe()` instead of leveraging the `async` pipe in the template, especially when the observable’s value is only needed for presentation and its lifecycle naturally aligns with the component’s.

While manual subscriptions can be valid in certain scenarios (e.g., cold observables, side effects, or one-off HTTP requests), treating them as code smells and refactoring them blindly can introduce risks — particularly around performance or behavior. It’s important to carefully assess trade-offs before making changes.

The key drawback is that manual subscriptions shift the responsibility of unsubscribing to the developer, which increases the risk of memory leaks and lifecycle issues if not handled properly.  
See also: [Not Unsubscribing Subscriptions](not_unsubscribing_subscriptions.md).

The `async` pipe abstracts away subscription management by automatically subscribing to an observable and disposing of it when the component is destroyed. This leads to cleaner, safer, and more maintainable code.


## Why This Is a Code Smell

- **Memory leak risk:** When the developer forgets to unsubscribe, observables may stay active after the component is destroyed, causing memory leaks.
- **Increased complexity:** Manual subscription management introduces boilerplate logic for tracking and cleaning up resources, making the code harder to read and maintain.
- **Potential data inconsistency:** Manually setting values in the component increases the risk of race conditions or stale data if updates occur outside the subscription’s control.

---

## Non-Compliant Code Example

```ts
@Component({
  template: '<span>{{someStringToDisplay}}</span>'
})
export class Foo implements OnInit {
  someStringToDisplay = "";

  ngOnInit() {
    someObservable
      .pipe(
        takeUntilDestroyed(), 
        map(/* ... */)
      )
      .subscribe((next) => {
        this.someStringToDisplay = next;
        this.ref.markForCheck();
      });
  }
}
```

```ts
@Component({
  template: '<span>{{someStringToDisplay}}</span>'
})
export class Foo implements OnInit, OnDestroy {
  someStringToDisplay = "";
  private subscription = Subscription.EMPTY;

  ngOnInit() {
    this.subscription = someObservable
      .pipe(map(/*...*/))
      .subscribe((next) => {
        this.someStringToDisplay = next;
      });
  }

  ngOnDestroy() {
    this.subscription.unsubscribe();
  }
}
```

---

## Compliant Code Example

```ts
@Component({
  template: '<span>{{someStringToDisplay$ | async}}</span>'
})
export class Foo {
  someStringToDisplay$ = someObservable.pipe(map(/*...*/));
}
```

---

## Sources

- [https://dev.to/this-is-angular/7-deadly-sins-of-angular-1n2j](https://dev.to/this-is-angular/7-deadly-sins-of-angular-1n2j) (5th sin)
- [https://www.freecodecamp.org/news/best-practices-for-a-clean-and-performant-angular-application-288e7b39eb6f/](https://www.freecodecamp.org/news/best-practices-for-a-clean-and-performant-angular-application-288e7b39eb6f/) (Section 5)
- [https://levelup.gitconnected.com/refactoring-angular-applications-be18a7ee65cb](https://levelup.gitconnected.com/refactoring-angular-applications-be18a7ee65cb) (Section 2.3 – second solution for *Memory Leaks*)
- [https://blog.brecht.io/rxjs-best-practices-in-angular/](https://blog.brecht.io/rxjs-best-practices-in-angular/) (Section 7: *Avoiding manual subscribes in Angular*)
- [https://zydesoft.com/must-know-clean-code-principles-in-angular/](https://zydesoft.com/must-know-clean-code-principles-in-angular/) (Section 9)
- [https://blog.eyas.sh/2018/12/use-asyncpipe-when-possible/](https://blog.eyas.sh/2018/12/use-asyncpipe-when-possible/)