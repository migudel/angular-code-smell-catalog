# Include Logic Instead of RxJS Operators

## Description

This code smell occurs when conditional logic, filtering, transformation, or any kind of data manipulation is implemented directly inside the `subscribe()` block or auxiliary functions, instead of leveraging RxJS operators such as `filter`, `map`, or `tap`. This practice breaks the declarative nature of RxJS and deteriorates code clarity and maintainability.

## Why This Is a Code Smell

- **Breaks RxJS's declarative paradigm**: Embedding imperative logic inside `subscribe()` defeats the expressive and composable power of RxJS operators.
- **Reduces data flow readability**: Placing conditions or transformations in `subscribe()` obscures the purpose of the observable pipeline.
- **Increases coupling and complexity**: Mixing side effects with business logic violates the separation of concerns principle.
- **Limits reusability**: Logic encapsulated in `subscribe()` cannot be easily reused across different reactive streams or tested independently.
- **Hinders debugging and testing**: Errors within `subscribe()` are harder to trace and isolate compared to well-structured operators like `filter()` or `map()`.
- **Deviates from reactive best practices**: A subscription should focus solely on terminal side effects (e.g., UI rendering, storage, routing), not on core data manipulation.

---

## Non-Compliant Code Example

```ts
obs$.pipe(
  map(v => v * 10)
).subscribe(v => {
  if (v < 50) {
    console.log("OK");
  }
});
```

---

## Compliant Code Example

```ts
obs$.pipe(
  map(v => v * 10),
  filter(v => v < 50)
).subscribe(v => {
  console.log("OK");
});
```

---

## Sources

- [https://www.slideshare.net/slideshow/rxjs-best-bad-practices-for-angular-developers/233392471](https://www.slideshare.net/slideshow/rxjs-best-bad-practices-for-angular-developers/233392471) (Section 10)
