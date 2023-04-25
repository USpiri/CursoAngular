### **Angular Configuration (angular.json)**

Es solo una recomendación disponible para usar a la hora de compilar nuestra aplicación.   
Reemplaza el archivos de environments por environments.local en caso de que se trabaje en desarrollo y environments.prod para producción.

```json
"configurations": {
    "development": {
        "budgets": [
            {
                "type": "initial",
                "maximumWarning": "2mb",
                "maximumError": "5mb"
            }
        ],
        "fileReplacements": [
            {
                "replace": "src/environments/environment.ts",
                "with": "src/environments/environment.local.ts"
            }
        ],
        "optimization": false,
        "buildOptimizer": false,
        "vendorChunk": true,
        "extractLicenses": false,
        "sourceMap": true,
        "namedChunks": true
    },
    "production": {
        "budgets": [
            {
                "type": "initial",
                "maximumWarning": "2mb",
                "maximumError": "5mb"
            }
        ],
        "fileReplacements": [
            {
                "replace": "src/environments/environment.ts",
                "with": "src/environments/environment.prod.ts"
            }
        ],
        "optimization": true,
        "buildOptimizer": true,
        "vendorChunk": false,
        "outputHashing": "all",
        "extractLicenses": true,
        "sourceMap": false,
        "namedChunks": false
    }
}
```

## **Angular Standalone Components**

El término "standalone" se refiere a componentes, directivas o pipes que pueden ser usadas independientemente de un NgModule. En esta página se encuentra el funcionamiento y creación de ellos. [Angular Standalone Components](https://www.telerik.com/blogs/angular-14-introducing-standalone-components)

## **Barrels**

Para ayudar a la organización del proyecto podemos utilizar conceptos como *"Barrels"*. La principal idea es agregar un archivo llamado "index.ts" para cada una de las carpetas que contengan un archivo .ts. Dentro de este se exportará cada una de las variables exportables de los .ts permitiendo crear una especie de jerarquía que nos va a ayudar a acortar largas cadenas de imports.  
Son solo una recomendación ya que ellos no aparecen en la Angular Style Guide.

#### **Ejemplo:**  

Supongamos 3 componentes diferentes dentro de `./app/modules/home/component`:  

```ts
export class SomeComponent {} 
export class SomeClass {} 
export class SomeService {} 
```

Sin Barrels necesitaríamos 3 imports diferentes para cada uno de estos componentes:

```ts
import { SomeComponent } from '../component/some.component.ts';
import { SomeClass }     from '../component/some.model.ts'; 
import { SomeService }   from '../component/some.service.ts'; 
```

Agregando este barrel dentro de `./app/modules/home/component`  

```ts
export * from './some.model.ts';   // re-export all of its exports 
export * from './some.service.ts'; // re-export all of its exports 
export { SomeComponent } from './some.component.ts'; // re-export the named thing 
```

Ahora solo tendríamos que importar lo que necesitemos de este barrel  

```ts
import { SomeComponent, SomeService, SomeClass } from '../some'; // el archivo index está implicito
```

**Nota:** Esta [extensión](https://marketplace.visualstudio.com/items?itemName=mikehanson.auto-barrel) resulta muy útil para la implementacion de Barrels

## **Lazy Loading**

Una buena práctica es cargar de forma perezosa (lazy load) los módulos de nuestra aplicación siempre que sea posible. Obligando a que *SOLO* cargue lo necesario cuando sea necesario permitiendo que el tiempo de arranque de nuestra app disminuya considerablemente y nos brinde mayor seguridad al proyecto.  

#### **Sin Lazy Loading**

```ts
import ...;

const routes: Routes = [
  { path: '', redirectTo: 'home', pathMatch: 'full' },
  { path: 'home', component: HomeModule },
  { path: '**', redirectTo: 'home' }
];

@NgModule({...})
export class AppRoutingModule { }
```

#### **Con Lazy Loading**

```ts
import ...;

const routes: Routes = [
  { path: '', redirectTo: 'home', pathMatch: 'full' },
  { path: 'home', loadChildren: () => import('./modules').then( m => m.HomeModule ) },
  { path: '**', redirectTo: 'home' }
];

@NgModule({...})
export class AppRoutingModule { }
```

## **Observables**

Se pueden combinar los observables. Algunas veces nos encontraremos con que estamos llamando a más de un endpoint a la vez, entonces ¿Porqué no combinarlos?  
Esto es posible gracias a `combineLatest` de rxjs. De esta forma, incluso si alguno de los valores no llega, podremos acceder al resultado (o error) de la suscripción.


```tsx
import { combineLatest } from 'rxjs';

...ngOnInit() {

  combineLatest(
    this.yourService.getObservable1(),
    this.yourService.getObservable2()
  ).subscribe(
    ([result1, result2]) = {
      ...
    }
  )

}
```

De esta forma resulta mucho más fácil desuscribirnos en el ngOnDestroy.  

### **Unsuscribe**

Una buena recomendación es que (Por más que Angular por defecto se desuscriba de las peticiones HTTP automáticamente), si se sabe que el observable va a traer información una única vez desde el back, usar `.pipe(first())` despues de suscribirnos al componente. De esta forma una vez obtenidos los datos el sistema se desuscribe solo.

En caso de necesitar una constante actualización de los datos utilizaremos el ngOnDestroy para desuscribirse del observable.

```tsx
private subs: Subscription | undefined;

ngOnDestroy(){
  if (this.subs){
    this.subs.unsuscribe():
  }
}

private handleSubsOrder( orderId:string ){
  this.subs = this.cartCheckoutFacade.startOrderFulfilmentPolling(irderId)
}
```
