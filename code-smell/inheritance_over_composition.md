# Inheritance Over Composition

## Description

In Angular application development, it's common to reuse shared logic across components.

This code smell occurs when such reuse leads to a rigid architecture based on class inheritance, where multiple components extend a base component to share functionality. This approach often results in tight coupling and limited flexibility.

Instead, it's recommended to use **composition**: create reusable components or services with interchangeable logic that can be injected and configured per use case. This leads to a more modular, flexible, and testable architecture.

## Why This Is a Code Smell

- **Tight coupling:** Inheritance creates a strong dependency between base and child components, making changes harder and reducing flexibility.
- **Limited reusability:** Derived components inherit all functionality from the base class —even unnecessary parts— leading to bloated and less maintainable components.
- **Difficult unit testing:** Shared dependencies in the base class complicate the testing of derived components, especially when mocking is required.
- **Fragile maintenance:** Any change in the base component can introduce unexpected side effects in all components that inherit from it, increasing the risk of regressions.

---

## Non-Compliant Code Example

### HTML Shared Across Components

```html
<form [formGroup]="myform" (ngSubmit)="save()">
  <h1>Recovery password</h1>
  <input type="text" formControlName="email" />
  <button>Save</button>
  <span *ngFor="let error of errors">{{ error }}</span>
</form>
```

### Base and Inherited Components

```ts
import { FormBuilder, Validators } from '@angular/forms';

export class BaseForm {
  errors = [];
  myform = this.fb.group({
    email: ['', [Validators.required, Validators.email]],
  });

  constructor(private fb: FormBuilder) {}

  save() {
    if (!this.myform.valid) {
      this.showErrors();
    } else {
      this.errors = [];
      console.log('saving data!');
    }
  }

  showErrors() {
    const emailError = this.myform.get('email')?.errors;
    Object.keys(emailError || {}).forEach((value) => {
      this.errors = [...value];
    });
  }
}
```

```ts
@Component({
  selector: 'app-newsletter',
  templateUrl: './share.component.html',
})
export class NewsletterComponent extends BaseForm {
  constructor(public fb: FormBuilder) {
    super(fb);
  }
}
```

```ts
@Component({
  selector: 'app-recovery-password',
  templateUrl: './share.component.html',
})
export class RecoveryPasswordComponent extends BaseForm {
  constructor(public fb: FormBuilder) {
    super(fb);
  }
}
```

In this example, `NewsletterComponent` and `RecoveryPasswordComponent` inherit all logic from `BaseForm`, even if they only need a small subset of it—resulting in unnecessary overhead and rigid coupling.

---

## Compliant Code Example

To achieve greater flexibility and adhere to the **Dependency Inversion Principle** (DIP), we can define an abstract class or interface that establishes a contract. Any service implementing this contract can then be injected interchangeably without modifying the consumer component.  
This pattern decouples the component from concrete implementations, making the application more maintainable and testable.

### Abstract Contract

```ts
export abstract class AbstractFormWrapper {
  abstract myform: FormGroup;
  abstract errors: string[];
  abstract save(form: FormGroup): boolean;
}
```

### Concrete Implementation 1 (`FormWrapperService`)

```ts
@Injectable()
export class FormWrapperService implements AbstractFormWrapper {
  myform: FormGroup;
  private _baseForm: BaseForm;

  public get errors(): string[] {
    return this._baseForm.errors;
  }

  constructor(private fb: FormBuilder, private http: HttpClient) {
    this._baseForm = new BaseForm(this.fb, this.http, 'A');
    this.myform = this._baseForm.myform;
  }

  save(form: FormGroup): boolean {
    this._baseForm.myform = form;
    this._baseForm.save();
    return this._baseForm.errors.length === 0;
  }
}
```

### Concrete Implementation 2 (`FormWrapperTrackingService`)

```ts
@Injectable()
export class FormWrapperTrackingService implements AbstractFormWrapper {
  private _anotherBaseForm: BaseForm;
  myform: FormGroup;

  public get errors(): string[] {
    return this.translationToSpanish();
  }

  constructor(private fb: FormBuilder, private http: HttpClient) {
    this._anotherBaseForm = new BaseForm(this.fb, this.http, 'A');
    this.myform = this._anotherBaseForm.myform;
  }

  save(form: FormGroup): boolean {
    this._anotherBaseForm.myform = form;
    this._anotherBaseForm.save();
    console.log('sending data to another service');
    return this._anotherBaseForm.errors.length === 0;
  }

  private translationToSpanish(): string[] {
    return this._anotherBaseForm.errors.map((a) => {
      return this.translate(a);
    });
  }

  private translate(string) {
    return 'Un error';
  }
}
```

### Component Usage with Dependency Injection

By providing `AbstractFormWrapper` in the component and choosing a concrete implementation (`FormWrapperService` or `FormWrapperTrackingService`), we can switch behavior without modifying component logic.

```ts
@Component({
  selector: 'app-waiting-list',
  templateUrl: './waiting-list.component.html',
  providers: [ {
      provide: AbstractFormWrapper,
      useClass: FormWrapperService
      // Or FormWrapperTrackingService
    } ]
})
export class WaitingListComponent {
  myform: FormGroup;
  errors: string[] = [];

  constructor(private formWrapper: AbstractFormWrapper) {
    this.myform = formWrapper.myform;
  }

  save() {
    if (!this.formWrapper.save(this.myform)) {
      this.errors = this.formWrapper.errors;
    }
  }
}
```

## Sources

- [https://levelup.gitconnected.com/refactoring-angular-applications-be18a7ee65cb](https://levelup.gitconnected.com/refactoring-angular-applications-be18a7ee65cb) (Section 1.1)
- [https://danywalls.com/understand-composition-and-inheritance-in-angular](https://danywalls.com/understand-composition-and-inheritance-in-angular)
- [https://dev.to/this-is-angular/you-dont-want-a-basecomponent-in-your-app-23hn](https://dev.to/this-is-angular/you-dont-want-a-basecomponent-in-your-app-23hn)
- [https://dev.to/vixero/common-mistakes-that-backend-programmers-make-in-angular-434d](https://dev.to/vixero/common-mistakes-that-backend-programmers-make-in-angular-434d) (Section 5)

