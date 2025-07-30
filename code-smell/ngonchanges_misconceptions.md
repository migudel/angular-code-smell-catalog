# `ngOnChanges` Misconceptions

## Description

This code smell refers to the improper and often confusing use of Angular's `ngOnChanges` and `ngDoCheck` lifecycle hooks —especially when developers do not fully understand when or how these hooks are triggered.

Although both hooks are designed to react to changes, they serve **different purposes** and exhibit **very different behaviors**. Mixing logic between them often leads to redundant, inefficient, or hard-to-maintain code.

- `ngOnChanges` is specifically designed to **detect changes in `@Input` properties**. It only fires when the **reference** of an input object changes (e.g., when using `{ updated: true, ...old }`). It does **not detect internal mutations** if the reference remains the same. Many developers mistakenly assume it detects any change in the input data, which it does not.

- `ngDoCheck`, on the other hand, triggers **on every change detection cycle** and allows for **custom change detection logic**. It is useful when you need to detect internal mutations or perform deep comparisons. However, it is also more error-prone and can introduce **performance or logic issues** if not used carefully.

In general, when you only need to react to reference changes, prefer `ngOnChanges`. When tracking deep or internal mutations, `ngDoCheck` may be appropriate —but it requires caution and additional management to avoid inefficiency or unintended behavior. [lifecycles-hooks]

> [!note]
> When using `ngDoCheck`, be mindful of performance implications. It is strongly recommended to configure `ChangeDetectionStrategy.OnPush` and ensure changes are signaled correctly.
>
> While examples below do not use OnPush explicitly, you should consider:
>
> - Cloning objects with updated properties (e.g., `{ updated: true, ...old }`) so that reference changes can be detected by `ngOnChanges`.
> - Using Observables instead of manual mutations, aligning with Angular's reactive patterns.
> - As a last resort, calling `ChangeDetectorRef.markForCheck()` —though this should be avoided in favor of better architectural strategies.


## Why This Is a Code Smell

- **Violates the single responsibility principle**: Logic is split or duplicated between `ngDoCheck` and `ngOnChanges`, making the code harder to reason about.
- **Unnecessary complexity** and **underutilization of the framework**: Using `ngDoCheck` when `ngOnChanges` suffices increases cognitive load and maintenance effort. While `ngOnChanges` provides structured `SimpleChanges` automatically, `ngDoCheck` requires custom logic to track changes.
- **Misunderstanding of `ngOnChanges`**: Many developers incorrectly assume it tracks deep mutations, leading to unexpected bugs and stale UI states.
- **False sense of reactivity**: Relying solely on `ngOnChanges` may mask deeper issues in state propagation or mutation tracking.
- **Performance degradation**: `ngDoCheck` executes on every change detection cycle; if misused, it can introduce costly and unnecessary checks.
- **Harder to test and debug**: Logic scattered across lifecycle hooks is more difficult to isolate, test, and debug.

---

## Common Code: Parent Components Triggering Changes

These parent components interact with child components that detect input changes using Angular lifecycle hooks like `OnChanges` and `DoCheck`. Two types of changes are demonstrated:

1. **Mutation of an object's internal property (same reference)**
2. **Replacement of the object or value (reference change)**

### Parent Component: Modifying an Object’s Property Internally
[parent-change-object-property]:#parent-component-modifying-an-objects-property-internally

In this case, the component's property `user` is passed to the child component as an input. However, only a **property** of the `user` object is modified—**the object reference remains the same**. As a result, Angular’s `ngOnChanges` lifecycle hook in the child component **will not be triggered**, since Angular performs change detection based on reference changes, not internal mutations.

```ts
@Component({
  selector: 'app-parent',
  template: `
    <app-child [user]="user"></app-child>
    <button (click)="updateAge()">Update Age</button>
  `
})
export class ParentComponent {
  user = { name: 'Pepe', age: 25 };

  updateAge() {
    // Internal mutation: object reference doesn't change
    this.user.age += 1;
  }
}
```

### Parent Component: Changing Object Reference
[parent-change-reference]:#parent-component-changing-object-reference

In this case, the component's property `user` is passed to the child component. Instead of mutating a property of the object, the entire `user` object is replaced with a **new reference**. This change **will be detected** by Angular’s `ngOnChanges` lifecycle hook in the child component, as Angular detects input changes through **reference comparison**.

