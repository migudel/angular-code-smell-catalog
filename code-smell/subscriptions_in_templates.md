A continuación tienes un resumen y la descripción completa del code smell **“Manual subscriptions (suscripciones manuales en plantillas HTML)”** siguiendo la estructura solicitada.

En Angular, suscribir manualmente un observable directamente en la plantilla (por ejemplo, usando `(click)="obs$.subscribe(...)"`) en lugar de emplear el **`async` pipe** o manejar la suscripción desde el componente produce un **anti-patrón** que obliga a controlar manualmente el ciclo de vida de la suscripción, con alto riesgo de fugas de memoria, generación de múltiples suscripciones redundantes y código difícil de mantener ([Medium][1])([Infinum][2])([GeeksforGeeks][3]).

# Manual subscriptions en plantillas HTML

## Description

Cuando suscribes un observable directamente en la plantilla—por ejemplo en un binding de evento como `(click)="data$​.subscribe(d => …)"`—cada interacción crea una nueva suscripción que nunca se cancela automáticamente, ya que no existe lógica en el componente que las gestione ([Infinum][2]). Esto contraviene la recomendación de Angular de utilizar el **`async` pipe**, que internamente se encarga de subscribir y desubscribir de forma segura cuando el componente se destruye ([GeeksforGeeks][3]).

## Why is a code smell

* **Fugas de memoria:** Cada suscripción manual sin `unsubscribe()` persiste en memoria hasta que el observable completa o el proceso finaliza, lo que con el tiempo incrementa el consumo de recursos ([Stack Overflow][4]).
* **Suscripciones redundantes:** Acciones repetidas (clicks, renderizados) generan múltiples suscripciones simultáneas al mismo flujo de datos, pudiendo disparar lógica varias veces de forma inesperada ([Medium][1]).
* **Código verboso y disperso:** Al manejar lógica de suscripción en la plantilla, pierde claridad la ubicación del `unsubscribe()`, dificultando la lectura y el mantenimiento del código ([Infinum][2]).
* **Se rompe el ciclo de vida de Angular:** Las plantillas no disponen de hooks (`ngOnDestroy`), por lo que no hay punto natural para cancelar la suscripción cuando el componente se destruye ([Stack Overflow][5]).

## Non-Compliant code example(s)

```html
<!-- example.component.html -->
<button (click)="items$​.subscribe(items => onItems(items))">
  Cargar ítems
</button>
```

En este fragmento, cada vez que el usuario hace clic se crea una nueva suscripción a `items$` que nunca se desuscribe, acumulando subscripciones en memoria ([Infinum][2]).

## Compliant code example(s)

### 1. Usando `async` pipe en la plantilla

```html
<!-- example.component.html -->
<button (click)="onItems(items$ | async)">
  Cargar ítems
</button>
```

El **`async` pipe** gestiona automáticamente la suscripción y la cancelación cuando el componente se destruye, evitando fugas de memoria y simplificando el código ([GeeksforGeeks][3]).

### 2. Suscribir en el componente y cancelar en `ngOnDestroy`

```typescript
// example.component.ts
import { Component, OnDestroy } from '@angular/core';
import { Subscription } from 'rxjs';
import { DataService } from './data.service';

@Component({ /* ... */ })
export class ExampleComponent implements OnDestroy {
  private sub!: Subscription;

  constructor(private ds: DataService) {}

  onLoad() {
    this.sub = this.ds.getItems()
      .subscribe(items => this.handleItems(items));
  }

  ngOnDestroy() {
    this.sub.unsubscribe();
  }
}
```

Aquí la suscripción se maneja en el componente, lo que permite llamar a `unsubscribe()` en `ngOnDestroy` y controlar el ciclo de vida adecuadamente ([Stack Overflow][5]).

### 3. Patrón `takeUntil` para cancelar múltiples suscripciones

```typescript
// example.component.ts
import { Component, OnDestroy } from '@angular/core';
import { Subject } from 'rxjs';
import { takeUntil } from 'rxjs/operators';
import { DataService } from './data.service';

@Component({ /* ... */ })
export class ExampleComponent implements OnDestroy {
  private destroy$ = new Subject<void>();

  loadItems() {
    this.ds.getItems()
      .pipe(takeUntil(this.destroy$))
      .subscribe(items => this.handleItems(items));
  }

  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

El operador `takeUntil` automáticamente cancela todas las suscripciones ligadas cuando `destroy$` emite, eliminando el riesgo de fugas de memoria y simplificando la gestión de múltiples flujos ([Medium][1]).

[1]: https://medium.com/%40ConorJonOReilly/subscriptions-practices-to-avoid-in-angular-4eefa0316727?utm_source=chatgpt.com "Subscriptions & practices to avoid in Angular | by Conor O'Reilly"
[2]: https://infinum.com/handbook/frontend/angular/angular-guidelines-and-best-practices/formatting-naming-and-best-practices?utm_source=chatgpt.com "Frontend Handbook | Angular / Angular guidelines and best ..."
[3]: https://www.geeksforgeeks.org/difference-between-subscribe-and-async-pipe/?utm_source=chatgpt.com "Difference Between subscribe() and async pipe. - GeeksforGeeks"
[4]: https://stackoverflow.com/questions/66222400/is-it-really-better-to-use-async-pipe-instead-of-subscribe-in-this-particular?utm_source=chatgpt.com "Is it really better to use async-pipe instead of subscribe() in this ..."
[5]: https://stackoverflow.com/questions/60552742/in-angular-where-to-subscribe-in-service-or-in-component-why?utm_source=chatgpt.com "In Angular where to subscribe in service or in component & why?"
