# Introducción básica a Node.js para aplicaciones web

Node.js (o simplemente Node) es una plataforma de desarrollo para implementar aplicaciones web en el lado del servidor escritas en Javascript. En principio surgió como una plataforma especialmente apropiada para aplicaciones asíncronas y en tiempo real. No obstante, dada la cada vez mayor popularidad de Javascript se está usando mucho para aplicaciones web más convencionales. 

> En la asignatura vamos a usar Node como plataforma de servidor no por sus virtudes con respecto a otras plataformas sino básicamente por permitirnos *usar el mismo lenguaje de programación en el cliente y en el servidor: Javascript*.

## Instalar Node


En la página de Node hay [instaladores](https://nodejs.org/en/download/) para Linux, Windows y Mac. Un problema es que a veces necesitan permiso de superusuario para instalarse. Como alternativa, se puede usar un gestor de versiones de Node, que nos permite tener varias versiones coexistiendo en nuestra máquina y generalmente no necesitan permisos de superusuario para instalarlas.

### Gestores de versiones Node en Linux/Mac

`n` es un gestor de versiones de Node para Linux/OSX potente y fácil de usar e instalar.
Para instalar la última versión LTS de Node con `n`, simplemente tecleamos en una terminal:

```bash
curl -L https://git.io/n-install | bash
```
La instalación se hace en el directorio `$HOME/n`, para no requerir permisos de superusuario. A partir de aquí se recomienda leer la [documentación](https://github.com/tj/n) de `n` para saber cómo instalar versiones adicionales de Node o seleccionar la versión activa para cada proyecto.


### Gestores de versiones Node en Windows

La propia Microsoft recomienda un gestor de versiones de Node denominada [nvm](https://nodejs.org/en/download/) (hay otro gestor de versiones de node llamado [nvm](https://github.com/nvm-sh/nvm) para Linux/OSX, aunque es otro proyecto distinto).

> Nosotros no vamos a usar Express para implementar el servidor, por lo que en realidad el código anterior no es relevante para nosotros. Nuestro *backend* usa pocketbase, que es el que hará de servidor. Usaremos Node de momento para ejecutar el código que "ataque" a ese backend para obtener resultados, es decir, la capa de servicios de la aplicación con sus casos de uso.


Node es un intérprete en línea de comandos, por lo que si tecleamos `node` entraremos en el REPL. Aquí podemos escribir instrucciones de Javascript y ver el resultado inmediato. Podemos salir pulsando Ctrl-C dos veces. También podemos escribir el código en un fichero con extensión `.js` y ejecutarlo con `node mi_fichero.js`.

## Gestión de paquetes. Creación de un proyecto

`Node` incluye un sistema de *paquetes* con gestión automática de dependencias al estilo de los usados en las distribuciones de linux (como los `.deb` o `.rpm`).

La herramienta de gestión de paquetes en Node se llama `npm`  y es similar al `apt-get` de Debian, o al Maven del mundo Java).

Un proyecto Node no es más que un directorio que debe contener un archivo especial llamado `package.json`** con información sobre el proyecto: nombre, versión, autor, dependencias, ... Gracias a este archivo podemos automatizar ciertas tareas como por ejemplo la ejecución del proyecto o la instalación en un solo paso de todas las dependencias.

Podemos crear el `package.json` con el comando `npm init` que nos irá solicitando los datos del proyecto de manera interactiva.

Para crear un proyecto de prueba:

```bash
mkdir hola_node
cd hola_node
npm init
```

## Instalación de dependencias en un proyecto

Es muy típico que un proyecto tenga dependencias de terceros. En el `package.json` debemos listar estas dependencias para que se pueda automatizar el proceso de instalación del proyecto en una nueva máquina.

**En Node es habitual que las dependencias se instalen de modo local, en el propio directorio del proyecto**. Por ejemplo, vamos a instalar el paquete `colors`, para colorear la salida por la consola.

```bash
# estando dentro del directorio del proyecto
$ npm i colors  #i de install
```

El comando anterior **crea un subdirectorio `node_modules` en el directorio actual**, conteniendo el código del paquete `colors` (y los paquetes de los que depende, si los hubiera).

Nótese que si distintos proyectos en los que estamos trabajando comparten dependencias, estas estarán repetidas en cada proyecto, ya que no tenemos un repositorio centralizado local (como sí pasa por ejemplo en Maven). De este modo evitamos problemas de versionado de dependencias, ya que cada proyecto usa directamente la versión que necesita, pero a costa de duplicar la información.


> La versión o versiones de las dependencias que son válidas para el proyecto se especifican mediante *semver* o *semantical versioning*, que es una especie de estándar. Este estándar incluye unas normas sobre cómo numerar las versiones de los proyectos (`MAJOR.MINOR.PATCH`) y cuándo cambiar cada número de versión y además una sintaxis para especificar qué conjunto de versiones son compatibles con nuestro proyecto. Por ejemplo el `^` indica que nos vale cualquier versión igual o superior a la especificada mientras no cambie el número `MAJOR`. Podéis ver más información sobre el tema en la [documentación de semver](https://docs.npmjs.com/cli/v6/using-npm/semver) de npm y en esta [calculadora de *semver*](https://semver.npmjs.com/) en la que al meter una expresión de rango de versiones para un paquete npm, nos dará una lista de las incluídas en la misma. Por cierto,  de momento probablemente no queráis en vuestro proyecto [una versión de colors superior a la 1.4.0](https://fossa.com/blog/npm-packages-colors-faker-corrupted/) `:/`.

Cuando queramos instalar paquetes que se usen solo en el desarrollo (por ejemplo para *testing*) podemos instalarlos con `npm i <paquete> --save-dev`. De este modo la referencia se añade al apartado `devDependencies` del `package.json`. Con un proyecto que nos hayamos bajado de otro sitio, si instalamos las dependencias con `npm i --production` no se instalarán las `devDependencies`, solo las de "producción". 

Gracias a las `dependencies` del `package.json`, cuando subamos un proyecto Node propio a un  repositorio no es necesario adjuntar las librerías, solo el código propio. Cuando nos bajamos un proyecto Node de un tercero (por ejemplo de Github)  podemos **instalar todas sus dependencias** simplemente con

```bash
$ npm i 
```




> Como hemos visto, las dependencias del proyecto se suelen instalar en modo *local* (o sea, en el mismo directorio del proyecto). Los paquetes que incluyen herramientas en línea de comandos se suelen instalar en modo *global* (o sea, en un directorio global compartido por todos los proyectos).

![](https://imgs.xkcd.com/comics/dependency.png)

## Uso de las dependencias del proyecto en nuestro código

De manera similar a como se usa el include en C o el import en Java, podemos incluir las librerías que nos hemos bajado en nuestro propio código. De hecho en Javascript hay dos formas de hacerlo por razones históricas:

- Con la instrucción `require` incluímos lo que se denominan módulos CommonJS
- Con la instrucción `import` incluímos módulos ESM (ECMAScript Modules)

El estándar actual y recomendado es ESM (sobre todo en navegadores), mientras que CommonJS es el que surgió originalmente con Node. Por desgracia ambos formatos son incompatibles, pero la mayoría de paquetes que se distribuyen actualmente con `npm` ofrecen compatibilidad con ambos. Algunos nuevos se publican ya solo en ESM y otros muy antiguos solo están disponibles en CommonJS.

Vamos a usar ESM desde ya porque así nos servirá también para cuando lleguemos al lado del cliente. No obstante, como no es el formato nativo de Node nos tocará modificar el `package.json` para que lo use. **Edita el `package.json` y añade la siguiente propiedad:**

```json
"type": "module"
```

Ten en cuenta que las propiedades van separadas por comas, así que asegúrate de añadir la necesaria antes de donde hayas colocado la nueva propiedad.

A partir de este momento ya podemos hacer uso del paquete importándolo con **import**

```javascript
import colors from 'colors'
//Evidentemente cada paquete tiene su API así que habrá que consultar la documentación para saber cómo usarlo
//pone las letras en color verde
console.log(colors.green('Saludos de Marte'))
//Pone cada letra de un color
console.log(colors.rainbow('Todos amamos node.js'))
```

Habitualmente **cada paquete "exporta" un objeto a través del cual accedemos a su API**. Ese objeto es obtenido por el `import`. En el caso de `colors` si leemos la documentación veremos que hay varios métodos para cambiar el color: `green`, `red`, `rainbow`, etc.

> De momento esto nos basta para poder usar `import` de manera básica, pero como veremos, existe la posibilidad de importar solo los métodos que necesitemos de un paquete y por supuesto de definir nuestros propios paquetes para modularizar nuestra aplicación.