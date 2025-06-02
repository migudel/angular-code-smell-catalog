# No Usage of Bundlers to Reduce Size of Application

## Description

This code smell occurs when an Angular application fails to leverage the available tools and configurations to minimize the final bundle size delivered to the client. This includes misconfigured builds, the absence of *lazy loading*, or inefficient use of third-party libraries.

Since Angular 9, the introduction of the **Ivy** compiler and rendering engine has enabled significantly smaller bundle sizes through features like **tree-shaking** and more granular component compilation. Ivy became the default and only rendering engine starting from Angular 12. However, taking full advantage of its benefits requires adhering to specific optimization practices:

- Using the `ng build --configuration production` command
- Removing unused global imports
- Using pure pipes (`pure: true`)
- Applying lazy loading for modules and components
- Configuring build budgets in `angular.json`
- Structuring the app by domain using standalone components where appropriate

These techniques reduce the size of generated bundles, leading to faster load times, better user experience, and improved performance on low-powered or mobile devices.

---

## Why This Is a Code Smell

- **Leads to unnecessarily large bundles**: increasing download, parse, and execution times in the browser.
- **Fails to leverage Ivy’s optimizations**: such as eliminating unused code and efficiently compiling components.
- **Violates the principle of progressive modularity**: by loading the entire application eagerly.
- **Impacts user experience negatively**: especially under poor network conditions or on low-end devices.
- **Hinders maintainability and scalability**: by lacking visibility and constraints on bundle sizes and artifacts.

---

## Non-Compliant Code Example

- Not using `ng build --configuration production`

- No budget configuration in `angular.json`

- Importing entire libraries unnecessarily:

  ```ts
  // Tree-shaking is not possible
  import * as lodash from 'lodash';
  ```

- Eagerly loading large modules:

  ```ts
  import { AdminModule } from './admin/admin.module';

  const routes: Routes = [
    {
      path: 'admin',
      children: AdminModule.routes
    }
  ];
  ```

---

## Compliant Code Example

### Optimized Build Command

```bash
ng build --configuration production
```

### Specific Imports

```ts
// Enables Tree-shaking
import cloneDeep from 'lodash/cloneDeep'; 
```

### Lazy Loading Modules

```ts
const routes: Routes = [
  {
    path: 'admin',
    loadChildren: () =>
      import('./admin/admin.module').then(m => m.AdminModule)
  }
];
```

### Budget Configuration in `angular.json`

```json
"configurations": {
  "production": {
    "budgets": [
      {
        "type": "initial",
        "maximumWarning": "2mb",
        "maximumError": "5mb"
      },
      {
        "type": "anyComponentStyle",
        "maximumWarning": "6kb"
      }
    ],
    "optimization": true,
    "outputHashing": "all",
    "aot": true,
    "extractCss": true,
    "namedChunks": false,
    "extractLicenses": true,
    "vendorChunk": false,
    "buildOptimizer": true,
    "sourceMap": false
  }
}
```

---

> [!Tip]
> To improve bundle efficiency and reduce overall size:
>
> - **Use pure pipes** to enable Ivy to exclude unused logic:
>
>   ```ts
>   @Pipe({ name: 'uppercaseName', pure: true })
>   export class UppercaseNamePipe implements PipeTransform {
>     transform(name: string): string {
>       return name.toUpperCase();
>     }
>   }
>   ```
> - **Split code by domain**, and use `standalone` components when possible to encourage bundle fragmentation and modular delivery.

---

## Sources

- [https://medium.com/codex/avoid-these-bad-practices-when-you-are-an-angular-developer-135323db74c7](https://medium.com/codex/avoid-these-bad-practices-when-you-are-an-angular-developer-135323db74c7) – Section 4 (*Shake your code*)

