# Multiple Subscriptions

## Description

This code smell occurs when there are **multiple subscriptions to the same `Observable`**, either:

- Explicitly via multiple `.subscribe()` calls in different parts of the component, or
- Implicitly in the HTML template by using the `async` pipe multiple times on the same `Observable`.

This pattern can lead to **undesired side effects**, especially if the `Observable` is **cold** (e.g., an HTTP request), since it re-executes every time it's subscribed to—potentially duplicating requests, computations, or event emissions.

In Angular, this often happens when the same `Observable` is used with the `async` pipe in multiple places without sharing the underlying stream. The recommended solution is to **share the stream** using operators like `shareReplay` or `publishReplay(1), refCount()` for **local caching**, or to store the intermediate result in a variable.

---

## Why This Is a Code Smell

- **Triggers unnecessary executions**: including HTTP calls, timers, or calculations.
- **Causes duplicated side effects**: such as multiple actions or redundant rendering.
- **Complicates error tracing**: multiple active subscriptions over the same source obscure the flow.
- **Degrades performance**: more operations, more subscriptions, more re-renders.
- **Breaks clean reactivity**: the `Observable` becomes non-deterministic and less predictable.

---

## Non-Compliant Code Example

```ts
@Component({
  selector: 'journey-list-item',
  templateUrl: './journey-list-item.component.html',
  styleUrls: ['./journey-list-item.component.scss'],
})
export class JourneyListItemComponent {
  @Input() 
  set journeyId(value: number) {
    this.journeyId$.next(value);
  }
  @Output() addAction = new EventEmitter<void>();
  
  private journeyId$ = new BehaviorSubject<Journey | undefined>(undefined);
  journey$ = this.journeyId$.pipe(
    switchMap((id) => this.journeyService.getJourney(id))
  );
}
```

```html
<h1>{{ (journey$ | async).title }}</h1>
<div *ngIf="let journey of (journey$ | async)">
  <span>{{ journey?.description }}</span>
</div>
```

---

## Compliant Code Example

### Use a single subscription in the template

```html
<ng-container *ngIf="journey$ | async as journey">
  <h1>{{ journey.title }}</h1>
  <p>{{ journey.description }}</p>
</ng-container>
```

This way, the subscription result is stored in a local variable, and the `Observable` is only subscribed to once.

### Use `shareReplay(1)` and `distinctUntilChanged()`

```ts
@Component({
  selector: 'journey-list-item',
  templateUrl: './journey-list-item.component.html',
  styleUrls: ['./journey-list-item.component.scss'],
})
export class JourneyListItemComponent implements OnDestroy {
  @Input()
  set journeyId(value: number) {
    this.journeyId$.next(value);
  }

  @Output() addAction = new EventEmitter<void>();

  private readonly journeyId$ = new BehaviorSubject<Journey | undefined>(undefined);
  journey$ = this.journeyId$.pipe(
    distinctUntilChanged(),
    switchMap((id) => this.journeyService.getJourney(id)),
    shareReplay(1) // Replays to multiple subscribers without re-fetching
  );
}
```

> [!note]
> If using manual subscriptions, manage them using [`takeUntilDestroyed`](#angular-16-with-takeuntildestroyed) or with `takeUntil` and a dedicated destruction signal. Prefer `takeUntilDestroyed` if available.


### Use `publishReplay(1)` and `refCount()`

This approach caches the result for reuse across multiple subscribers, avoiding redundant operations.

```ts
pageTitle = this.route.params.pipe(
  map(params => params["id"]),
  flatMap(id =>
    this.http.get(`api/pages/${id}/title`, { responseType: "text" })
  ),
  publishReplay(1),
  refCount()
);
```

```html
<h1>{{ pageTitle | async }}</h1>
<p>You are viewing {{ pageTitle | async }}.</p>
```

### Angular 16+ with `takeUntilDestroyed`

```ts
@Component({...})
export class JourneyListItemComponent {
  journey: Journey | undefined;

  constructor(private journeyService: JourneyService) {
    this.journeyId$.pipe(
      distinctUntilChanged(),
      switchMap(id => this.journeyService.getJourney(id)),
      shareReplay(1),
      takeUntilDestroyed()
    ).subscribe(journey => this.journey = journey);
  }
}
```

---

## Sources

- [https://medium.com/@robert.maiersilldorff/code-smells-in-angular-deep-dive-part-i-d63dd5f5215e](https://medium.com/@robert.maiersilldorff/code-smells-in-angular-deep-dive-part-i-d63dd5f5215e) – Section 5
- [https://blog.eyas.sh/2018/12/use-asyncpipe-when-possible/](https://blog.eyas.sh/2018/12/use-asyncpipe-when-possible/)
