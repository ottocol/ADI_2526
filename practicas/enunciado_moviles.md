# Práctica evaluable 4: Aplicaciones web en dispositivos móviles

El objetivo de esta práctica es convertir el *frontend* desarrollado en la práctica anterior a un formato apropiado para ejecutar en un dispositivo móvil. El "producto final" de la práctica debería tener apariencia de aplicación para móvil aunque en realidad estará desarrollado con HTML+CSS+JS.

Para que la app web tenga aspecto de aplicación móvil usaremos [Ionic](https://ionicframework.com/), que proporciona componentes de interfaz de usuario con aspecto similar al nativo en iOS o Android. Ionic se puede usar con apps Vue, Angular, React o JS "vanilla".


## Requisitos mínimos (hasta 7 puntos)

> Aunque gran parte del código de la práctica 3 os va a servir para esta, **el proyecto de esta práctica debería ser uno nuevo** y distinto de la práctica anterior, para evitar problemas. Lo que cambiará menos será el Javascript y lo que cambiará más será el HTML de los componentes, que ahora usará etiquetas de Ionic.

Cread un nuevo proyecto de Ionic para Vue [como se describe](https://ionicframework.com/docs/vue/quickstart) en la documentación. Hay tres plantillas a elegir, para los requisitos mínimos la mejor es la más sencilla, la `blank`, pero podéis también probar las otras dos para ver el aspecto que tienen. 


> Ionic usa *typescript* como lenguaje por defecto. Typescript es Javascript con tipos añadidos. Es decir, podemos especificar los tipos de las variables, funciones, etc. Los tipos son opcionales, de modo que en realidad un archivo .js es como un archivo .ts en el que no hubiéramos especificado ningún tipo. Si Ionic o VSCode te dan problemas con algún .js que importes de prácticas anteriores, prueba a renombrarlos a .ts. Y si examinas algún .ts generado por la plantilla de Ionic (por ejemplo el `index.ts` del router) verás que son Javascript con algunos tipos especificados en puntos concretos. 

**Como requisitos mínimos se pide que en la aplicación se pueda hacer login, logout y se pueda hacer CRUD de un recurso cualquiera** (el que proceda en vuestra aplicación, películas, coches, productos, ...). Eso sí, las "pantallas" de la app deberían estar organizadas adecuadamente para un dispositivo móvil. En Ionic una "pantalla" es una `<ion-page>`.

- Una pantalla inicial solo con el formulario de login
- Una pantalla con una lista (en Ionic, [`<ion-list>`](https://ionicframework.com/docs/api/list) ) en la que en cada fila aparezca un elemento, con botones para editar/borrar, y haya alguna forma de añadir un elemento. Además desde esta pantalla se debería poder hacer logout, lo que redirigiría a la pantalla de login.

Cómo implementéis visualmente las funcionalidades de editar, borrar y añadir queda a vuestra elección. Echad un vistazo a la [documentación sobre los componentes visuales](https://ionicframework.com/docs/components) de Ionic para averiguar cuáles os podrían ser útiles.




## Requisitos adicionales (hasta 3 puntos)
 
- **CRUD de más de un recurso (1 punto)** otra pantalla adicional donde se pueda hacer CRUD de otro recurso, para navegar entre ambas necesitaréis *tabs* o un *sidebar menu*.
- **Implementar notificaciones en tiempo real (1 punto)** Una funcionalidad interesante de pocketbase es el [realtime API](https://pocketbase.io/docs/api-realtime/) que permite suscribirse en tiempo real a cambios en las colecciones. Añadid esta funcionalidad a vuestra app de manera que cuando se modifiquen datos en el backend se reflejen automáticamente en el frontend. No es necesario que la modificación de datos la hagáis mediante una interfaz vuestra, podéis modificar los datos con el panel de control de pocketbase. Grabad un video en el que se vea cómo al modificar los datos se refleja en la interfaz.
- **Desplegar la app como una pwa: (2 puntos, 1 por frontend y 1 por backend)** 
    - En la documentación de Ionic para vue se describe [cómo convertir la app web en una PWA](https://ionicframework.com/docs/vue/pwa) (añadiendo automáticamente un manifest y un service worker) y cómo desplegarla. En dicha documentación se usa como *host* para el despliegue Firebase Hosting pero probablemente os resulte más fácil daros de alta y desplegar en Vercel o Netlify (mirad sus webs para ver cómo desplegar una app *frontend*).
    - Para el backend podéis crear una máquina virtual gratuita en Azure a través del [portal de Azure en la UA](https://eps.ua.es/es/eservices/microsoft-azure.html). Una vez dados de alta, si en el portal de Azure entráis en los servicios gratuitos tenéis máquinas virtuales gratuitas windows o linux donde se puede instalar Pocketbase y así tener frontend y backend en la nube. Aquí tenéis [el proceso que seguí yo para instalar pocketbase en una MV Azure](https://chill-dumpling-bba.notion.site/desplegar-pocketbase-en-Azure-Virtual-Machines-2b583983c89280ee9000dac0f332ca23). En lugar de Azure se puede usar por ejemplo Digital Ocean si reclamáis el [Github Student Developer Pack](https://education.github.com/pack).
- Cualquier otra funcionalidad relacionada con el funcionamiento de la aplicación en dispositivos móviles, por ejemplo probar a implementar alguna funcionalidad en otro framework móvil como quasar para Vue. Consultad antes con el profesor para ver cómo se podría valorar.


## Entrega

El plazo de entrega concluye el **viernes 19 de diciembre de 2025 a las 23:59**. La entrega se realizará como es habitual a través de Moodle.

Entregad también un LEEME.txt donde expliquéis brevemente las partes optativas que habéis hecho y cualquier detalle que consideréis necesario.


