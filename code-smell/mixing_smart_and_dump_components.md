# Mixing Smart and Dumb Components

## Description

This code smell occurs when a component mixes business logic (such as data retrieval, transformation, and distribution) with presentation logic and view management. This usually indicates a flawed design or misuse of Angular's component model.

The **Smart (container)** and **Dumb (presentational)** component pattern advocates for a clear separation of concerns:

- **Smart Components** are responsible for fetching data, managing state, and coordinating services.
- **Dumb Components** are concerned only with presenting the data received through `@Input()` and emitting events via `@Output()`.

By keeping the dumb component decoupled from business logic, we increase its reusability, testability, and maintainability.

## Why This Is a Code Smell

- **Violation of the Single Responsibility Principle**: A monolithic component that handles both business logic and UI breaks the SOLID SRP principle, making it harder to reason about or evolve.
- **Difficult to unit test**: Mixing logic and UI requires unnecessary dependencies in unit tests, which increases test complexity and brittleness.
- **Limited reusability**: The UI logic is tightly coupled with context-specific business rules, making it harder to reuse the presentation layer in other contexts without duplication.
- **Higher maintenance cost**: Any change—whether in logic or UI—requires modifying a single large file, increasing the risk of introducing regressions or unrelated bugs.

---

## Non-Compliant Code Example

```ts
@Component({
  selector: 'app-home',
  template: `
    <h2>All Lessons</h2>
    <h4>Total Lessons: {{lessons.length}}</h4>
    <div class="lessons-list-container v-h-center-block-parent">
        <table class="table lessons-list card card-strong">
            <tbody>
            <tr *ngFor="let lesson of lessons" (click)="selectLesson(lesson)">
                <td class="lesson-title"> {{lesson.description}} </td>
                <td class="duration">
                    <i class="md-icon duration-icon">access_time</i>
                    <span>{{lesson.duration}}</span>
                </td>
            </tr>
            </tbody>
        </table>
    </div>
  `,
  styleUrls: ['./home.component.css']
})
export class HomeComponent implements OnInit {

  lessons: Lesson[];

  constructor(private lessonsService: LessonsService) {}

  ngOnInit() {
    this.lessonsService.findAllLessons()
      .pipe(tap(console.log))
      .subscribe(lessons => this.lessons = lessons);
  }

  selectLesson(lesson) {
    // ...
  }
}
```

---

## Compliant Code Example

### Dumb Component (Presentational)

```ts
@Component({
  selector: 'lessons-list',
  template: `
    <table class="table lessons-list card card-strong">
        <tbody>
        <tr *ngFor="let lesson of lessons" (click)="selectLesson(lesson)">
            <td class="lesson-title">{{lesson.description}}</td>
            <td class="duration">
                <i class="md-icon duration-icon">access_time</i>
                <span>{{lesson.duration}}</span>
            </td>
        </tr>
        </tbody>
    </table>
  `,
  styleUrls: ['./lessons-list.component.css']
})
export class LessonsListComponent {
  @Input() lessons: Lesson[];
  @Output() lesson = new EventEmitter<Lesson>();

  selectLesson(lesson: Lesson) {
    this.lesson.emit(lesson);
  }
}
```

### Smart Component (Container / Business Logic)

```ts
@Component({
  selector: 'app-home',
  template: `
    <h2>All Lessons</h2>
    <h4>Total Lessons: {{lessons.length}}</h4>
    <div class="lessons-list-container v-h-center-block-parent">
        <lessons-list [lessons]="lessons" (lesson)="selectLesson($event)"></lessons-list>
    </div>
  `,
  styleUrls: ['./home.component.css']
})
export class HomeComponent implements OnInit {

  lessons: Lesson[];

  constructor(private lessonsService: LessonsService) {}

  ngOnInit() {
    this.lessonsService.findAllLessons()
      .subscribe(lessons => this.lessons = lessons);
  }

  selectLesson(lesson: Lesson) {
    // ...
  }
}
```

---

## Sources

- [https://dev.to/this-is-angular/7-deadly-sins-of-angular-1n2j](https://dev.to/this-is-angular/7-deadly-sins-of-angular-1n2j) (3rd sin)
- [https://angular-enterprise.com/en/ngpost/courses/bad-practices/](https://angular-enterprise.com/en/ngpost/courses/bad-practices/) (Point 1)
- [https://levelup.gitconnected.com/refactoring-angular-applications-be18a7ee65cb](https://levelup.gitconnected.com/refactoring-angular-applications-be18a7ee65cb) (Section 1.2)
- [https://www.freecodecamp.org/news/best-practices-for-a-clean-and-performant-angular-application-288e7b39eb6f/](https://www.freecodecamp.org/news/best-practices-for-a-clean-and-performant-angular-application-288e7b39eb6f/) (Section 12)
- [https://roshancloudarchitect.me/identifying-and-eliminating-code-smells-in-angular-micro-frontends-advanced-techniques-for-6f07a781f93d](https://roshancloudarchitect.me/identifying-and-eliminating-code-smells-in-angular-micro-frontends-advanced-techniques-for-6f07a781f93d) (Section 3)
- [https://zydesoft.com/must-know-clean-code-principles-in-angular/](https://zydesoft.com/must-know-clean-code-principles-in-angular/) (Sections 7 and 12)

> [!Note]
> See also:
>
> - [Angular University – Smart vs Presentational Components][1]
> - [Clean Code Principles in Angular][3]
> - [Stackademic – Smart/Dumb Components in Angular][4]
> - [Composable & Pure Components (Jack the Nomad)][5]
> - [Scalable Apps with Smart vs Dumb Components][6]

[1]: https://blog.angular-university.io/angular-2-smart-components-vs-presentation-components-whats-the-difference-when-to-use-each-and-why/
[3]: https://zydesoft.com/must-know-clean-code-principles-in-angular/
[4]: https://blog.stackademic.com/angular-smart-dumb-components-118f557b667c
[5]: https://jackthenomad.com/how-to-write-good-composable-and-pure-components-in-angular-2-1756945c0f5b
[6]: https://tejas-variya.medium.com/smart-vs-dumb-components-in-angular-the-secret-to-scalable-apps-49c2f49103eb

