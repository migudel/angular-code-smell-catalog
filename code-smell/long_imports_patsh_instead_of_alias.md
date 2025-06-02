# Long Import Paths Instead of Aliases

## Description

This code smell arises when long and deeply nested **relative import paths** are used (e.g., `../../../../services/user/user.service`) instead of defining and using **import aliases** (e.g., `@services/user/user.service`). As the folder structure of an Angular application grows, deep relative paths become increasingly difficult to read, maintain, and refactor.

Angular (via TypeScript) supports **custom path aliases** defined in the `tsconfig.json` file. Using aliases results in cleaner, more maintainable, and more stable imports, improving overall code quality and developer experience.

## Why This Is a Code Smell

- **Reduces readability**: long relative paths are visually noisy and hard to follow.
- **Increases maintenance burden**: changes in folder structure require updating many imports.
- **Error-prone**: it’s easy to miscount the number of `../` segments and reference the wrong path.
- **Hurts scalability**: large projects become harder to navigate and maintain with relative imports.
- **Hinders navigation in editors**: autocomplete and path resolution become less predictable and harder to manage.

---

## Non-Compliant Code Example

```ts
import { UserService } from '../../../../core/services/user/user.service';
import { environment } from '../../../../environments/environment';
```

---

## Compliant Code Example

```ts
// tsconfig.json or tsconfig.base.json
{
  "compilerOptions": {
    "baseUrl": "./",
    "paths": {
      "@core/*": ["src/app/core/*"],
      "@env": ["src/environments/environment"],
      "@shared/*": ["src/app/shared/*"]
    }
  }
}
```

```ts
import { UserService } from '@core/services/user/user.service';
import { environment } from '@env';
```

> [!tip]
> Define aliases for common folders like `@core`, `@shared`, `@env`, `@features`, and `@models` to streamline imports and improve maintainability.

---

## Sources

- [https://javascript-conference.com/blog/angular-code-smells/](https://javascript-conference.com/blog/angular-code-smells/) – Section 1 (*Code smells*)
