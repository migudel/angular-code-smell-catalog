# Use a Component in Multiple Modules

## Description

One of the most common—and often overlooked—mistakes in Angular is **attempting to declare the same component in multiple modules** (`NgModule`). Angular **explicitly disallows** a component from being listed in the `declarations` array of more than one module, and doing so will result in compilation errors.

This code smell points to a lack of modular organization in the project’s architecture. The proper solution is to **declare the component once in its own feature module, export it from there, and import that module wherever the component is needed**.

---

## Why This Is a Code Smell

- **Explicitly forbidden by Angular**: A component cannot be declared in more than one module. Doing so produces compile-time errors such as:
  > `Type HeroComponent is part of the declarations of 2 modules...`
- **Indicates poor architectural planning**: Attempting to reuse a component without encapsulating it in its own module.
- **Hinders scalability**: Duplication-related issues accumulate as the codebase grows.
- **Breaks modular reusability principles**: Shared components should be properly encapsulated and reusable via module exports.

---

## Non-Compliant Code Example

The same component is declared in two different modules:

```ts
@NgModule({
  declarations: [HeroComponent],
  exports: [HeroComponent]
})
export class AdminModule {}
```

```ts
@NgModule({
  declarations: [HeroComponent], // Already declared in AdminModule
  exports: [HeroComponent]
})
export class DashboardModule {}
```

---

## Compliant Code Example

Declare the component in a dedicated feature module and reuse it through `exports`/`imports`:

```ts
// hero.module.ts
@NgModule({
  declarations: [HeroComponent],
  exports: [HeroComponent]
})
export class HeroModule {}
```

```ts
// admin.module.ts
@NgModule({
  imports: [HeroModule]
})
export class AdminModule {}
```

```ts
// dashboard.module.ts
@NgModule({
  imports: [HeroModule]
})
export class DashboardModule {}
```

---

## Sources

- [https://www.tatvasoft.com/outsourcing/2021/07/top-angular-developer-pitfalls.html](https://www.tatvasoft.com/outsourcing/2021/07/top-angular-developer-pitfalls.html) – Section 1.6
- [https://chudovo.com/most-common-angular-mistakes-every-developer-should-avoid/](https://chudovo.com/most-common-angular-mistakes-every-developer-should-avoid/) – Section 3 (*Declaring the Same Component in More than One NgModule*)
- [https://medium.com/@OPTASY.com/what-are-the-5-most-common-angular-mistakes-that-developers-make-53f6d7c5bf65](https://medium.com/@OPTASY.com/what-are-the-5-most-common-angular-mistakes-that-developers-make-53f6d7c5bf65) – Section 5
