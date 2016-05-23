Angular JS (1.X)
================

## Qué es Angular
- Es un framework para dar estructura a las aplicaciones web, especialmente las SPA (Single Page Applications o aplicaciones de una sola página). Se basa en el patrón MVC (Model - View - Controller) pero con algunas diferencias e introduce terminología propia.
- Está orientado a aplicaciones donde predominan las operaciones CRUD.
- "Extiende" HTML para permitir agregar interactividad a las páginas en forma declarativa.
- Ayuda a modularizar el código de la página (JS y HTML), liberarse de las marañas de callbacks, obtener y enviar datos desde y hacia servicios externos de manera más práctica y prolija, y minimizar la manipulación directa del DOM, evitando, por ejemplo, que se haga extremadamente difícil entender por qué se está disparando ese evento misterioso y dónde está el código del handler que no está haciendo lo que debe.
- Permite crear componentes reutilizables (que por supuesto sólo se pueden utilizar en aplicaciones de Angular).
- Facilita el testing del código.

## Qué no es Angular
- **No es una biblioteca de animaciones y cosas lindas**. Es muy común asociar Angular u otros frameworks similares con "páginas que se vean geniales", pero la verdad es que Angular no está orientado a eso ni provee grandes cosas en ese aspecto.
- No es meramente un reemplazo para jQuery, sino que es algo mucho más grande, que contiene un subset de jQuery (y de hecho soporta la versión completa de jQuery de ser necesaria). Sin embargo, nos evitará en gran parte tener que recurrir a jQuery, y encapsular cualquier uso de dicha biblioteca en un tipo determinado de estructuras de Angular, llamadas *directives*.
- No es una biblioteca para hacer web components, aunque en Horizon, en cierta medida, se utiliza de esa manera. Se puede utilizar una arquitectura basada en componentes -de hecho es recomendable- pero sólo se pueden usar los componentes en el marco de una aplicación de Angular. En Horizon aún se mezclan estilos de programación y encontramos código de Angular mezclado con variables globales...

## Conceptos importantes

### Modelos
En Angular, los modelos son POJOs (Plain Old JavaScript Objects), lo que significa que no se necesita ningún objeto especial, ni heredar ningún prototipo en particular. Cualquier objeto sirve como modelo. La ventaja de esto es la flexibilidad, y la desventaja es que es difícil mantener una representación consistente de los modelos, ya que ni siquiera hace falta definir una estructura para los mismos, y todo el manejo de modelos queda en manos del programador.

Dicho esto, hay un tipo de objeto que es esencial para representar el modelo en la vista y también reflejar en el modelo los cambios en la vista: el Scope.

### Scope
Uno de los conceptos más importantes de Angular es el scope. En parte, es algo similar al *context* en Django. Pero en Angular, los scopes se pueden anidar, y en ellos se pueden registrar *watches*, que escuchan cambios en el modelo y disparan callbacks cuando estos ocurren. También pueden emitir eventos a otros scopes y escuchar los provenientes de los demás. **TODO: ejemplos**

#### Sintaxis "controller as"
A partir de Angular 1.3 ya no es necesario inyectar el servicio *$scope* en los controllers, gracias a la llamada "sintaxis controller as", que consiste en declarar en el html el controller de la siguiente manera:

```HTML
<div ng-controller="StuffController as myCtrl">
```

Esto hace que se guarde la referencia al controller con el nombre *myCtrl* y tiene varias ventajas:
- Toda referencia al scope de este controller en el HTML deberá estar precedida de `myCtrl.` (por ejemplo, si tenemos un miembro llamado *variable*, lo podríamos mostrar usando la expresión `{{myCtrl.variable}}` en lugar de solo `{{variable}}`). Esto es muy útil cuando hay controllers anidados, ya que evitamos conflictos de nombres.
- En el JS, en lugar de usar `$scope`, podemos usar directamente `this`.

### Views
En Angular las Views son en realidad meramente lo que en Django llamamos Templates. Lo peculiar de Angular en este aspecto es que lenguaje de templating de Angular es directamente HTML.

#### Expresiones y one-time bindings
En cualquier parte de la vista podemos evaluar y mostrar el resultado de una expresión entre dobles llaves (`{{}}`) [**Observación: en Horizon será `{%%}` por compatibilidad con Django**]. Es importante tener en cuenta que al hacer esto, por defecto, se va a crear un *watch* para cada expresión. El problema es que con cada binding, el ciclo que hace Angular para buscar y procesar cambios (el llamado *digest cycle*) se vuelve más pesado. Se calcula que Angular soporta hasta 2000 bindings simultaneos, lo cual no es tanto como parece. Para aligerar este problema se agregó en Angular 1.3 la posibilidad de hacer bindings que se procesen sólo la primera vez. La manera de hacer esto es agregando `::` antes de la expresión, después de las primeras dos llaves. Por ejemplo, `{{variable}}` hará un binding normal que se actualizará cada vez que la variable `variable` del scope actual cambie, mientras que `{{::variable}}` tendrá fijo el primer valor que tome dicha variable.

