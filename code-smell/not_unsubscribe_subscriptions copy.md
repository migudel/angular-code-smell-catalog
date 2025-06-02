# Memory Leaks: Not Unsubscribing Subscriptions

## Description

En aplicaciones Angular, una **fuga de memoria** por **no cancelar suscripciones** ocurre cuando un componente se suscribe a un observable y no llama a `unsubscribe()` antes de que el componente se destruya, manteniendo vivo el objeto de la suscripción en memoria incluso después de que la vista haya sido eliminada ([Level Up Coding][1]). Estas suscripciones huérfanas pueden acumularse gradualmente a medida que los usuarios navegan por la aplicación, consumiendo recursos innecesariamente y degradando el rendimiento ([Angular Training][2]).

## Why is a code smell

1. **Retención de memoria:** Cada suscripción no cancelada impide que el recolector de basura libere los recursos asociados, provocando un incremento constante del uso de memoria ([Medium][3]).
2. **Impacto en rendimiento:** Con el tiempo, el exceso de suscripciones activas ralentiza la aplicación y puede originar comportamientos inesperados, como callbacks ejecutándose en componentes destruidos ([LinkedIn][4]).
3. **Difícil de depurar:** Identificar y localizar fugas de memoria por suscripciones olvidadas es complejo, especialmente en aplicaciones grandes con múltiples puntos de suscripción ([Stack Overflow][5]).

## Non-Compliant code example(s)

```typescript
import { Component, OnInit } from '@angular/core';
import { DataService } from './data.service';

@Component({
  selector: 'app-example',
  template: `<p>{{ data }}</p>`,
})
export class ExampleComponent implements OnInit {
  data: any;

  constructor(private dataService: DataService) {}

  ngOnInit() {
    // Suscripción sin cancelación posterior
    this.dataService.getData()
      .subscribe(value => {
        this.data = value;
      });
  }

  // No hay ngOnDestroy ni unsubscribe()
}
```

Este fragmento muestra una suscripción en `ngOnInit` sin el correspondiente `unsubscribe()` en `ngOnDestroy`, lo que provoca la fuga de memoria ([Level Up Coding][1]).

## Compliant code example(s)

### 1. Cancelar manualmente en `ngOnDestroy`

```typescript
import { Component, OnInit, OnDestroy } from '@angular/core';
import { Subscription } from 'rxjs';
import { DataService } from './data.service';

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
    // Cancelación explícita de la suscripción
    this.sub.unsubscribe();
  }
}
```

Esta práctica previene la fuga de memoria al liberar la suscripción cuando el componente se destruye ([Level Up Coding][1]).

### 2. Usar patrón `takeUntil`

```typescript
import { Component, OnInit, OnDestroy } from '@angular/core';
import { Subject } from 'rxjs';
import { takeUntil } from 'rxjs/operators';
import { DataService } from './data.service';

@Component({
  selector: 'app-example',
  template: `<p>{{ data }}</p>`,
})
export class ExampleComponent implements OnInit, OnDestroy {
  data: any;
  private destroy$ = new Subject<void>();

  constructor(private dataService: DataService) {}

  ngOnInit() {
    this.dataService.getData()
      .pipe(takeUntil(this.destroy$))
      .subscribe(value => {
        this.data = value;
      });
  }

  ngOnDestroy() {
    // Emite y completa para cancelar todas las suscripciones ligadas
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

El operador `takeUntil` automatiza la cancelación de múltiples suscripciones y es especialmente útil en componentes con varios observables ([Level Up Coding][1]).

### 3. Utilizar el pipe `async` en la plantilla

```typescript
// En el componente
data$ = this.dataService.getData();

// En la plantilla HTML
<p>{{ data$ | async }}</p>
```

El **async pipe** maneja automáticamente la suscripción y la cancelación al destruir el componente, reduciendo la necesidad de lógica manual en el código TypeScript ([Angular Training][2]).

[1]: https://levelup.gitconnected.com/preventing-angular-subscription-memory-leaks-65f18eb9cb8e?utm_source=chatgpt.com "Preventing Angular Subscription Memory Leaks - Level Up Coding"
[2]: https://www.angulartraining.com/daily-newsletter/how-to-avoid-memory-leaks-with-rxjs-observables/?srsltid=AfmBOor-nqkgF5q-D9x9wfnz51oyPwRYJ5zKz1e-_yZzd9sMTyAevkUN&utm_source=chatgpt.com "How to avoid memory leaks with RxJs observables? - Angular Training"
[3]: https://medium.com/frontend-simplified/how-i-handle-subscriptions-in-angular-to-avoid-memory-leaks-fbc00ac34616?utm_source=chatgpt.com "How I handle subscriptions in Angular to avoid memory leaks"
[4]: https://www.linkedin.com/pulse/prevent-observable-memory-leaks-angular-rohtash-sethi-mud9c?utm_source=chatgpt.com "Prevent Observable Memory Leaks in Angular - LinkedIn"
[5]: https://stackoverflow.com/questions/77274781/rxjs-subscriptions-in-services-memory-leak?utm_source=chatgpt.com "RxJs subscriptions in services... Memory leak? - Stack Overflow"
