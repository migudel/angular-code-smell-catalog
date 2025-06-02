# God Component

## Description

A **God Component** in Angular is one that takes on too many responsibilities. It often handles data fetching, state management, event handling, complex business logic, and UI rendering all within a single component. This violates the **Single Responsibility Principle (SRP)** and hinders both scalability and testability.

> [!Note]
> This code smell often appears alongside [`Mixing Smart and Dumb Components`](mixing_smart_and_dump_components.md), due to their high similarity and conceptual overlap.
> The key difference is that **God Component** refers to an abuse of responsibilities within a single component, while *Mixing Smart and Dumb Components* refers to mixing component roles—combining presentation-only components (dumb) with those responsible for orchestration and data management (smart).

## Why This Is a Code Smell

- **Limited reusability**: When a component handles multiple concerns, its logic becomes harder to extract and reuse across the application.
- **Complicated testing**: Testing a component with many responsibilities requires mocking or stubbing many unrelated behaviors.
- **Poor maintainability**: Large, monolithic components are harder to read, navigate, and safely refactor.
- **Violation of the Single Responsibility Principle**: Mixing unrelated responsibilities in a single component leads to fragile, error-prone code.
- **Tight coupling between layers**: Business logic and UI are intertwined, reducing flexibility and separation of concerns.
- **Increased risk of hidden bugs**: Oversized components can contain subtle issues that are harder to isolate and fix.

---

## Non-Compliant Code Example

```ts
@Component({
  selector: 'app-dashboard',
  templateUrl: './dashboard.component.html'
})
export class DashboardComponent implements OnInit {
  users: User[] = [];
  searchTerm = '';
  isLoading = false;

  constructor(private http: HttpClient, private router: Router) {}

  ngOnInit(): void {
    this.isLoading = true;
    this.http.get<User[]>('/api/users').subscribe(data => {
      this.users = data;
      this.isLoading = false;
    });
  }

  search(): void {
    this.isLoading = true;
    this.http.get<User[]>(`/api/users?q=${this.searchTerm}`).subscribe(data => {
      this.users = data;
      this.isLoading = false;
    });
  }

  goToDetail(userId: number): void {
    this.router.navigate(['/users', userId]);
  }

  trackById(index: number, user: User): number {
    return user.id;
  }
}
```

---

## Compliant Code Example

```ts
// user.service.ts
@Injectable({ providedIn: 'root' })
export class UserService {
  constructor(private http: HttpClient) {}

  getUsers(query: string = ''): Observable<User[]> {
    return this.http.get<User[]>(`/api/users${query ? `?q=${query}` : ''}`);
  }
}
```

```ts
// dashboard.component.ts
@Component({
  selector: 'app-dashboard',
  templateUrl: './dashboard.component.html',
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class DashboardComponent implements OnInit {
  users$: Observable<User[]>;
  searchTerm = '';
  isLoading = false;

  constructor(private userService: UserService, private router: Router) {}

  ngOnInit(): void {
    this.loadUsers();
  }

  loadUsers(): void {
    this.isLoading = true;
    this.users$ = this.userService.getUsers(this.searchTerm).pipe(
      finalize(() => this.isLoading = false)
    );
  }

  goToDetail(userId: number): void {
    this.router.navigate(['/users', userId]);
  }

  trackById(index: number, user: User): number {
    return user.id;
  }
}
```

---

## Sources

- [https://dev.to/this-is-angular/7-deadly-sins-of-angular-1n2j](https://dev.to/this-is-angular/7-deadly-sins-of-angular-1n2j) (4th sin)
- [https://codeburst.io/angular-bad-practices-eab0e594ce92](https://codeburst.io/angular-bad-practices-eab0e594ce92) (1st section: *Not making a REAL use of Angular’s components*)
- [https://javascript-conference.com/blog/angular-code-smells/](https://javascript-conference.com/blog/angular-code-smells/) (5th section: *Injecting services*)
- [https://medium.com/codex/avoid-these-bad-practices-when-you-are-an-angular-developer-135323db74c7](https://medium.com/codex/avoid-these-bad-practices-when-you-are-an-angular-developer-135323db74c7) (1st section: *Write external system interactions inside component*)
- [https://www.freecodecamp.org/news/best-practices-for-a-clean-and-performant-angular-application-288e7b39eb6f/](https://www.freecodecamp.org/news/best-practices-for-a-clean-and-performant-angular-application-288e7b39eb6f/) (13th practice)
- [https://www.tatvasoft.com/outsourcing/2021/07/top-angular-developer-pitfalls.html](https://www.tatvasoft.com/outsourcing/2021/07/top-angular-developer-pitfalls.html) (Section 1.2)
- [https://levelup.gitconnected.com/refactoring-angular-applications-be18a7ee65cb](https://levelup.gitconnected.com/refactoring-angular-applications-be18a7ee65cb) (Section 4.1)
