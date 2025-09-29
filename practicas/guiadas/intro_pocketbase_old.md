# Introducción a Pocketbase


Pocketbase es un **Backend As A Service** ligero pero razonablemente potente y que podemos instalar en nuestra propia máquina de manera sencilla, ya que es simplemente un ejecutable de unos 12 Mb. Tiene una serie de funcionalidades que nos permitirá implementar el *backend* de manera mucho más rápida que si lo tuviéramos que hacer desde cero:

- **Gestión de usuarios y autenticación**: CRUD de usuarios de la aplicación, login y logout. La autenticación se puede hacer con usuario/contraseña o con servicios de terceros (Google. Github, ...) a través de OAuth.
- **Base de datos realtime**: tiene embebida una instancia de SQLite y proporciona un API de mayor nivel de abstacción que SQL para poder trabajar con los datos. Además podemos observar los cambios en los datos en tiempo real.
- **Storage**: si nuestra aplicación necesita almacenar archivos o que los puedan subir los usuarios (fotos, audios, documentos, ...) podemos subirlos al servidor a través de un API sencillo proporcionado por Pocketbase, o podemos conectar nuestra aplicación con un servicio de almacenamiento externo como S3.

## Descargar y ejecutar por primera vez

Ve a la documentación de PocketBase y baja el ZIP para tu plataforma (Linux/Mac/Win). Tras descomprimirlo obtendrás el ejecutable `pocketbase` (`pocketbase.exe` en Windows).

Para arrancar el servidor, ir a la carpeta donde hayas descomprimido el ejecutable y teclear:

```bash
./pocketbase serve #Linux/Mac
pocketbase serve #Windows
```

Creará automáticamente una carpeta `pb_data` con la base de datos y arrancará un navegador en el que pedirá los datos para crear un usuario administrador. Una vez creado este abrirá la interfaz web 

## La interfaz web de administración

Desde esta interfaz (que por defecto está en `http://localhost:8090/_/`) podemos añadir usuarios a nuestra aplicación y asignarles mecanismos de autenticación, crear las colecciones (==tablas) de la base de datos, etc.

> Todo lo que se puede hacer desde la interfaz también se puede hacer desde la API, por ejemplo cuando la app esté en producción también se pueden dar de alta los nuevos usuarios a través de la API. No obstante hay tareas que son más cómodas en la interfaz web, como crear los usuarios de prueba, crear la estructura inicial de la base de datos, etc.

Vamos a crear una aplicación para gestionar la lista de la compra, en la que cada usuario tendrá almacenada la lista de cosas que quiere comprar. Cada item a comprar tendrá simplemente un *nombre* y un campo booleano *comprado* que indicará si la lo hemos comprado o no. De momento para simplificar cada usuario solo puede tener una lista.

### Usuarios

Vamos a añadir algún usuario de prueba:

1. Pulsa sobre el icono de base de datos (*Collections*) en la barra vertical de iconos de la izquierda de la pantalla
2. Selecciona la colección  *users*.
3. Pulsa el botón un botón `New Record` en la parte superior derecha de la pantalla 
4. Añade un usuario con el email y contraseña que quieras (mientras luego lo recuerdes...) 

El resto de campos son bastante autoexplicativos salvo quizás `verified` que indica si el usuario tras darse de alta ha verificado su dirección de email haciendo clic en el típico enlace de "verifica tu cuenta". Si queréis ver cómo funciona, lo podéis mirar en la documentación, de momento podéis dejarlo en off (*false*).

### Colecciones

Vamos a añadir una colección llamada `lista`, donde se almacenarán los items de la lista de la compra de todos los usuarios, cada registro estará relacionado con el usuario al que pertenece.

1. Pulsa sobre el icono de base de datos (*Collections*) en la barra vertical de iconos de la izquierda de la pantalla
2. Pulsa sobre el botón  `New Collection`
3. Dale como nombre `lista`. 
4. Verás que por defecto cada registro tiene un id autogenerado, una fecha de creación y una fecha de última actualización. Con el botón `New Field` ve añadiendo los campos necesarios: 

    - `nombre` de tipo `plain text`
    - `comprado` de tipo `Bool`. 
    - `usuario` de tipo `Relation` (selecciona la colección `users` y el tipo `single`)

5. Pulsa sobre `Create` (esquina inferior derecha) para acabar de crear la colección
6. Puedes añadir un dato de ejemplo con el botón `New Record`. Verás que para seleccionar el usuario tienes un *picker* que te permite elegir cualquier usuario de la colección `Users`. Añade un par de registros con el usuario que creaste antes para luego poder probar a obtener su lista de la compra.