#### ngModel
ngModel es la directive esencial para los formularios en Angular, porque es la que establece el binding entre el valor de un input en la vista y el scope. Sin ngModel, las validaciones no funcionan y el scope no mantiene el valor de los campos del formulario.

HTML:

```HTML
<div ng-controller="myController as ctrl">
	<form>
		<input type="text" ng-model="ctrl.nombre" name="nombre">
	</form>
</div>
```

JS:

```JavaScript
...
.controller('myController',function(){
	this.nombre = "Pablito";
});
```

En el ejemplo, se vincula el atributo "nombre" del scope con el campo "nombre" del formulario a través de ngModel. Será inicializado con el nombre "Pablito", pero cada vez que se modifique el texto del input, cambiará el valor de la variable "nombre" en el controller.

### Controllers
Los controllers sirven para vincular el modelo con la vista, y son los que saben cómo manejar los eventos provenientes de la vista.

Lo que en Django llamamos View, en Angular (y en la mayoría de los frameworks que no sean Django) es un Controller.

En general, como en el caso de Angular el vínculo entre vista y modelo es bidireccional, en la jerga se habla de ViewModel en lugar de Controller, pero Angular como framework conserva el nombre "original".

Como todos los otros componentes, un Controller es una función. Recibe como parámetros los servicios que le son inyectados. Para inyectar estos servicios hay varias alternativas, pero las recomendables son dos:
- Pasarle al método `controller` el nombre del controller y un array de N elementos donde los primeros N-1 elementos sean los nombres de los servicios (como strings) a inyectar, más un último elemento que sea la función del controlador, como en este ejemplo:

```JavaScript
angular
	.module('billingApp')
	.controller('reportController',['$scope', '$http', '$filter', 'nubeliuSpinnerService', 'billingApp.reportDataService', '$attrs', '$q',
	function($scope, $http, $filter, spinner, $data, $attrs, $q){
		...
	}]);
```

- Usar la propiedad $inject de la función del controlador. Por ejemplo:

```JavaScript
var billingModule = angular.module('billingApp');
var ReportController = function($scope, $http, $filter, spinner, $data, $attrs, $q) {
	...
};
ReportController.$inject = ['$scope', '$http', '$filter', 'nubeliuSpinnerService', 'billingApp.reportDataService', '$attrs', '$q'];
billingModule.controller('reportController',ReportController);
```

La segunda alternativa es más verbosa, pero parece ser la preferida por los developers del core de Horizon.

### Services
Los servicios en Angular son objetos Singleton que pueden ser usados en otros componentes (controllers, directives, filters, otros services) pasándoles una referencia mediante el mecanismo de Dependency Injection.

El acceso a datos **siempre** debe ser realizado a través de services, sean estos los provistos por Angular que veremos en la próxima sección, o services propios.

Hay varias maneras de crear un service. La más recomendable, por combinar facilidad con flexibilidad, es utilizar el método `factory`. Esta manera de crear un servicio requiere retornar un objeto que contenga los métodos del service implementado. Por ejemplo:

```JavaScript
	angular
		.module('billingApp')
		.factory('unServicio',[function(){
			return {
				hola: function(nombre){
						return 'hola' + nombre;
				}
			};
	}]);
```
#### Servicios built-in importantes
- $scope: Es importante para definir *watches* y aplicar cambios que no surjan de APIs de Angular (por ejemplo, si hacemos algún cambio en el DOM con jQuery) con `$scope.$apply`.
- $http: Sirve para hacer HTTP Requests asíncronas. Usa Promises.
- $q: Para construir Promises.
- $attrs: permite acceder a los atributos de un elemento (sirve por ejemplo para pasar argumentos extra a un controller)

### Directives
Las directives son, en pocas palabras, las que hacen a Angular interesante y al mismo tiempo uno de sus mayores problemas, por culpa del pobre diseño de la forma de definirlas. **TODO: Es un tema complejo, por ahora lo hablo nada más**

- restrict (E [element], A [attribute], C [class], M [comment])
- template
- templateUrl
- transclude: es necesario si queremos hacer cosas con el contenido (los hijos) del elemento de la directive
- scope
- require: para limitar el uso de la directive al caso en el que se encuentre en el scope de un cierto controller
- replace

#### Directives built-in importantes
Nota: los nombres dados son los nombres en JavaScript, escritos en camelCase. En HTML cambian los nombres y usan snake-case. Por ejemplo, `ngApp` se vuelve `ng-app`.
- ngApp: define el elemento raíz de la aplicación
- ngController: vincula la vista con un controlador
- ngRepeat: itera sobre una lista
- ngSrc: sirve para asignar dinámicamente el atributo src de una imagen
- ngClass: asigna clases de CSS según atributos del scope
- ngModel: establece two-way bindings entre inputs y el scope
- ngChange: dispara una expresión cuando el input cambia
- ngShow y ngHide: muestran u ocultan dinámicamente el elemento al cual modifican según el valor de una expresión
- ngIf: agrega o elimina del DOM el elemento al cual modifica según el valor de una expresión (en lugar de solo mostrar u ocultar)

