# Use of Stateful Streams

## Description

This code smell arises when application state is handled **outside the RxJS operator chain**, such as storing intermediate results in component properties (e.g., `this.customer`, `this.code`, `this.result`) for later reuse.

While this approach may seem practical, it breaks the reactive and declarative nature of RxJS and **turns the stream logic into a chain of implicit side effects**. As a result, the code becomes harder to reason about, more error-prone, and more difficult to maintain.

Instead, it is recommended to **carry intermediate state within the stream itself**, by composing a “bundle” object that encapsulates all relevant data as it flows through the pipeline. This ensures **purity, consistency, and traceability** in reactive programming.

## Why This Is a Code Smell

- **Breaks reactivity**: State is manually extracted from the stream and stored externally.
- **Introduces side effects**: Mutations occur outside the observable chain, affecting its behavior unpredictably.
- **Reduces clarity**: It becomes unclear when values are set or in which order operations occur.
- **Makes error handling fragile**: A failure at any step can leave the external state in an inconsistent state.
- **Decreases maintainability**: Code becomes harder to refactor, test, and reason about.

---

## Non-Compliant Code Example

```ts
this.route.params.pipe(
  switchMap(({ customerId }) => customerService.getCustomer(customerId)),
  tap((customer) => {
    this.customer = customer;
    this.code = makeCode(customer);
  }),
  switchMap(() => myService.retrieveByCode(this.code)),
  tap((result) => {
    this.result = result;
  }),
  switchMap(() => otherService.byCustomerAndResult(this.customer, this.result)),
).subscribe(combinedResult => {
  this.result = combinedResult;
  this.view = moreComplexComutation(this.customer, this.code, combinedResult);
});
```

---

## Compliant Code Example

```ts
createStream<number>([1, 2], 25).pipe(
  switchMap(id => requestCustomer(id)),
  switchMap((customer) => {
    const code = makeCode(customer);
    return requestByCode(code).pipe(
      map(result => ({ customer, code, result }))
    );
  }),
  switchMap(({ customer, result, ...bundle }) =>
    requestByCustomerAndResult(customer, result).pipe(
      map(combinedResult => ({ ...bundle, customer, result, combinedResult }))
    )
  )
).subscribe(({ customer, code, result, combinedResult }) => {
  updateView('customer', customer.name);
  updateView('code', code);
  updateView('result', result.result);
  updateView('combined', combinedResult);
});
```

---

## Sources

- [RxJS Anti-Pattern: Stateful Streams – Thinktecture](https://www.thinktecture.com/en/angular/rxjs-antipattern-2-state/)
- [Source Allies – State Management Anti-Patterns](https://www.sourceallies.com/2020/11/state-management-anti-patterns/) – Section 5
