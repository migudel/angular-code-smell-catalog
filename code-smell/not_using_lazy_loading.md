# Not Using Lazy Loading

## Description

Lazy loading is a design pattern that delays the import and initialization of modules or standalone components, directives, or pipes in Angular until the user navigates to the route where they are needed.

This code smell occurs when this pattern is not applied, causing the application to bundle all modules together into a single package that is loaded at startup. This results in slower initial loading times.

## Why This Is a Code Smell

- **High initial load times**: Loading the entire application eagerly increases the time to first render, degrading the experience on slow networks.
- **Increased memory usage**: The browser must keep all modules in memory, even if some are never accessed during the session.
- **Poor perceived performance**: Large bundles can block rendering, giving the impression of slowness.
- **Limited scalability**: As the application grows, failing to defer module loading increases bundle size, making maintenance and optimization harder.

---

## Non-Compliant Code Example

```typescript
const routes: Routes = [
  {
    path: '',
    component: LandingComponent
  },
  {
    path: 'about',
    component: AboutComponent
  },
  {
    path: 'products',
    component: ProductListComponent
  },
  {
    path: 'products/:id',
    component: ProductComponent
  },
  {
    path: 'shops',
    component: ShopListComponent
  },
  {
    path: 'shops/:id',
    component: ShopComponent
  }
];
```

In this example, all components are preloaded and included in the main package, even if the user never navigates to those routes.

---

## Compliant Code Example

```typescript
// Main routes
const routes: Routes = [
  // Eager loading for main web sections
  {
    path: '',
    component: LandingComponent
  },
  // Lazy loading for a standalone component
  {
    path: 'about',
    loadComponent: () => 
      import("./features/about/about.component.ts")
      .then((c) => c.AboutComponent),
  },
  // Lazy loading for feature modules
  // All routes defined inside will be prefixed with this path
  // e.g. 'products' + ':id' = 'products/:id'
  // This structure helps scale the application in a modular way
  {
    path: 'products',
    loadChildren: () => 
      import('./features/products/products.routes.ts')
      .then((m) => m.PRODUCTS_ROUTES),
  },
  {
    path: 'shops',
    loadChildren: () => 
      import('./features/shops/shops.routes.ts')
      .then((m) => m.SHOPS_ROUTES),
  }
];
```
```ts
// Product routes
const PRODUCTS_ROUTES: Routes = [
  {
    path: '',
    component: ProductListComponent
  },
  {
    path: ':id',
    component: ProductComponent
  },
];

```
```ts
// Shop routes
const SHOPS_ROUTES: Routes = [
  {
    path: '',
    component: ShopListComponent
  },
  {
    path: ':id',
    component: ShopComponent
  }
];

```

Lazy loading is achieved by using the `Route.loadComponent` and `Route.loadChildren` methods, instructing Angular to dynamically import the components only when the route is accessed. This approach enables progressive loading of grouped sections of the application, improving performance and modularity.

---

## Sources
> [!Note]
> See also:
>
> - [Angular v17: Lazy Loading NgModules][1]
> - [Angular Migration Guide: Route Lazy Loading][2]
> - [Routing in Angular, Routing between modules, Lazy loading in Angular][3]
> - [Lazy Load Standalone Components with "loadComponent"][4]

- [https://dev.to/this-is-angular/7-deadly-sins-of-angular-1n2j](https://dev.to/this-is-angular/7-deadly-sins-of-angular-1n2j) (7th sin: *Eagerly loading all features*)
- [https://javascript-conference.com/blog/angular-code-smells/](https://javascript-conference.com/blog/angular-code-smells/) (Section 4: *Loading Speed*)
- [https://angular-enterprise.com/en/ngpost/courses/bad-practices/](https://angular-enterprise.com/en/ngpost/courses/bad-practices/) (7th point: *lack of splitting in modules, especially lazy modules...*)
- [https://www.freecodecamp.org/news/best-practices-for-a-clean-and-performant-angular-application-288e7b39eb6f/](https://www.freecodecamp.org/news/best-practices-for-a-clean-and-performant-angular-application-288e7b39eb6f/) (Section 8)
- [https://levelup.gitconnected.com/refactoring-angular-applications-be18a7ee65cb](https://levelup.gitconnected.com/refactoring-angular-applications-be18a7ee65cb) (Section 2.1: *Lazy Loading Feature Modules*)
- [https://roshancloudarchitect.me/identifying-and-eliminating-code-smells-in-angular-micro-frontends-advanced-techniques-for-6f07a781f93d](https://roshancloudarchitect.me/identifying-and-eliminating-code-smells-in-angular-micro-frontends-advanced-techniques-for-6f07a781f93d)
- [https://zydesoft.com/must-know-clean-code-principles-in-angular/](https://zydesoft.com/must-know-clean-code-principles-in-angular/) (Section 10)

[1]: https://v17.angular.io/guide/lazy-loading-ngmodules
[2]: https://angular.dev/reference/migrations/route-lazy-loading
[3]:https://chitaranjanbiswal93.medium.com/routing-in-angular-routing-between-modules-lazy-loading-in-angular-c6e6ad259767
[4]: https://ultimatecourses.com/blog/lazy-load-standalone-components-via-load-component
