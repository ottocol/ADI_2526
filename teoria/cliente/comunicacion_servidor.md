<!-- .slide: class="titulo" -->

## Tema 4: Javascript en clientes web
## Parte II: Comunicación con el servidor y persistencia local



---

Necesitamos alguna funcionalidad de JS en el navegador para poder hacer peticiones al servidor (Un API típicamente REST, salvo casos como Pocketbase u otros APIs RPC)

<img src="images_logica/modern_webapp.png" class="r-stretch">

- El servidor actúa como una especie de "repositorio remoto de datos"
- Además nos vendría bien tener un repositorio local

---

<!-- .slide: class="titulo" -->

## 4.4. AJAX


---

## AJAX

*Asynchronous Javascript And XML*

Es una combinación de tecnologías:

- **API `fetch`** para **hacer peticiones HTTP al servidor** con Javascript y recibir la respuesta sin cambiar de página
- ***XML***: al comienzo era el formato de intercambio de datos cliente/servidor. Hoy en día se usa JSON.
- **API DOM**: ya visto, permite actualizar **solo parte de la página** con los datos del servivor

---


## Formato de datos en AJAX: casi siempre JSON

Originalmente se usaba XML para intercambiar datos, pero es tedioso de *parsear* con JS (hay que usar el API DOM), mientras que convertir una cadena de texto JSON a objeto JS es trivial con el API estándar

```javascript
//Si esto fuera AJAX de verdad, este texto vendría en la respuesta del servidor
var texto = '{"login":"pepe", "nombre": "Pepe Pérez"}'
//De cadena JSON a objeto JS
var objeto = JSON.parse(texto)
console.log(objeto.login) //pepe
//De objeto JS a cadena JSON
console.log(JSON.stringify(objeto))
```

---

## Una nota sobre seguridad en el lado del servidor