### Filters
Como los de Django, aplican una función al resultado de una expresión. En Angular, además, los parámetros del filter pueden ser expresiones y actualizarse cuando cambie su resultado. Normalmente son funciones "puras", o sea, reciben un valor y parámetros y retornan un valor, sin efectos secundarios, y para cierta entrada siempre se obtiene la misma salida (*pueden* hacerse filtros "stateful", pero no es recomendable).
Ejemplo:

		{{objeto_date | date:'yyyy-MM-dd HH:mm'}} //2016-05-23 12:37

#### Filters built-in importantes
- date: formatea fechas
- currency: formatea valores de dinero
- orderBy: para ordenar listas
- filter: filtra los datos de una lista según una expresión

----------------

# Angular JS en OpenStack

## Expresiones
**IMPORTANTE**: Para permitir la compatibilidad con Django, las expresiones de Angular en Horizon **no van entre `{{}}`**, sino entre **`{%%}`**.

## Módulo principal de Horizon

### Kilo

En **OpenStack Kilo**, el módulo principal (la *app*) lleva el nombre **hz**. Este módulo está definido en `horizon/static/horizon/js/angular/horizon.js` y el resto de los archivos relativos a Angular se distribuyen entre la carpeta `horizon/static/horizon/js/angular` y la carpeta `horizon/static/angular`.

En todas las páginas encontramos el tag body de la siguiente manera:

```HTML
<body ng-app="hz" class="ng-scope">
```

Dada esta situación, tenemos dos alternativas:

- Extender con nuestro código el módulo **hz**, o
- Importar en el módulo **hz** los módulos que nosotros escribamos

Es evidente que la segunda opción es mucho mejor que la primera, ya que evitamos conflictos y mantenemos bien claro dónde está todo el código que pertenece a cada módulo. El problema con la segunda opción es que hay que agregar manualmente nuestros módulos en algún archivo *enabled* para que la app **hz** sepa que tiene que cargarlos, dado que no podemos (o no debemos) agregarle directamente nuestro módulo como dependencia a la app **hz** en su declaración.

### Liberty y posteriores
A partir de Liberty, el módulo raíz ha pasado a llamarse **horizon.framework** y también cambiaron completamente la estructura de carpetas. En esta versión, los archivos generales de Angular se ubican en la carpeta `horizon/static/framework`, pero la app de la página se llama **horizon.app** y el módulo principal se encuentra en `openstack_dashboard/static/app/app.module.js`. En `openstack_dashboard/static/app/` hay otros archivos que se usan en el nuevo dashboard. Toda esta nueva estructura implica probablemente una incompatibilidad difícil de resolver.

En Mitaka parece estar todo bastante bien organizado y la [Guía oficial] explica todo lo necesario en cuanto a la estructura del código JS.

## Importar módulos de Angular

Hay que indicarle a Horizon los módulos de Angular que debe cargar. Esto debe hacerse en el enabled del módulo correspondiente, pero teniendo cuidado que no aparezca agregado más de una vez entre los distintos enabled, porque si esto sucede Horizon no arranca.

En **billing-ui** está hecho de la siguiente manera:

#### `billing-ui/billing_ui/enabled/showback/_02_billing_reports.py`

```Python
# A list of angular modules to be added as dependencies to horizon app.
ADD_ANGULAR_MODULES = ['billingApp']

# A list of javascript files to be included for all pages
ADD_JS_FILES = ['billing_ui/js/app.js']
```

## Tests

### Unit tests
Los tests unitarios se escriben en archivos con extensión `.spec.js`. Por ejemplo, si tenemos un archivo llamado `servicio.js`, los tests del código de ese archivo deben estar en el mismo directorio, en `servicio.spec.js`.

Los archivos `.spec.js` se pueden manejar automáticamente si tenemos
```Python
AUTO_DISCOVER_STATIC_FILES = True
```
en el archivo *enabled*.

### Tests "End To End"
TODO

### Correr los tests
Para correr los tests se puede acceder al test runner desde el browser en `<raíz de horizon>/jasmine/` (por ejemplo: `http://127.0.0.1:8000/jasmine/`). **En nuestro horizon-kilo están faltando algunos archivos core de jasmine, así que los tests no funcionan**.

En versiones más nuevas de OpenStack los tests se pueden correr sin usar el browser (ver [Guía oficial]). Es necesario tener instalado nodejs y npm para hacer esto.

Links
=====
[Guía Oficial de Angular en OpenStack][Guía oficial] (Aplica a versiones recientes solamente)

[Guía oficial]:http://docs.openstack.org/developer/horizon/topics/angularjs.html
