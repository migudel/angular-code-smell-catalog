# Include Functions in Templates

## Description

This code smell appears when functions are called directly inside Angular templates—whether in interpolation expressions (`{{ }}`) or structural directives like `*ngIf` or `*ngFor`.

Although it may seem convenient, such calls can severely impact performance and reduce maintainability. These functions are executed on every change detection cycle, often resulting in unnecessary repeated invocations.

Even `getters`, which may appear harmless, are function calls and follow the same behavior: they are reevaluated every time Angular checks the template for changes.

To avoid this, it's recommended to:

- Precompute values inside the component class and bind to variables instead.
- Use custom **pure pipes** to perform declarative transformations in the view.

> [!NOTE]
> There is an `@angular-eslint` rule related to this smell: [`no-pipe-impure`](https://github.com/angular-eslint/angular-eslint/blob/main/packages/eslint-plugin/src/rules/no-pipe-impure.ts), which ensures that custom pipes are pure, although it does not fully address this issue.

## Why This Is a Code Smell

- **Performance degradation:** Functions inside templates are re-evaluated multiple times during change detection, leading to CPU overuse.
- **Hidden inefficiencies:** These issues are difficult to identify by inspecting only the HTML, as the performance cost isn't visually apparent.
- **Reduced readability:** Embedding complex expressions in templates makes them harder to understand and maintain.
- **Testing complications:** Since you cannot control how many times the function is executed, unit testing becomes less predictable and more brittle.

---

## Non-Compliant Code Example

```html
<div *ngIf="isAdult(user)">
  Welcome, {{ user.name.toUpperCase() }}!
</div>

<p>Date: {{ formatDate(createdAt) }}</p>
```

---

## Compliant Code Example

```html
<div *ngIf="isAdultUser">
  Welcome, {{ user.name | uppercaseName }}!
</div>

<p>Date: {{ formattedDate }}</p>
```

```ts
@Component({ 
  selector: 'app-example',
  templateUrl: './newsletter.component.html',
  imports: [UppercaseNamePipe]
})
export class ExampleComponent implements OnInit {

  @Input() user!: { name: string; age: number; };
  @Input() createdAt!: Date;
  
  isAdultUser: boolean = false;
  formattedDate: string = '';

  ngOnInit() {
    this.updateIsAdultUser();
    this.updateFormattedDate();
  }

  private updateIsAdultUser(): void {
    this.isAdultUser = this.user.age > 18 && !this.isBanned(this.user);
  }

  private updateFormattedDate(): void {
    this.formattedDate = this.formatDate(this.createdAt);
  }
}
```

```ts
@Pipe({
  name: 'uppercaseName',
  pure: true
})
export class UppercaseNamePipe implements PipeTransform {
  transform(name: string): string {
    return name.toUpperCase();
  }
}
```

---

## Sources

- [https://alex-klaus.com/angular-code-review/](https://alex-klaus.com/angular-code-review/) (Section 2)
- [https://levelup.gitconnected.com/refactoring-angular-applications-be18a7ee65cb](https://levelup.gitconnected.com/refactoring-angular-applications-be18a7ee65cb) (Section 2.2)
- [https://medium.com/@robert.maiersilldorff/code-smells-in-angular-deep-dive-part-i-d63dd5f5215e](https://medium.com/@robert.maiersilldorff/code-smells-in-angular-deep-dive-part-i-d63dd5f5215e) (Section 2)

