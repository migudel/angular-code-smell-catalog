# Include Logic in Templates

## Description

This code smell occurs when templates contain excessive or complex logic.

While Angular templates support basic conditions, relying too heavily on in-template logic—such as chained comparisons, nested conditions, or negations—makes the view difficult to read, maintain, and test. Overuse of structural directives like `*ngIf`, `*ngFor`, or `ngSwitch` with complex expressions is another common symptom.

To improve clarity and maintainability, it's recommended to:

- Delegate logic to the component through precomputed properties for values that change infrequently.
- Use lightweight `getters` only for simple or frequently updated logic (though avoid them if performance could be impacted).
- Encapsulate repeated or complex logic in **custom structural directives**.

## Why This Is a Code Smell

- **Breaks separation of concerns:** It mixes business logic with view rendering, violating the single-responsibility principle.
- **Reduces readability:** Makes it harder to understand what the template does at a glance.
- **Complicates debugging:** Logic embedded in templates is harder to inspect and trace.
- **Increases complexity:** Deeply nested conditions lead to bloated, fragile views.
- **Hinders reuse and testing:** Inline conditions can’t be easily reused or unit tested in isolation.

---

## Non-Compliant Code Example

```ts
@Component({
  template: `
    <p *ngIf="role === 'developer'">Status: Developer</p>
  `
})
export class TestComponent implements OnInit {
  ngOnInit(): void {
    this.role = 'developer';
  }
}
```

---

## Compliant Code Example

### Move Logic to the Component

```ts
@Component({
  template: `
    <p *ngIf="showDeveloperStatus">Status: Developer</p>
  `
})
export class TestComponent implements OnInit {
  role = 'developer';
  showDeveloperStatus = false;

  ngOnInit(): void {
    this.updateStatus();
  }

  private updateStatus(): void {
    this.showDeveloperStatus = this.role === 'developer';
  }
}
```

---

### Use Custom Structural Directives

```ts
@Directive({
  selector: '[appIfAdultUser]'
})
export class IfAdultUserDirective {
  @Input() set appIfAdultUser(user: User) {
    const isAdult = user.age > 18;
    this.viewContainer.clear();
    if (isAdult) {
      this.viewContainer.createEmbeddedView(this.templateRef);
    }
  }

  constructor(
    private templateRef: TemplateRef<any>,
    private viewContainer: ViewContainerRef
  ) {}

  private isBanned(user: User): boolean {
    // Business rule logic
    return false;
  }
}
```

```html
<!-- Clear and reusable condition -->
<div *appIfAdultUser="user">
  Welcome, {{ user.name }}!
</div>
```

---

## Sources

- [https://www.freecodecamp.org/news/best-practices-for-a-clean-and-performant-angular-application-288e7b39eb6f/](https://www.freecodecamp.org/news/best-practices-for-a-clean-and-performant-angular-application-288e7b39eb6f/) (Section 17)
- [https://levelup.gitconnected.com/refactoring-angular-applications-be18a7ee65cb](https://levelup.gitconnected.com/refactoring-angular-applications-be18a7ee65cb) (Section 4.3)
