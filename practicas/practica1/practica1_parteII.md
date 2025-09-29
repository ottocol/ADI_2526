# Práctica 1, parte II: Desarrollo del *backend* de la aplicación web

> Recordad que el prototipo desarrollado en la parte 1 de la práctica se valoraba con hasta 3 puntos, a los que hay que sumar en esta parte los 5 de los requerimientos básicos y los 2 de los adicionales.

## Requerimientos básicos (5 puntos)

En la parte II de la práctica 1 desarrollaremos el *backend* de la aplicación web que prototipásteis en la parte I. La implementación la haremos con el BaaS PocketBase. 

### Casos de uso a implementar

Recordad que en la parte I os hablábamos de una serie de casos de uso, que ahora debéis implementar:

- Autentificación de usuarios
- Crear un nuevo elemento (recetas, fotos, películas…), pasando como parámetro un objeto JS con los datos
- Buscar elementos (por texto, por género, por categoría… dependerá del tipo de elemento).
- Listar todos los elementos de un tipo dado
- Editar un elemento, pasando como parámetro su id y un objeto JS con los nuevos datos
- Eliminar un elemento, sabiendo su id

A esto debéis añadir el **CRUD de usuarios**.

Aunque en PocketBase podríamos implementar todo esto con una API REST ya lo habéis hecho en otras asignaturas. Usaremos el API JS de PocketBase, que es estilo RPC (procedural u orientado a funciones). Es decir, implementaréis una capa de servicios con funciones como `buscarPelicula(titulo)` o `eliminarPelicula(id)`.


### Implementación como una capa de servicios

El código cliente que en el futuro haga uso de los servicios del *backend* no debería necesitar conocer que este está implementado en PocketBase, es decir debéis crear funciones o métodos que actúen como una "capa de servicios" aislando del API de PocketBase.

Por ejemplo podéis crear una función `login(email, password)` que acabe llamando al `authWithPassword` de PocketBase, o en el caso del *crowdfunding* una función o método `listarProyectosMasPopulares(num)` que devuelva los datos de los `num` proyectos más populares llamando internamente al API de BD de PocketBase. Y así con todos los "servicios" proporcionados por el *backend*.

### El *singleton* `PocketBase`

El objeto creado con `PocketBase()` (en los ejemplos de la documentación se suele llamar `pb`) es el que contiene la mayoría de los métodos del API. Además contiene el estado actual de autenticación (o sea, si hay un usuario logueado). Si cada vez que llamamos a una función de la capa de servicios creamos un nuevo objeto `PocketBase()` perderemos la autenticación. Por eso es mejor instanciar este objeto como un *singleton* y compartirlo entre todas las funciones de la capa de servicios:

```javascript
//Archivo pb.js
import PocketBase from "pocketbase";

export const pb = new PocketBase("http://127.0.0.1:8090");
```

```javascript
//Otro archivo, perteneciente a la capa de servicios
import {pb} from "./pb.js"

export async function login(email, password) {
  return pb.collection("users").authWithPassword(email, password);
}

```

> Si ejecutáramos el código desde el navegador no sería estrictamente necesario compartir la instancia "pb" ya que la información de autenticación la guarda automáticamente PocketBase en el propio navegador, en un almacenamiento llamado `localStorage`. No obstante, no es una mala práctica ya que no tener que crear una nueva instancia de `PocketBase()` en cada llamada a nuestra capa de servicios va a ser más eficiente.

## Requerimientos adicionales (hasta 2 puntos)

> Tened en cuenta que no os puedo poner más de un 10 en la práctica. No hagáis partes adicionales "de más".

- **Pruebas unitarias (hasta 1 punto):** si implementáis una suite de pruebas unitarias para probar las funcionalidades implementadas. Hay muchas herramientas para pruebas unitarias en Javascript, por ejemplo Mocha o Jest, tendréis que buscar cómo configurarlas y usarlas.
- **CRUD sobre más de un recurso (hasta 1 punto):** en los requisitos mínimos se pide  implementar CRUD sobre al menos un recurso. Para esta parte adicional se pide implementar CRUD sobre otro recurso adicional.
- **Acceso a un API REST externo (hasta 0.5 puntos):** que algún caso de uso implique acceder a un API REST externo. Por ejemplo si vuestra app fuera de reserva de hoteles os podría interesar mostrar el tiempo que va a hacer en el destino en la fecha elegida. 
- **paginación de resultados (hasta 0.5 puntos):** implementar una búsqueda o listado con paginación, tendréis que mirar en la documentación de PocketBase.

Podéis realizar cualquier otra ampliación o mejora que se os ocurra, pero consultadlo antes con el profesor para asegurarse de que sería válida para la evaluación y tener una idea de hasta cuánto se podría valorar en la nota.

## Evaluación de la práctica y normas de entrega

La fecha límite para la entrega será  **el lunes 20 de octubre a las 23:59**. La entrega se realizará comprimiendo todos los archivos de vuestro proyecto en un .zip y subiéndolos a moodle (¡¡acordáos de no comprimir el `node_modules`!!).

Además del código tendréis que entregar la siguiente documentación:

- Siempre que uséis un LLM deberíais incluir un log con los prompts usados (para abreviar no es necesario que incluyáis las respuestas completas del LLM)
- Un video entre 2 y 4 minutos de duración tipo screencast en el que los dos componentes del grupo expliquéis al 50% la estructura del código desarrollado y los requerimientos adicionales, si los hay.