> Pocketbase usa como base de datos SQLite, que es relacional. Internamente se creará un campo en la tabla `lista` que será una clave ajena a la tabla `Users`.

### Reglas de acceso

Como veremos cuando lleguemos al desarrollo en el lado del cliente, los APIs de Pocketbase se pueden llamar desde el navegador. Pocketbase no tiene control sobre las peticiones que se hagan desde el navegador y no puede confiar en que el código que las hace no sea código manipulado o malicioso.

Hay que buscar por tanto algún mecanismo para proteger los datos de las colecciones ante accesos no autorizados. Muchos BaaS como Pocketbase emplean **reglas de acceso**, que son reglas que permiten o no las operaciones sobre las colecciones basándose típicamente en el usuario que está autentificado combinado con el que figura en algún campo de la colección. Por ejemplo en nuestro caso no deberíamos permitir el CRUD sobre los registros de la colección `lista` salvo que el usuario autentificado sea el mismo que figura en el campo `usuario` del registro. Otro ejemplo: si tuviéramos noticias en un periódico podríamos exigir que solo las pudieran ver los usuarios autentificados (sean quienes sean).

Cada colección tiene 5 reglas correspondientes a las acciones del API sobre colecciones: listar (varios registros), ver (uno solo, por su id), crear, actualizar y borrar. Si una regla está vacía quiere decir que esa operación solo la podrá hacer el superusuario.

Queremos que solo se puedan crear elementos de la lista de la compra si el usuario que hace la petición está autentificado. Como Pocketbase usa *tokens* para autentificación, eso equivale a decir que en la petición tiene que venir un *token* correspondiente al usuario. O dicho en la sintaxis de reglas de Pocketbase, que el id del usuario autentificado no debe estar vacío:

```javascript
@request.auth.id != ""
```

