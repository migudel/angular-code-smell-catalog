# Modify DOM Directly

## Description

This code smell occurs when the DOM is directly modified using native browser APIs such as `document.querySelector`, `document.getElementById`, `innerHTML`, or `appendChild`, or by relying on external libraries like jQuery. It also applies when directly accessing the DOM via `ElementRef.nativeElement`.

Angular is a framework built around a **declarative and reactive model**. It provides safe, scalable, and testable mechanisms to reflect state changes in the view through its templating system, directives, bindings, and encapsulation. Bypassing these abstractions to manipulate the DOM manually **breaks the framework's design principles**, introduces security risks, reduces maintainability, and complicates testing.

Instead of directly manipulating the DOM, Angular encourages the use of:

- **Template bindings** (`[hidden]`, `[style]`, `[class]`, `[attr.disabled]`, `[innerHTML]`, etc.)
- **Structural and attribute directives** (`*ngIf`, `*ngFor`, `ngClass`, `ngStyle`)
- **Custom directives** for more advanced behavior
- `ElementRef` should only be used in highly controlled and fully sanitized contexts.
- **`Renderer2`** for safe and decoupled DOM manipulation

> [!warning]
> This last option is provided by Angular and had one property ([`nativeElement`](https://angular.dev/api/core/ElementRef)) which it is not recommended. Direct DOM manipulation should only be considered if it is absolutely necessary. Before applying it, carefully reconsider other safer alternatives. If you must use it, it is strongly recommended to combine it with DomSanitizer to ensure maximum security.

## Why This Is a Code Smell

- **Breaks Angular's declarative model**: Direct DOM manipulation circumvents the binding system and alters the view outside of Angular’s controlled update cycle.
- **Introduces security risks**: Unsanitized use of `innerHTML` can open the door to Cross-Site Scripting (XSS) attacks.
- **Reduces platform compatibility**: Code relying on direct DOM access may fail in server-side rendering environments like Angular Universal or in Web Workers.
- **Impairs maintainability**: Coupling logic to HTML structure makes code fragile to view changes.
- **Complicates testability**: Direct DOM access leads to fragile and unpredictable tests.
- **Violates separation of concerns**: Blends UI logic with platform-specific manipulation.

---

## Non-Compliant Code Example

```ts
ngAfterViewInit(): void {
  const input = document.querySelector('.input');
  if (input) {
    input.setAttribute('disabled', 'true');
    input.classList.add('highlighted');
  }

  const container = document.getElementById('main');
  if (container) {
    container.innerHTML = '<p>Hello</p>';
  }
}
```

---

## Compliant Code Examples

### Declarative bindings

```ts
@Component({
  selector: 'app-test',
  templateUrl: './test.component.html',
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class TestComponent {
  isDisabled = true;
  isHighlighted = true;
  htmlContent = this.sanitizer.bypassSecurityTrustHtml('<p>Hello</p>');
  // The following code is not recommended if it is not absolutely 
  // necessary but is shown to correspond to the non-compliant example
  constructor(private sanitizer: DomSanitizer) {}
}
```

```html
<!-- Declarative and safe -->
<input [disabled]="isDisabled" [ngClass]="{ 'highlighted': isHighlighted }" />
<div [innerHTML]="htmlContent"></div>
```

### Custom directive

```ts
@Directive({
  selector: '[appAutoFocus]'
})
export class AutoFocusDirective implements AfterViewInit {
  constructor(private el: ElementRef) {}

  ngAfterViewInit(): void {
    this.el.nativeElement.focus();
  }
}
```

```html
<input appAutoFocus />
```

### Controlled DOM manipulation using Renderer2

```ts
@Component({ ... })
export class ButtonComponent implements AfterViewInit {
  @ViewChild('myBtn') buttonRef!: ElementRef;

  constructor(private renderer: Renderer2) {}

  ngAfterViewInit(): void {
    this.renderer.setAttribute(this.buttonRef.nativeElement, 'disabled', 'true');
    this.renderer.addClass(this.buttonRef.nativeElement, 'highlighted');
  }
}
```

```html
<button #myBtn>Click me</button>
```

---

## Sources

- [TatvaSoft – Common Angular Pitfalls](https://www.tatvasoft.com/outsourcing/2021/07/top-angular-developer-pitfalls.html) – Section 1.4
- [SonarSource Rule RSPEC-6268](https://rules.sonarsource.com/typescript/RSPEC-6268/)
- [Chudovo – Most Common Angular Mistakes](https://chudovo.com/most-common-angular-mistakes-every-developer-should-avoid/) – Sections 1 and 2
- [OPTASY – Top 5 Angular Mistakes](https://medium.com/@OPTASY.com/what-are-the-5-most-common-angular-mistakes-that-developers-make-53f6d7c5bf65) – Section 4
