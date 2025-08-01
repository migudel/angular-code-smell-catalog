# Multiple Subscriptions

## Description

This code smell occurs when there are **multiple subscriptions to the same `Observable`**, either:

- Explicitly via multiple `.subscribe()` calls in different parts of the component, or
- Implicitly in the HTML template by using the `async` pipe multiple times on the same `Observable`.

This pattern can lead to **undesired side effects**, especially if the `Observable` is **cold** (e.g., an HTTP request), since it re-executes every time it's subscribed to —potentially duplicating requests, computations, or event emissions.

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
> [!warning]
> The solution presented in the source [Blog eyas][1], a 2018 article, (`publishReplay(n) + refCount()`) actually is deprecated since RxJS v7 and will disappear in the upcoming v8.  
> Instead, they recommend using [`share`](https://rxjs.dev/api/operators/share) to share the subscription ref or [`shareReplay`](https://rxjs.dev/api/operators/shareReplay) to cache subscription value.

### Sharing the Subscription

This approach converts a **cold observable** (such as an HTTP request) into a **hot observable**, sharing the subscription to the original source.

```ts
pageTitle = this.route.params.pipe(
  map(params => params["id"]),
  mergeMap(id =>
    this.http.get(`api/pages/${id}/title`, { responseType: "text" })
  ),
  share()
);
```

In this example, only **one subscription** is made to the source observable:

```html
<h1>{{ pageTitle | async }}</h1>
<p>You are viewing {{ pageTitle | async }}.</p>
```

> [!warning]
> This approach only shares emitted values with **already subscribed subscribers**.
> If a new subscriber subscribes after a value has been emitted, it will **not receive the previous value** and must wait for the next emission.

### Caching the Subscription Value

To address the limitation of the previous approach, use `shareReplay({ refCount, bufferSize })`, the modern replacement for the old `publishReplay(1).refCount()` pattern. This approach caches the last `bufferSize` value(s) and replays them to new subscribers that subscribe after the value has been emitted.

```ts
// Deprecated (older RxJS patterns)
journey$ = this.journeyId$.pipe(
  distinctUntilChanged(),
  switchMap(id => this.journeyService.getJourney(id)),
  publishReplay(1),
  refCount()
);

// Recommended (RxJS 7+)
journey$ = this.journeyId$.pipe(
  distinctUntilChanged(),
  switchMap(id => this.journeyService.getJourney(id)),
  shareReplay({
    bufferSize: 1,
    refCount: true
  })
);

```
### Use a single subscription in the template

Group subscriptions in the template by declaring them once as a template variable. This allows you to work with the resolved value directly, rather than treating it as a stream.

```html
<ng-container *ngIf="journey$ | async as journey">
  <h1>{{ journey.title }}</h1>
  <p>{{ journey.description }}</p>
</ng-container>
```

---

## Sources

- [https://medium.com/@robert.maiersilldorff/code-smells-in-angular-deep-dive-part-i-d63dd5f5215e](https://medium.com/@robert.maiersilldorff/code-smells-in-angular-deep-dive-part-i-d63dd5f5215e) – Section 5
- [https://blog.eyas.sh/2018/12/use-asyncpipe-when-possible/](https://blog.eyas.sh/2018/12/use-asyncpipe-when-possible/)

[1]:https://blog.eyas.sh/2018/12/use-asyncpipe-when-possible/