En pocketbase existen una serie de palabras clave para especificar condiciones a cumplir por los campos de los registros, o detalles de la petición como el usuario autentificado o las cabeceras presentes. Tenéis la [sintaxis completa](https://pocketbase.io/docs/api-rules-and-filters/) en la documentación.

Queremos que los elementos de la lista de la compra solo se puedan listar, ver y borrar si el usuario autentificado es el puesto en el campo "usuario" (luego veremos qué pasa con actualizar)

```javascript
@request.auth.id = usuario
```

Actualizar es un caso especial ya que no solo queremos que el usuario autentificado sea el del campo "usuario", sino también que no pueda cambiar este último campo.

```javascript
usuario = @request.auth.id && (@request.body.usuario = "" || @request.body.usuario = @request.auth.id)
```

> Nótese que las reglas de acceso en Pocketbase también funcionan como filtros, si luego en el API intentamos listar todos los registros de la colección `lista`, como en la regla de listar tenemos que solo se permiten los asociados al usuario autentificado solo saldrán automáticamente estos.


## El API Javascript

Pocketbase ofrece un API con diferentes implementaciones: REST, Javascript y Go. Desde Javascript, que es el lenguaje que usamos en la asignatura, podríamos acceder tanto al API REST como al nativo Javascript. Usaremos este último, de estilo RPC, ya que es el más directo desde JS.

De momento no vamos a usar un navegador para implementar y probar la parte de código relacionada con Pocketbase, usaremos Node.

Para crear nuestro proyecto con Pocketbase:

1. Crear un proyecto Node
    - Crear un directorio `demo-pb`
    - Entrar en él y ejecutar `npm init`. Esto creará el `package.json`.
2. Editar el `package.json` y añadirle la propiedad `"type":"module"` (recordad que es para poder usar la sentencia `import`).
2. Instalar el SDK JS de Pocketbase: `npm i pocketbase`

Aquí vamos a ver unos pocos ejemplos de llamadas al API, para más detalles podéis consultar la [documentación](https://pocketbase.io/docs/api-records/) del mismo. 

Por ejemplo, veamos cómo podríamos autentificarnos como un usuario, obtener nuestra lista de la compra y hacer logout. Escribe el siguiente código en un fichero `listar.js` dentro del proyecto `demo-pb`.

```javascript
import PocketBase from 'pocketbase';

const pb = new PocketBase('http://127.0.0.1:8090');

try {
    //login con email y password
    //CAMBIA EL EMAIL Y EL PASSWORD por los del usuario que creaste para hacer pruebas
    const authData = await pb.collection("users").authWithPassword("pepe@ua.es", "pepepepe");

    //no hace falta seleccionar solo nuestros registros, ya que la regla de acceso funciona como filtro
    const lista = await pb.collection("lista").getFullList()
    for(let i=0; i<lista.length; i++) {
        console.log(lista[i].nombre, lista[i].comprado ? "comprado" : "pendiente")
    }

    //Esto es como hacer logout. En realidad lo que hacemos es borrar las credenciales del cliente
    //con lo que ya no se van a enviar al servidor con cada petición
    pb.authStore.clear()
}
catch(error) {
    console.log(error.message)
}
```

Para ejecutarlo, teclear en una terminal 

```javascript
node listar.js
```

El caso de uso de "añadir un item a mi lista de la compra" se podría implementar como sigue, puedes escribirlo en un archivo `crear.js` y ejecutarlo con `node crear.js`: 

```javascript
import PocketBase from 'pocketbase';

const pb = new PocketBase('http://127.0.0.1:8090');

//CAMBIA el email y el password por los del usuario que creaste para hacer pruebas
try {
    const authData = await pb.collection("users").authWithPassword("pepe@ua.es", "pepepepe");

    const datos = {
        nombre: "zumo",
        comprado: false,
        //El id del usuario actual, dentro de authData tenemos más info, como el token de autenticación
        usuario: authData.record.id
    }

    const record = await pb.collection("lista").create(datos)
    console.log("Registro creado con id: ", record.id)

    pb.authStore.clear()
}
catch(error) {
    console.log(error)
}
```

> Como hemos visto, el objeto devuelto por `authWithPassword` devuelve información sobre el usuario actualmente autentificado. Tenemos la información de autenticación almacenada también en `pb.authStore`.

El código anterior tiene un **problema de seguridad**: un código malicioso podría asignar un id de usuario distinto al actual, con lo que estaríamos metiendo cosas en la lista de la compra de otro usuario. También podría no asignar ningún id (aunque esto último lo podríamos capturar con una regla de acceso). En Pocketbase tenemos los *hooks*, que son funciones que se ejecutan ante determinados eventos, por ejemplo, crear un registro en la base de datos. Podemos aprovechar este *hook* para asignarle desde el servidor al campo usuario el usuario autentificado actualmente, así nos aseguramos de que va a ser siempre el correcto:

```javascript


```

El resto de casos de uso pendientes (actualizar, borrar) no es excesivamente difícil de implementar consultando la documentación de Pocketbase. 

## Desplegar apps con pocketbase 

Al ser pocketbase un ejecutable linux sin más requerimientos puede instalarse en cualquier máquina virtual que se pueda desplegar en internet, es decir en cualquier VPS, por ejemplo Droplets de Digital Ocean, Virtual Machines de Azure, Lightsail de AWS, ...

| Programa                                  | Qué ofrece                                                                                                            | ¿Tarjeta? | Notas clave                                                                                                                                                    |
| ----------------------------------------- | --------------------------------------------------------------------------------------------------------------------- | --------: | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Azure for Students**                    | **100 USD de crédito** + servicios gratis 12 meses. **Sin tarjeta** para activar.                                     |    **No** | Permite crear VMs Linux/Windows y servicios gestionados dentro del crédito. Podéis entrar con vuestra cuenta de `mscloud.ua.es`. ([azure.microsoft.com][2])                                |

| **GitHub Student Developer Pack**         | Beneficios para estudiantes (p. ej. **$200 en DigitalOcean** para nuevos usuarios; GitHub Copilot, Codespaces, etc.). |     Varía | El pack no pide tarjeta; **los partners** pueden pedirla al canjear. DigitalOcean da $200/1 año a verificados. ([education.github.com][5])                     |

[1]: https://eps.ua.es/es/eservices/microsoft-azure.html?utm_source=chatgpt.com "Microsoft azure - Escuela Politecnica Superior, UA"
[2]: https://azure.microsoft.com/en-us/free/students?utm_source=chatgpt.com "Azure for Students – Free Account Credit"
[3]: https://aws.amazon.com/education/awseducate/?utm_source=chatgpt.com "AWS Educate - Cloud Skills for Education"
[4]: https://web.ua.es/en/dtic/noticias-dtic/iii-edition-of-the-free-training-program-in-aws-cloud-architecture.html?utm_source=chatgpt.com "III Edition of the Free Training Program in AWS Cloud ..."
[5]: https://education.github.com/pack?utm_source=chatgpt.com "GitHub Student Developer Pack"