(Si escribiéramos nuestro propio servidor, cosa que no hacemos con Pocketbase) No nos podemos fiar de lo que llega desde el cliente, usar validadores de esquema JSON, que comprueban que el objeto cumpla un formato, como [ajv](https://ajv.js.org) o [zod](https://zod.dev) 

```javascript
import * as z from 'https://cdn.jsdelivr.net/npm/zod@4.1.12/+esm';

const UsuarioSchema = z.object({
  login: z.string(),
  nombre: z.string(),
});

let texto = '{"login":"pepe@ua.es", "nombre":"Pepe Pérez", "edad":22}';

const datos = JSON.parse(texto); //puede tener campos adicionales y/o de tipos distintos
const usuario = UsuarioSchema.parse(datos); // lanza excepción si no encaja, filtra campos adicionales
```

[https://playcode.io/2577994](https://playcode.io/2577994)

---

## Hacer peticiones HTTP con el `fetch` API

Por defecto se hace una petición GET


```html
<h1 id="header">Avatar de Github</h1>
<input type="search" id="buscado"><button id="buscar">Buscar usuario de Github</button>
<img src="" id="imagen">
```

```javascript
document.getElementById("buscar").addEventListener("click", async function(){
  let resp = await fetch('https://api.github.com/users/' + document.getElementById("buscado").value)
  let datos = await resp.json()
  console.log(datos)
  document.getElementById("imagen").src = datos.avatar_url
})
```

[https://playcode.io/2577537](https://playcode.io/2577537)

---


## Peticiones más complejas con `fetch`

Para cambiar el tipo de petición, añadir cabeceras, cuerpo de petición, etc, podemos pasar un segundo parámetro que es un objeto JS con las propiedades:

[https://jsbin.com/razelec/edit?html,js,console](https://jsbin.com/razelec/edit?html,js,console)<!-- .element: class="caption" -->

```javascript
//reqres.in es un API REST "fake" al que podemos hacer peticiones
var usuario = {};
usuario.login = "Pepe"
usuario.nombre = "Pepe Pérez"
var respuesta = await fetch('https://reqres.in/api/users', {
    method: 'POST',
    headers: {
      'Content-type':'application/json'
    },
    body: JSON.stringify(usuario)
})
var datos = await respuesta.json();
console.log(datos.login)
```

---

## Formularios y AJAX

En **aplicaciones "tradicionales"** el navegador es el responsable de enviar los datos tecleados en los campos del formulario, y este proceso se desencadena automáticamente con un `input` de `type=submit`.

En **aplicaciones con AJAX** y formularios el código JS es el que debe recolectar los datos contenidos en los campos y enviarlos con `fetch`. Este proceso se puede desencadenar con un `<button>` convencional. Si usamos un  `type=submit` necesitaremos `preventDefault()` para que el navegador no desencadene también el envío, *en caso contrario la página actual se perdería*.


---


```html
<form id="form_login">
  <input type="text" id="username"/>
  <input type="text" id="email"/>
  <input type="submit" value="Dar de alta">
</form>
```


```javascript
document.getElementById('form_login').addEventListener('submit', async function (e) {
    e.preventDefault();
    const datos = {
      username: document.getElementById('username').value,
      email: document.getElementById('email').value,
    };
    let resp = await fetch('https://jsonplaceholder.typicode.com/users', {
      method: 'POST',
      headers: {
        'Content-type': 'application/json',
      },
      body: JSON.stringify(datos),
    });
    let datosResp = await resp.json();
    console.log(datosResp);
});
```

[https://playcode.io/2578013](https://playcode.io/2578013)

---

## ¿Y qué pasa con el API de pocketbase?

El API JS de Pocketbase es de tipo RPC y hace transparentes las llamadas remotas, por eso no vemos el `fetch` (pero estar, está)

Podéis verlo en las "Herramientas para desarrolladores" de vuestro navegador, típicamente dentro de la solapa "Red" pueden verse las peticiones HTTP.  Debe haber una opción llamada "XHR" para ver las hechas con AJAX


---


<!-- .slide: class="titulo" -->

## 4.5 CORS


---

## Restricciones de seguridad

**Política de seguridad del “mismo origen”**: un `fetch` solo puede hacer una petición al mismo *host* del que vino la página en la que está definido

Por ejemplo, el Javascript de una página de `www.vuestrositio.com` en principio no puede hacer peticiones AJAX a Facebook (salvo que FB lo permita **explícitamente**)

![](http://www.lucadentella.it/blog/wp-content/uploads/2013/07/cross-blocked.jpg)


---

## CORS

*(Cross Origin Resource Sharing)* : permite saltarse la *same origin policy* con la colaboración del servidor

- En cada petición *cross-domain* el navegador envía automáticamente una cabecera `Origin` con el origen de la petición. Es imposible falsearla desde JS 
- El servidor puede enviar una cabecera `Access-Control-Allow-Origin` indicando los orígenes desde los que se puede acceder a la respuesta. Si encajan con el origen de la petición el navegador dará “luz verde” 

```http
HTTP/1.1 200 OK  
Server: Apache/2.0.61   
Access-Control-Allow-Origin: * 
```

⚠️ CORS es para proteger al navegador y a su usuario, no al servidor

---


## ¿Necesitamos CORS para usar un API propio?

No, pero nos da la libertad de que el servidor del API y el del *frontend* sean distintos

![](images_logica/servidor_front_y_servidor_back.png) <!-- .element: class="stretch" -->


---

## Implementar CORS en el servidor

Aunque "simplemente" se trata de generar las cabeceras adecuadas desde el servidor, en prácticamente todos los lenguajes de programación en el servidor tenemos alguna librería/paquete/plugin... que añade CORS a nuestra aplicación de modo sencillo.

Por ejemplo en Express tenemos el paquete `cors`:

```javascript
var cors = require('cors')
//Suponiendo "app" la variable obtenida como app=express()
app.use(cors())
//cors es un middleware, un software entre la petición y la respuesta
```

---


Aclaración: CORS **no es un mecanismo de protección del servidor**. Nada nos impide escribir código en Java/Python/Ruby...etc, incluso NodeJS, que pueda hacer la petición HTTP desde fuera del navegador. Se trata de una **limitación de JS en el navegador**

---

<!-- .slide: class="titulo" -->

## 4.6
## Persistencia local


---

## ¿Por qué almacenar datos en el cliente?

- **Variables compartidas** entre diferentes páginas de la aplicación: recordemos que el ámbito de una variable JS es el HTML en el que está el JS que la define
- **Datos permanentes** que queremos conservar entre diferentes "sesiones" de navegación. Por ejemplo guardar un *token* JWT para no tener que autentificarnos siempre.
- **Datos que no podemos sincronizar** con el servidor por ejemplo por falta de conectividad en este momento (por ejemplo, porque estamos en el metro)

---

## API Local Storage

- Objeto global `localStorage` donde podemos almacenar **pares "clave/valor"** que no se pierden aunque se cierre el navegador
- Podemos almacenar un dato bajo una clave, recuperarlo conociendo la clave o iterar por todas las claves y valores

```javascript
localStorage.login = "pepe"
```
- El valor es siempre una **cadena**, para otro tipo de datos hay que hacer la conversión manualmente
    + Aunque ya hemos visto que la conversión objeto<->cadena es sencilla gracias a `JSON.stringify` y `JSON.parse`
- El **ámbito** es el sitio web actual. Hay otro objeto `sessionStorage` que reduce más este ámbito, a la "pestaña" actual del navegador, y además se pierde si se cierra.

---

## Ejemplo de `localStorage`

```javascript
//http://jsbin.com/bofabe/edit?html,js,output
function guardarNombre() {
  nombre = prompt("¿cómo te llamas?")
  localStorage.setItem("usuario", nombre)
  //esta sintaxis es equivalente a lo anterior
  localStorage.usuario = nombre
  //y esta también
  localStorage["usuario"] = nombre
  edad = prompt("¿Cuántos años tienes?")
  localStorage.edad = edad
}
function mostrarNombre() {
  alert("Me acuerdo de ti, " + localStorage.usuario +
  " vas a cumplir " + (parseInt(localStorage.edad) + 1) + " años!!")
}
function mostrarTodosLosDatos() {
  datos=""
  for(var i=0; i<localStorage.length; i++) {
     clave = localStorage.key(i)
     datos = datos + clave + "=" + localStorage[clave] + '\n'   
  }
  alert("LocalStorage contiene " + datos)
}
```

---

## Bases de datos en el cliente

- El API nativo es IndexedDB: una base de datos de pares clave-valor (tipo NoSQL). 
- Probablemente es de demasiado bajo nivel como para usarlo directamente

<div class="row">
  <div class="column half">
    <img src="images_logica/dexie.png">
    <a href="https://dexie.org/">Dexie</a>
  </div>
  <div class="column half">
    <img src="images_logica/pouchDB.png">
    <a href="https://pouchdb.com/">PouchDB</a>
  </div>
</div>  

Ya veremos esto en los temas de dispositivos móviles
