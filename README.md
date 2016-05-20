Angular JS (1.X)
================

## Modelos
En Angular, los modelos son POJOs (Plain Old JavaScript Objects), lo que significa que no se necesita ningún objeto especial, ni heredar ningún prototipo en particular. Cualquier objeto sirve como modelo. La ventaja de esto es la flexibilidad, y la desventaja es que es difícil mantener una representación consistente de los modelos, ya que ni siquiera hace falta definir una estructura para los mismos, y todo el manejo de modelos queda en manos del programador.

Dicho esto, hay un tipo de objeto que es esencial para representar el modelo en la vista y también reflejar en el modelo los cambios en la vista: el Scope.

## Scope
Uno de los conceptos más importantes de Angular es el scope. TODO

## Views
En Angular las Views son en realidad meramente lo que en Django llamamos Templates. Lo peculiar de Angular en este aspecto es que lenguaje de templating de Angular es directamente HTML.

## Controllers
Los controllers sirven para vincular el modelo con la vista, y son los que saben cómo manejar los eventos provenientes de la vista.

Lo que en Django llamamos View, en Angular (y en la mayoría de los frameworks que no sean Django) es un Controller.

En general, como en el caso de Angular el vínculo entre vista y modelo es bidireccional, en la jerga se habla de ViewModel en lugar de Controller, pero Angular como framework conserva el nombre "original".

	angular
		.module('billingApp')
		.controller('reportController',['$scope', '$http', '$filter', 'nubeliuSpinnerService', 'billingApp.reportDataService', '$attrs', '$q',
		function($scope, $http, $filter, spinner, $data, $attrs, $q){
			...
		}]);
## Services, Factories, Providers

## Angular JS en OpenStack

## Directives

## Módulo principal de Horizon
### Kilo

En **OpenStack Kilo**, el módulo principal (la *app*) lleva el nombre **hz**. Este módulo está definido en `horizon/static/horizon/js/angular/horizon.js` y el resto de los archivos relativos a Angular se distribuyen entre la carpeta `horizon/static/horizon/js/angular` y la carpeta `horizon/static/angular`. 

En todas las páginas encontramos el tag body de la siguiente manera:

	<body ng-app="hz" class="ng-scope">

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

#### billing-ui/billing_ui/enabled/showback/_02_billing_reports.py

	# A list of angular modules to be added as dependencies to horizon app.
	ADD_ANGULAR_MODULES = ['billingApp']
	
	# A list of javascript files to be included for all pages
	ADD_JS_FILES = ['billing_ui/js/app.js']

## Tests

### Unit tests
Los tests unitarios se escriben en archivos con extensión `.spec.js`. Por ejemplo, si tenemos un archivo llamado `servicio.js`, los tests del código de ese archivo deben estar en el mismo directorio, en `servicio.spec.js`.

Los archivos `.spec.js` se pueden manejar automáticamente si tenemos
	AUTO_DISCOVER_STATIC_FILES = True
en el archivo *enabled*.

### Correr los tests
Para correr los tests se puede acceder al test runner desde el browser en `<raíz de horizon>/jasmine/` (por ejemplo: `http://127.0.0.1:8000/jasmine/`). **En nuestro horizon-kilo están faltando algunos archivos core de jasmine, así que los tests no funcionan**.

En versiones más nuevas de OpenStack los tests se pueden correr sin usar el browser (ver [Guía oficial]). Es necesario tener instalado nodejs y npm para hacer esto.

Links
=====
[Guía Oficial de Angular en OpenStack][Guía oficial] (Aplica a versiones recientes solamente)

[Guía oficial]:http://docs.openstack.org/developer/horizon/topics/angularjs.html 