```ts
@Component({
  selector: 'app-parent',
  template: `
    <app-child [name]="userName"></app-child>
    <button (click)="changeName()">Change Name</button>
  `
})
export class ParentComponent {
  private original = 'Pepe';
  private clicks = 0;
  userName = this.original;

  changeName() {
    // Reference change: triggers ngOnChanges in the child
    this.userName = `${this.original}${++this.clicks}`;
  }
}
```

## Non-Compliant Code Example

### Misuse of `ngOnChanges`

> [!note]
> The change in this example originates from the parent defined in [**Parent Component: Modifying an Object’s Property Internally**][parent-change-object-property].

This component incorrectly assumes that Angular will detect changes to a nested object's internal properties. However, since the object reference remains the same, the `ngOnChanges` lifecycle hook is **not triggered**.

```ts
import { Component, Input, OnChanges, SimpleChanges } from '@angular/core';

@Component({
  selector: 'app-child',
  template: `<p>{{ user.name }} - {{ user.age }}</p>`
})
export class ChildComponent implements OnChanges {
  @Input() user: { name: string; age: number };

  ngOnChanges(changes: SimpleChanges) {
    if (changes['user']) {
      console.log('User input changed!');
    }
  }
}
```

> 🛑 This approach fails to detect internal mutations (e.g., `user.age++`) unless the object reference changes.

### Misuse of `ngDoCheck`

> [!note]
> The change in this example originates from the parent defined in [**Parent Component: Changing Object Reference**][parent-change-reference].

This component uses `ngDoCheck` to manually track primitive input changes. While it technically works, it replicates functionality that `ngOnChanges` already handles more efficiently, making this approach unnecessarily verbose and error-prone.

```ts
@Component({
  selector: 'app-child',
  template: `<p>{{ name }}</p>`,
})
export class ChildComponent implements DoCheck {
  @Input() name: string;
  private previousName: string;

  ngDoCheck() {
    if (this.name !== this.previousName) {
      console.log('Name changed:', this.name);
      this.previousName = this.name;
    }
  }
}
```

> 🛑 Prefer `ngOnChanges` for simple input comparisons. Reserve `ngDoCheck` for advanced scenarios where Angular's default change detection falls short.

---

## Compliant Code Example

### Proper Use of `ngDoCheck` for Object Mutations

> [!note]
> The change in this example originates from the parent defined in [**Parent Component: Modifying an Object’s Property Internally**][parent-change-object-property].

When you need to detect internal property changes in an object without changing its reference, use `KeyValueDiffers` with `ngDoCheck`.

```ts
@Component({
  selector: 'app-child',
  template: `<p>{{ user.name }} - {{ user.age }}</p>`
})
export class ChildComponent implements DoCheck {
  @Input() user: { name: string; age: number };
  private differ: any;

  constructor(private differs: KeyValueDiffers) {}

  ngOnInit() {
    this.differ = this.differs.find(this.user).create();
  }

  ngDoCheck() {
    const changes = this.differ.diff(this.user);
    if (changes) {
      console.log('User object changed internally');
    }
  }
}
```

> ✅ This approach correctly detects internal mutations even when the object reference remains unchanged.

### Proper Use of `ngOnChanges` for Reference Changes

> [!note]
> The change in this example originates from the parent defined in [**Parent Component: Changing Object Reference**][parent-change-reference].

This is the recommended and idiomatic use of `ngOnChanges` to detect changes in primitive inputs or object reference replacements.

```ts
@Component({
  selector: 'app-child',
  template: `<p>{{ name }}</p>`
})
export class ChildComponent implements OnChanges {
  @Input() name: string;

  ngOnChanges(changes: SimpleChanges) {
    if (changes['name']) {
      console.log('Name changed:', this.name);
    }
  }
}
```

> ✅ Use `ngOnChanges` for clean, reliable detection of changes to primitives or reference-type inputs.

---

## Sources

- [https://chudovo.com/most-common-angular-mistakes-every-developer-should-avoid/](https://chudovo.com/most-common-angular-mistakes-every-developer-should-avoid/) – Section 6 (*ngOnChanges misconceptions*)
- [https://medium.com/@OPTASY.com/what-are-the-5-most-common-angular-mistakes-that-developers-make-53f6d7c5bf65](https://medium.com/@OPTASY.com/what-are-the-5-most-common-angular-mistakes-that-developers-make-53f6d7c5bf65) – Section 1

[lifecycles-hooks]: https://v17.angular.io/guide/lifecycle-hooks#lifecycle-event-sequence
