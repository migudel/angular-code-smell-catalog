# No Usage of Bundlers to Reduce Size of Application

> [!note]
> This code smell applies to Angular 8.x applications (when Ivy was experimental and had to be enabled manually). Starting with Angular 9, Ivy is the default rendering engine, and as of Angular 12 it is the only supported engine.

## Description
Failing to enable Angular’s Ivy compiler and AOT (*Ahead‑of‑Time*) compilation prevents you from taking advantage of Ivy’s built‑in optimizations —most importantly tree‑shaking unused code, reducing bundle size, speeding up builds, and improving debugging.

This typically happens when:

- In **Angular 8**, the `enableIvy` flag is set to `false` under `angularCompilerOptions` in **`angular.json`**.
- AOT is disabled (e.g. `"aot": false`) under the build options in **`angular.json`** or **`tsconfig.json`**.

Beyond simply enabling Ivy and AOT, you can further optimize your bundles by:

- **Lazy‑loading** feature modules or standalone components to split bundles.
- **Removing unused imports** from third‑party libraries to help tree‑shaking eliminate dead code.
- **Using pure pipes** (`@Pipe({ pure: true })`) so that Angular can cache transforms and drop unused logic.

By combining these practices, your final production bundles will be smaller, load faster, and perform better on low‑powered or mobile devices.

## Why This Is a Code Smell

- **Excessively large bundles** lead to longer download, parse, and execution times.
- **Missed Ivy optimizations** mean unused code isn’t eliminated and component compilation is less efficient.
- **Poor user experience**, especially for users on slow networks or older hardware.

---

## Non‑Compliant Example

In this example, Ivy and AOT are both disabled, so none of Ivy’s optimizations can run:

### `tsconfig.json`
```jsonc
{
  "projects": {
    "my-existing-project": {
      "architect": {
        "build": {
          "options": {
            "aot": false,
          }
        }
      }
    }
  }
}   
```

### `angular.json`
```json
{
  "angularCompilerOptions": {
    "enableIvy": false,
  }
}
```

---

## Compliant Example

Enable both Ivy and AOT. From Angular 9 onward, you can remove `enableIvy` entirely (it defaults to `true`): 

### `tsconfig.json`
```json
{
  "projects": {
    "my-existing-project": {
      "architect": {
        "build": {
          "options": {
            "aot": true,
          }
        }
      }
    }
  }
}   
```

### `angular.json`
```jsonc
{
  "angularCompilerOptions": {
    // No need for "angularCompilerOptions.enableIvy" from Angular 9+
    "enableIvy": true,
  }
}
```

---

> [!Tip]
> **Additional bundle‑size optimizations:**
>
> - **Use pure pipes** to let Ivy cache and drop unused logic:
>
>   ```ts
>   @Pipe({ name: 'uppercaseName', pure: true })
>   export class UppercaseNamePipe implements PipeTransform {
>     transform(name: string): string {
>       return name.toUpperCase();
>     }
>   }
>   ```
> - **Split by domain** and prefer **standalone components** to encourage code fragmentation.
> - **Implement lazy loading** for feature modules or routes. See [Not Using Lazy Loading](not_using_lazy_loading.md) for details.

---

## Sources

- [https://medium.com/codex/avoid-these-bad-practices-when-you-are-an-angular-developer-135323db74c7](https://medium.com/codex/avoid-these-bad-practices-when-you-are-an-angular-developer-135323db74c7) – Section 4 (*Shake your code*)

