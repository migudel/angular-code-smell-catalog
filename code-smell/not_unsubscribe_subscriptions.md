# Not Unsubscribing Subscriptions

## Description

This code smell occurs when components subscribe to observables manually but fail to unsubscribe before the component is destroyed. This leads to **memory leaks** or unexpected behavior, as callbacks may continue to trigger even after the view is gone.

Over time, these "zombie" subscriptions accumulate and consume unnecessary resources, degrading the application's performance.

## Why This Is a Code Smell

- **Memory retention**: Each active subscription prevents the garbage collector from releasing associated memory, causing steady memory growth.
- **Performance degradation**: Multiple active subscriptions slow down the application and may trigger callbacks on destroyed components.
- **Hard to debug**: Identifying memory leaks due to forgotten subscriptions is especially challenging in large applications with multiple subscription points.

---

## Non-Compliant Code Example

```ts
@Component({
  selector: 'app-example',
  template: `<p>{{ data }}</p>`,
})
export class ExampleComponent implements OnInit {
  data: any;

  constructor(private dataService: DataService) {}

  ngOnInit() {
    this.dataService.getData()
      .subscribe(value => {
        this.data = value;
      });
  }
}
```

---

## Compliant Code Examples

### Manual unsubscription in `ngOnDestroy`

```ts
@Component({
  selector: 'app-example',
  template: `<p>{{ data }}</p>`,
})
export class ExampleComponent implements OnInit, OnDestroy {
  data: any;
  private sub!: Subscription;

  constructor(private dataService: DataService) {}

  ngOnInit() {
    this.sub = this.dataService.getData()
      .subscribe(value => {
        this.data = value;
      });
  }

  ngOnDestroy() {
    this.sub.unsubscribe();
  }
}
```

### Managing multiple subscriptions

```ts
@Component({
  selector: 'app-component',
  templateUrl: './app.component.html',
})
export class AppComponent implements OnInit, OnDestroy {
  subscription = new Subscription();

  ngOnInit(): void {
    this.subscription.add(
      interval(500).subscribe(x => console.log(`A: ${x}`))
    );
    this.subscription.add(
      interval(700).subscribe(x => console.log(`B: ${x}`))
    );
  }

  ngOnDestroy(): void {
    this.subscription.unsubscribe();
  }
}
```

### Using `takeUntilDestroyed` (Angular 16+)

```ts
@Component({
  selector: 'app-example',
  template: `<p>{{ displayText }}</p>`,
})
export class ExampleComponent implements OnInit {
  displayText: any;

  ngOnInit(): void {
    myObservable$.pipe(
      map(value => value.item),
      take(1),
      takeUntilDestroyed()
    ).subscribe(item => this.displayText = item);
  }
}
```

### Using the `AsyncPipe` in Templates

Using the `async` pipe directly in the template handles both subscription and unsubscription automatically:

```ts
@Component({
  selector: 'app-example',
  template: `<p>{{ myObservable$ | async }}</p>`,
})
export class ExampleComponent {
  myObservable$ = this.dataService.getData().pipe(map(data => data.item));
}
```

> [!Note]
> See also:
>
> - [Use `async` pipe when possible][1]
> - [`takeUntilDestroyed` in Angular v16 (Angular Love)][2]
> - [Angular Docs – takeUntilDestroyed()][3]
> - [Exploring takeUntilDestroyed (Netanel Basal)][4]

---

## Sources

- [https://marcoslooten.com/blog/4-common-angular-mistakes/](https://marcoslooten.com/blog/4-common-angular-mistakes/) (Section 1)
- [https://alex-klaus.com/angular-code-review/](https://alex-klaus.com/angular-code-review/) (Section 3)
- [https://medium.com/codex/avoid-these-bad-practices-when-you-are-an-angular-developer-135323db74c7](https://medium.com/codex/avoid-these-bad-practices-when-you-are-an-angular-developer-135323db74c7) (Section 3)
- [https://www.freecodecamp.org/news/best-practices-for-a-clean-and-performant-angular-application-288e7b39eb6f/](https://www.freecodecamp.org/news/best-practices-for-a-clean-and-performant-angular-application-288e7b39eb6f/) (Section 6)
- [https://www.tatvasoft.com/outsourcing/2021/07/top-angular-developer-pitfalls.html](https://www.tatvasoft.com/outsourcing/2021/07/top-angular-developer-pitfalls.html) (Section 1.1)
- [https://chudovo.com/most-common-angular-mistakes-every-developer-should-avoid/](https://chudovo.com/most-common-angular-mistakes-every-developer-should-avoid/) (Section 4)
- [https://levelup.gitconnected.com/refactoring-angular-applications-be18a7ee65cb](https://levelup.gitconnected.com/refactoring-angular-applications-be18a7ee65cb) (Section 2.3)
- [https://blog.brecht.io/rxjs-best-practices-in-angular/](https://blog.brecht.io/rxjs-best-practices-in-angular/) (Section 5)
- [https://www.slideshare.net/slideshow/rxjs-best-bad-practices-for-angular-developers/233392471](https://www.slideshare.net/slideshow/rxjs-best-bad-practices-for-angular-developers/233392471) (3rd Bad Practice)
- [https://www.sourceallies.com/2020/11/state-management-anti-patterns/](https://www.sourceallies.com/2020/11/state-management-anti-patterns/) (Section 3)
- [https://medium.com/@OPTASY.com/what-are-the-5-most-common-angular-mistakes-that-developers-make-53f6d7c5bf65](https://medium.com/@OPTASY.com/what-are-the-5-most-common-angular-mistakes-that-developers-make-53f6d7c5bf65) (Section 2)
- [https://zydesoft.com/must-know-clean-code-principles-in-angular/](https://zydesoft.com/must-know-clean-code-principles-in-angular/) (Section 5)

[1]: https://blog.eyas.sh/2018/12/use-asyncpipe-when-possible/
[2]: https://angular.love/takeuntildestroy-in-angular-v16
[3]: https://angular.dev/api/core/rxjs-interop/takeUntilDestroyed
[4]: https://medium.com/netanelbasal/getting-to-know-the-takeuntildestroyed-operator-in-angular-d965b7263856
