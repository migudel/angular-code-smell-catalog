# No Usage of Pipes

## Description

This code smell occurs when Angular **pipes**—especially *pure pipes*—are omitted in favor of less suitable alternatives such as getters, method calls within the template, or pre-transformed properties declared in the component.

By doing so, developers break Angular's declarative paradigm, increase maintenance complexity, and may introduce performance issues, particularly when the transformation is expensive or applied to large datasets.

## Why This Is a Code Smell

- **Violates separation of concerns**: Transformation logic is embedded either in the component logic or directly within the template, rather than being cleanly abstracted.
- **Decreases template readability**: Using functions or getters instead of pipes results in less expressive and harder-to-read templates.
- **Hurts performance**: Methods called in the template are re-evaluated on every change detection cycle—even if their inputs haven't changed.
- **Reduces maintainability and scalability**: Duplicating transformation logic across multiple components or controllers increases the likelihood of inconsistencies and bugs.
- **Neglects the benefits of pure pipes**: Pure pipes are automatically memoized and optimized by Angular, making them a lightweight and efficient solution when inputs remain unchanged.
- **Limits reusability**: Logic that could be encapsulated in a declarative, testable, and reusable pipe remains fragmented or duplicated.

---

## Non-Compliant Code Example

```ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-pipe-demonstration',
  template: `<form>
    <div>
      <label>Enter a file size</label>
      <input type="number" (change)="valuechange($event)" />
      <p>{{ formattedSize }}</p>
    </div>
  </form>`,
})
export class PipeDemonstrationComponent {
  formattedSize = '';

  public valuechange(event: any) {
    const size = Number.parseInt(event.target.value);
    this.formattedSize = (size / (1024 * 1024)).toFixed(2) + 'MB';
  }
}
```

---

## Compliant Code Example

```ts
import { Pipe, PipeTransform } from '@angular/core';

@Pipe({ name: 'filesize' })
export default class FileSizeFormatterPipe implements PipeTransform {
  transform(size: number): string {
    return (size / (1024 * 1024)).toFixed(2) + 'MB';
  }
}
```

```ts
import { Component } from '@angular/core';

@Component({
  selector: 'app-pipe-demonstration',
  template: `<form>
    <div>
      <label>Enter a file size</label>
      <input
        type="number"
        [(ngModel)]="size"
        [ngModelOptions]="{ standalone: true }"
      />
      <p>{{ size | filesize }}</p>
    </div>
  </form>`,
})
export class PipeDemonstrationComponent {
  size = 0;
}
```

---

## Sources

- [https://codeburst.io/angular-bad-practices-eab0e594ce92](https://codeburst.io/angular-bad-practices-eab0e594ce92) – Section 7 (*Not using/Misusing Pipes*)
- [https://medium.com/codex/avoid-these-bad-practices-when-you-are-an-angular-developer-135323db74c7](https://medium.com/codex/avoid-these-bad-practices-when-you-are-an-angular-developer-135323db74c7) – Section 2 (*Misusing or not using some Angular features*)
