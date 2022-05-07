{{meta {load_files: ["code/crow-tech.js", "code/chapter/11_async.js"]}}}

# Programación Asincrónica

{{quote {author: "Laozi", title: "Tao Te Ching", chapter: true}

Quién puede esperar tranquilamente mientras el barro se asienta?\
Quién puede permanecer en calma hasta el momento de actuar?

quote}}

{{index "Laozi"}}

{{figure {url: "img/chapter_picture_11.jpg", alt: "Picture of two crows on a branch", chapter: framed}}}

La parte central de una computadora, la parte que lleva a cabo los
pasos individuales que componen nuestros programas, es llamada
_((procesador))_. Los programas que hemos visto hasta ahora son cosas que
mantienen al procesador ocupado hasta que hayan terminado su trabajo. La
velocidad a la que algo como un ciclo que manipule números pueda ser
ejecutado, depende casi completamente de la velocidad del procesador.

Pero muchos programas interactúan con cosas fuera del procesador. por
ejemplo, podrian comunicarse a través de una ((red)) de computadoras o
solicitar datos del ((disco duro))—lo que es mucho más lento que
obtenerlos desde la ((memoria)).

Cuando una cosa como tal este sucediendo, sería una pena dejar que
el procesador se mantenga inactivo—podría haber algún otro trabajo que
este pueda hacer en el mientras tanto. En parte, esto es manejado por tu
sistema operativo, que cambiará el procesador entre múltiples programas en
ejecución. Pero eso no ayuda cuando queremos que un _unico_ programa pueda
hacer progreso mientras este espera una solicitud de red.

## Asincronicidad

{{index "synchronous programming"}}

En un modelo de programación _sincrónico_, las cosas suceden una a la vez.
Cuando llamas a una función que realiza una acción de larga duración, solo
retorna cuando la acción ha terminado y puede retornar el resultado.
Esto detiene tu programa durante el tiempo que tome la acción.

{{index "asynchronous programming"}}

Un modelo _asincrónico_ permite que ocurran varias cosas al mismo tiempo.
Cuando comienzas una acción, tu programa continúa ejecutándose. Cuando
la acción termina, el programa es informado y tiene acceso al
resultado (por ejemplo, los datos leídos del disco).

Podemos comparar a la programación síncrona y asincrónica usando un pequeño
ejemplo: un programa que obtiene dos recursos de la ((red)) y
luego combina resultados.

{{index "synchronous programming"}}

En un entorno síncrono, donde la función de solicitud solo retorna
una vez que ha hecho su trabajo, la forma más fácil de realizar esta tarea es
realizar las solicitudes una después de la otra. Esto tiene el inconveniente de
que la segunda solicitud se iniciará solo cuando la primera haya finalizado.
El tiempo total de ejecución será como minimo la suma de los dos tiempos de
respuesta.

{{index parallelism}}

La solución a este problema, en un sistema síncrono, es comenzar
((hilo))s adicionales de control. Un _hilo_ es otro programa activo
cuya ejecución puede ser intercalada con otros programas por el
sistema operativo—ya que la mayoría de las computadoras modernas contienen
múltiples procesadores, múltiples hilos pueden incluso ejecutarse al mismo
tiempo, en diferentes procesadores. Un segundo hilo podría iniciar la segunda
solicitud, y luego ambos subprocesos esperan a que los resultados vuelvan,
después de lo cual se vuelven a resincronizar para combinar sus resultados.

{{index CPU, blocking, "asynchronous programming", timeline, "callback function"}}

En el siguiente diagrama, las líneas gruesas representan el tiempo que el
programa pasa corriendo normalmente, y las líneas finas representan el tiempo
pasado esperando la red. En el modelo síncrono, el tiempo empleado por
la red es _parte_ de la línea de tiempo para un hilo de control dado.
En el modelo asincrónico, comenzar una acción de red conceptualmente
causa una _división_ en la línea del tiempo. El programa que inició
la acción continúa ejecutándose, y la acción ocurre junto a el,
notificando al programa cuando está termina.

{{figure {url: "img/control-io.svg", alt: "Control flow for synchronous and asynchronous programming",width: "8cm"}}}

{{index "control flow", "asynchronous programming", verbosity}}

Otra forma de describir la diferencia es que esperar que las acciones
terminen es _implicito_ en el modelo síncrono, mientras que es _explicito_,
bajo nuestro control, en el asincrónico.

La asincronicidad corta en ambos sentidos. Hace que expresar programas que
hagan algo no se ajuste al modelo de control lineal más fácil, pero también
puede hacer que expresar programas que siguen una línea recta sea más
incómodo. Veremos algunas formas de abordar esta incomodidad más adelante en
el capítulo.

Ambas de las plataformas de programación JavaScript importantes—((navegadore))s
y ((Node.js))—realizan operaciones que pueden tomar un tiempo asincrónicamente,
en lugar de confiar en ((hilo))s. Dado que la programación con hilos es
notoriamente difícil (entender lo que hace un programa es mucho más
difícil cuando está haciendo varias cosas a la vez), esto es generalmente
considerado una buena cosa.

## Tecnología cuervo

La mayoría de las personas son conscientes del hecho de que los ((cuervo))s
son pájaros muy inteligentes. Pueden usar herramientas, planear con anticipación,
recordar cosas e incluso comunicarse estas cosas entre ellos.

Lo que la mayoría de la gente no sabe, es que son capaces de hacer muchas cosas
que mantienen bien escondidas de nosotros. Personas de buena
reputación (un tanto excéntricas) expertas en ((córvidos)), me han dicho
que la tecnología cuervo no esta muy por detrás de la tecnología humana,
y que nos estan alcanzando.

Por ejemplo, muchas culturas cuervo tienen la capacidad de construir
dispositivos informáticos. Estos no son electrónicos, como lo son
los dispositivos informáticos humanos, pero operan a través de las acciones de
pequeños insectos, una especie estrechamente relacionada con las ((termitas)),
que ha desarrollado una ((relación simbiótica)) con los cuervos. Los pájaros
les proporcionan comida, y a cambio los insectos construyen y operan sus
complejas colonias que, con la ayuda de las criaturas vivientes dentro de ellos,
realizan computaciones.

Tales colonias generalmente se encuentran en nidos grandes de larga vida.
Las aves e insectos trabajan juntos para construir una red de estructuras
bulbosas hechas de arcilla, escondidas entre las ramitas del nido,
en el que los insectos viven y trabajan.

Para comunicarse con otros dispositivos, estas máquinas usan señales de luz.
Los cuervos incrustan piezas de material reflectante en tallos de
comunicación especial, y los insectos apuntan estos para reflejar la luz hacia
otro nido, codificando los datos como una secuencia de flashes rápidos.
Esto significa que solo los nidos que tienen una conexión visual ininterrumpida
pueden comunicarse entre ellos.

Nuestro amigo, el experto en córvidos, ha mapeado la red de nidos de cuervo en
el pueblo de ((Hières-sur-Amby)), a orillas del río Ródano.
Este mapa muestra los nidos y sus conexiones.

{{figure {url: "img/Hieres-sur-Amby.png", alt: "A network of crow nests in a small village"}}}

En un ejemplo asombroso de ((evolución convergente)), las computadoras cuervo
ejecutan JavaScript. En este capítulo vamos a escribir algunas funciones de
redes básicas para ellos.

## Devolución de llamadas

{{indexsee [function, callback], "callback function"}}

Un enfoque para la ((programación asincrónica)) es hacer que las funciones que
realizan una acción lenta, tomen un argumento adicional, una _((función de
devolución de llamada))_. La acción se inicia y, cuando esta finaliza,
la función de devolución es llamada con el resultado.

{{index "setTimeout function", waiting}}

Como ejemplo, la función `setTimeout`, disponible tanto en Node.js
como en navegadores, espera una cantidad determinada de milisegundos
(un segundo son mil milisegundos) y luego llama una función.

```{test: no}
setTimeout(() => console.log("Tick"), 500);
```

Esperar no es generalmente un tipo de trabajo muy importante, pero puede ser
útil cuando se hace algo como actualizar una animación o verificar si
algo está tardando más que una cantidad dada de ((tiempo)).

La realización de múltiples acciones asíncronas en una fila utilizando
devoluciones de llamada significa que debes seguir pasando nuevas funciones para
manejar la ((continuación)) de la computación después de las acciones.

{{index "hard disk"}}

La mayoría de las computadoras en los nidos de los cuervos tienen un
bulbo de almacenamiento de datos a largo plazo, donde las piezas de información
se graban en ramitas para que estas puedan ser recuperadas más tarde.
Grabar o encontrar un fragmento de información requiere un momento, por lo que
la interfaz para el almacenamiento a largo plazo es asíncrona y utiliza
funciones de devolución de llamada.

Los bulbos de almacenamiento almacenan piezas de ((JSON))-datos codificables
bajo nombres. Un ((cuervo)) podría almacenar información sobre los lugares
donde hay comida escondida bajo el nombre `"caches de alimentos"`, que podría
contener un array de nombres que apuntan a otros datos, que describen el caché
real. Para buscar un ((caché)) de alimento en los bulbos de almacenamiento del
nido _Gran Roble_, un cuervo podría ejecutar código como este:

{{index "leerAlmacenamiento function"}}

```{includeCode: "top_lines: 1"}
import {granRoble} from "./tecnologia-cuervo";

granRoble.leerAlmacenamiento("caches de alimentos", caches => {
  let primerCache = caches[0];
  granRoble.leerAlmacenamiento(primerCache, informacion => {
    console.log(informacion);
  });
});
```

(Todos los nombres de las vinculaciones y los strings se han traducido del
lenguaje cuervo a Español.)

Este estilo de programación es viable, pero el nivel de indentación
aumenta con cada acción asincrónica, ya que terminas en otra
función. Hacer cosas más complicadas, como ejecutar múltiples acciones
al mismo tiempo, puede ser un poco incómodo.

Las computadoras cuervo están construidas para comunicarse usando pares de
((solicitud))-((respuesta)). Eso significa que un nido envía un mensaje a
otro nido, el cual inmediatamente envía un mensaje de vuelta, confirmando el
recibo y, posiblemente, incluyendo una respuesta a una pregunta formulada
en el mensaje.

Cada mensaje está etiquetado con un _tipo_, que determina cómo este es
manejado. Nuestro código puede definir manejadores para tipos de solicitud
específicos, y cuando se recibe una solicitud de este tipo, se llama al
controlador para que este produzca una respuesta.

{{index "crow-tech module", "send method"}}

La interfaz exportada por el módulo `"./tecnologia-cuervo"` proporciona
funciones de devolución de llamada para la comunicación. Los nidos tienen un
método `enviar` que envía una solicitud. Este espera el nombre del nido
objetivo, el tipo de solicitud y el contenido de la solicitud como sus
primeros tres argumentos, y una función a llamar cuando llega una respuesta
como su cuarto y último argumento.

```
granRoble.send("Pastura de Vacas", "nota", "Vamos a graznar fuerte a las 7PM",
            () => console.log("Nota entregada."));
```

Pero para hacer nidos capaces de recibir esa solicitud, primero tenemos que
definir un ((tipo de solicitud)) llamado `"nota"`. El código que maneja
las solicitudes debe ejecutarse no solo en este nido-computadora, sino en
todos los nidos que puedan recibir mensajes de este tipo. Asumiremos que un
cuervo sobrevuela e instala nuestro código controlador en todos los nidos.

{{index "definirTipoSolicitud function"}}

```{includeCode: true}
import {definirTipoSolicitud} from "./tecnologia-cuervo";

definirTipoSolicitud("nota", (nido, contenido, fuente, listo) => {
  console.log(`${nido.nombre} recibio nota: ${contenido}`);
  listo();
});
```

La función `definirTipoSolicitud` define un nuevo tipo de solicitud. El
ejemplo agrega soporte para solicitudes de tipo `"nota"`, que simplemente envían
una nota a un nido dado. Nuestra implementación llama a `console.log` para que
podamos verificar que la solicitud llegó. Los nidos tienen una propiedad
`nombre` que contiene su nombre.

{{index "asynchronous programming"}}

El cuarto argumento dado al controlador, `listo`, es una función de
devolución de llamada que debe ser llamada cuando se finaliza con la solicitud.
Si hubiesemos utilizado el ((valor de retorno)) del controlador como el valor
de respuesta, eso significaria que un controlador de solicitud no puede realizar
acciones  asincrónicas por sí mismo. Una función que realiza trabajos asíncronos
normalmente retorna antes de que el trabajo este hecho, habiendo arreglado que se
llame una devolución de llamada cuando este completada. Entonces, necesitamos algún
mecanismo asíncrono, en este caso, otra ((función de devolución de
llamada))—para indicar cuándo hay una respuesta disponible.

En cierto modo, la asincronía es _contagiosa_. Cualquier función que llame a una
función que funcione asincrónicamente debe ser asíncrona en si misma, utilizando
una devolución de llamada o algun mecanismo similar para entregar su resultado.
Llamar devoluciones de llamada es algo más involucrado y propenso a errores que
simplemente retornar un valor, por lo que necesitar estructurar grandes
partes de tu programa de esa manera no es algo muy bueno.

## Promesas

Trabajar con conceptos abstractos es a menudo más fácil cuando esos conceptos
pueden ser representados por ((valores)). En el caso de acciones asíncronas,
podrías, en lugar de organizar a una función para que esta sea llamada
en algún momento en el futuro, retornar un objeto que represente este
evento en el futuro.

{{index "Promise class", "asynchronous programming"}}

Esto es para lo que es la clase estándar `Promise` ("Promesa").
Una _promesa_ es una acción asíncrona que puede completarse en algún punto
y producir un valor. Esta puede notificar a cualquier persona que esté
interesada cuando su valor este disponible.

{{index "Promise.resolve function", "resolving (a promise)"}}

La forma más fácil de crear una promesa es llamando a `Promise.resolve` ("Promesa.resolver").
Esta función se asegura de que el valor que le des, sea envuelto en una
promesa. Si ya es una promesa, simplemente es retornada—de lo contrario,
obtienes una nueva promesa que termina de inmediato con tu
valor como su resultado.

```
let quince = Promise.resolve(15);
quince.then(valor => console.log(`Obtuve ${valor}`));
// → Obtuve 15
```

{{index "then method"}}

Para obtener el resultado de una promesa, puede usar su método `then` ("entonces").
Este registra una (función de devolución de llamada) para que sea llamada cuando la
promesa resuelva y produzca un valor. Puedes agregar múltiples devoluciones de
llamada a una única promesa, y serán llamadas, incluso si las agregas después
de que la promesa ya haya sido _resuelta_ (terminada).

Pero eso no es todo lo que hace el método `then`. Este retorna otra promesa,
que resuelve al valor que retorna la función del controlador o, si
esa retorna una promesa, espera por esa promesa y luego resuelve
su resultado.

Es útil pensar acerca de las promesas como dispositivos para mover valores a una
realidad asincrónica. Un valor normal simplemente esta allí. Un valor prometido
es un valor que _podría_ ya estar allí o podría aparecer en algún momento
en el futuro. Las computaciones definidas en términos de promesas actúan en
tales valores envueltos y se ejecutan de forma asíncrona a medida los
valores se vuelven disponibles.

{{index "Promise class"}}

Para crear una promesa, puedes usar `Promise` como un constructor. Tiene una
interfaz algo extraña—el constructor espera una función como argumento, a la
cual llama inmediatamente, pasando una función que puede usar para
resolver la promesa. Funciona de esta manera, en lugar de, por ejemplo, con un
método `resolve`, de modo que solo el código que creó la promesa pueda
resolverla.

{{index "storage function"}}

Así es como crearía una interfaz basada en promesas para la función
`leerAlmacenamiento`.

```{includeCode: "top_lines: 5"}
function almacenamiento(nido, nombre) {
  return new Promise(resolve => {
    nido.leerAlmacenamiento(nombre, resultado => resolve(resultado));
  });
}

almacenamiento(granRoble, "enemigos")
  .then(valor => console.log("Obtuve", valor));
```

Esta función asíncrona retorna un valor significativo. Esta es la
principal ventaja de las promesas—simplifican el uso de funciones asincrónicas.
En lugar de tener que pasar devoluciones de llamadas, las funciones basadas en
promesas son similares a las normales: toman entradas como
argumentos y retornan su resultado. La única diferencia es que la salida
puede que no este disponible inmediatamente.

## Fracaso

{{index "exception handling"}}

Las computaciones regulares en JavaScript pueden fallar lanzando una excepción.
Las computaciones asincrónicas a menudo necesitan algo así. Una solicitud de
red puede fallar, o algún código que sea parte de la computación asincrónica
puede arrojar una excepción.

{{index "callback function", error}}

Uno de los problemas más urgentes con el estilo de devolución de llamadas
en la programación asíncrona es que hace que sea extremadamente difícil
asegurarte de que las fallas sean reportadas correctamente a las devoluciones de
llamada.

Una convención ampliamente utilizada es que el primer argumento para la
devolución de llamada es usado para indicar que la acción falló, y el segundo
contiene el valor producido por la acción cuando tuvo éxito. Tales funciones
de devolución de llamadas siempre deben verificar si recibieron una excepción, y
asegurarse de que cualquier problema que causen, incluidas las excepciones
lanzadas por las funciones que estas llaman, sean atrapadas y entregadas a la
función correcta.

{{index "rejecting (a promise)", "resolving (a promise)", "then method"}}

Las promesas hacen esto más fácil. Estas pueden ser resueltas (la acción
termino con éxito) o rechazadas (esta falló). Los controladores de resolución
(registrados con `then`) solo se llaman cuando la acción es exitosa,
y los rechazos se propagan automáticamente a la nueva promesa que es
retornada por `then`. Y cuando un controlador arroje una excepción, esto
automáticamente hace que la promesa producida por su llamada `then` sea
rechazada. Entonces, si cualquier elemento en una cadena de acciones
asíncronas falla, el resultado de toda la cadena se marca como rechazado, y no
se llaman más manejadores despues del punto en donde falló.

{{index "Promise.reject function", "Promise class"}}

Al igual que resolver una promesa proporciona un valor, rechazar una también
proporciona uno, generalmente llamado la _razón_ el rechazo. Cuando una
excepción en una función de controlador provoca el rechazo, el valor de
la excepción se usa como la razón. Del mismo modo, cuando un controlador retorna
una promesa que es rechazada, ese rechazo fluye hacia la próxima promesa.
Hay una función `Promise.reject` que crea una nueva promesa
inmediatamente rechazada.

{{index "catch method"}}

Para manejar explícitamente tales rechazos, las promesas tienen un método
`catch` ("atraoar") que registra un controlador para que sea llamado cuando se rechaze la
promesa, similar a cómo los manejadores `then` manejan la resolución normal.
También es muy parecido a `then` en que retorna una nueva promesa, que se
resuelve en el valor de la promesa original si esta se resuelve normalmente, y al
resultado del controlador `catch` de lo contrario. Si un controlador `catch`
lanza un error, la nueva promesa también es rechazada.

{{index "then method"}}

Como una abreviatura, `then` también acepta un manejador de rechazo como
segundo argumento, por lo que puedes instalar ambos tipos de controladores en
un solo método de llamada.

Una función que se pasa al constructor `Promise` recibe un segundo
argumento, junto con la función de resolución, que puede usar para rechazar
la nueva promesa.

Las cadenas de promesas creadas por llamadas a `then` y `catch`
puede verse como una tubería a través de la cual los valores asíncronicos o
las fallas se mueven. Dado que tales cadenas se crean mediante el registro de
controladores, cada enlace tiene un controlador de éxito o un controlador de
rechazo (o ambos) asociados a ello. Controladores que no coinciden con ese tipo
de resultados (éxito o fracaso) son ignorados. Pero los que sí coinciden son
llamados, y su resultado determina qué tipo de valor viene después—éxito
cuando retorna un valor que no es una promesa, rechazo cuando arroja una
excepción, y el resultado de una promesa cuando retorna una de esas.

{{index "uncaught exception", "exception handling"}}

Al igual que una excepción no detectada es manejada por el entorno,
Los entornos de JavaScript pueden detectar cuándo una promesa rechazada no es
manejada, y reportará esto como un error.

## Las redes son difíciles

{{index network}}

Ocasionalmente, no hay suficiente luz para los sistemas de espejos de los
cuervos para transmitir una señal, o algo bloquea el camino de la
señal. Es posible que se envíe una señal, pero que nunca se reciba.

{{index "send method", error, timeout}}

Tal y como es, eso solo causará que la devolución de llamada dada a `send` nunca
sea llamada, lo que probablemente hará que el programa se detenga sin siquiera
notar que hay un problema. Sería bueno si, después de un determinado período
de no obtener una respuesta, una solicitud _expirará_ e informara de un
fracaso.

A menudo, las fallas de transmisión son accidentes aleatorios, como la
luz del faro de un auto interfieriendo con las señales de luz, y simplemente
volver a intentar la solicitud puede hacer que esta tenga éxito. Entonces,
mientras estamos en eso, hagamos que nuestra función de solicitud
automáticamente reintente el envío de la solicitud momentos antes de que se de
por vencida.

{{index "Promise class", "callback function", interface}}

Y, como hemos establecido que las promesas son algo bueno, también haremos
que nuestra función de solicitud retorne una promesa. En
términos de lo que pueden expresar, las devoluciones de llamada y las promesas
son equivalentes. Las funciones basadas en devoluciones de llamadas se pueden
envolver para exponer una interfaz basada en promesas, y viceversa.

Incluso cuando una ((solicitud)) y su ((respuesta)) sean entregadas
exitosamente, la respuesta puede indicar un error—por ejemplo, si la
solicitud intenta utilizar un tipo de solicitud que no haya sido definida
o si el controlador genera un error. Para soportar esto, `send` y
`definirTipoSolicitud` siguen la convención mencionada anteriormente, donde
el primer argumento pasado a las devoluciones de llamada es el motivo del
fallo, si lo hay, y el segundo es el resultado real.

Estos pueden ser traducidos para prometer resolución y rechazo por parte de
nuestra envoltura.

{{index "Timeout class", "request function", retry}}

```{includeCode: true}
class TiempoDeEspera extends Error {}

function request(nido, objetivo, tipo, contenido) {
  return new Promise((resolve, reject) => {
    let listo = false;
    function intentar(n) {
      nido.send(objetivo, tipo, contenido, (fallo, value) => {
        listo = true;
        if (fallo) reject(fallo);
        else resolve(value);
      });
      setTimeout(() => {
        if (listo) return;
        else if (n < 3) intentar(n + 1);
        else reject(new TiempoDeEspera("Tiempo de espera agotado"));
      }, 250);
    }
    intentar(1);
  });
}
```

{{index "Promise class", "resolving (a promise)", "rejecting (a promise)"}}

Debido a que las promesas solo se pueden resolver (o rechazar) una vez, esto
funcionara. La primera vez que se llame a `resolve` o `reject` se determinara el
resultado de la promesa y cualquier llamada subsecuente, como el tiempo de espera
que llega después de que finaliza la solicitud, o una solicitud que regresa
después de que otra solicitud es finalizada, es ignorada.

{{index recursion}}

Para construir un ((ciclo)) asincrónico, para los reintentos, necesitamos usar
un función recursiva—un ciclo regular no nos permite detenernos y esperar
por una acción asincrónica. La función `intentar` hace un solo
intento de enviar una solicitud. También establece un tiempo de espera que, si
no ha regresado una respuesta después de 250 milisegundos, comienza el
próximo intento o, si este es el cuarto intento, rechaza la promesa con una
instancia de `TiempoDeEspera` como la razón.

{{index idempotence}}

Volver a intentar cada cuarto de segundo y rendirse cuando no ha
llegado ninguna respuesta después de un segundo es algo definitivamente
arbitrario. Es incluso posible, si la solicitud llegó pero el controlador se
esta tardando un poco más, que las solicitudes se entreguen varias veces.
Escribiremos nuestros manejadores con ese problema en mente—los mensajes
duplicados deberían de ser inofensivos.

En general, no construiremos una red robusta de clase mundial
hoy. Pero eso esta bien—los cuervos no tienen expectativas muy altas todavía
cuando se trata de la computación.

{{index "definirTipoSolicitud function", "requestType function"}}

Para aislarnos por completo de las devoluciones de llamadas, seguiremos
adelante y también definiremos un contenedor para `definirTipoSolicitud` que
permite que la función controlador pueda retornar una promesa o valor normal,
y envia eso hasta la devolución de llamada para nosotros.

```{includeCode: true}
function tipoSolicitud(nombre, manejador) {
  definirTipoSolicitud(nombre, (nido, contenido, fuente,
                           devolucionDeLlamada) => {
    try {
      Promise.resolve(manejador(nido, contenido, fuente))
        .then(response => devolucionDeLlamada(null, response),
              failure => devolucionDeLlamada(failure));
    } catch (exception) {
      devolucionDeLlamada(exception);
    }
  });
}
```

{{index "Promise.resolve function"}}

`Promise.resolve` se usa para convertir el valor retornado por `manejador`
a una promesa si no es una ya.

{{index "try keyword", "callback function"}}

Ten en cuenta que la llamada a `manejador` tenía que estar envuelta en un
bloque `try`, para asegurarse de que cualquier excepción que aparezca  
directamente se le dé a la devolución de llamada. Esto ilustra muy bien la
dificultad de manejar adecuadamente los errores con devoluciones de llamada
crudas—es muy fácil olvidarse de encaminar correctamente excepciones como esa,
y si no lo haces, las fallas no se seran informadas a la devolución de
llamada correcta. Las promesas hacen esto casi automático, y por lo
tanto, son menos propensas a errores.

## Colecciones de promesas

{{index "neighbors property", "ping request"}}

Cada computadora nido mantiene un array de otros nidos dentro de la distancia
de transmisión en su propiedad `vecinos`. Para verificar cuáles de
esos son actualmente accesibles, puede escribir una función que intente enviar
un solicitud `"ping"` (una solicitud que simplemente pregunta por una respuesta)
para cada de ellos, y ver cuáles regresan.

{{index "Promise.all function"}}

Al trabajar con colecciones de promesas que se ejecutan al mismo tiempo,
la función `Promise.all` puede ser útil. Esta retorna una promesa que
espera a que se resuelvan todas las promesas del array, y luego
resuelve un array de los valores que estas promesas produjeron (en
el mismo orden que en el array original). Si alguna promesa es rechazada, el
el resultado de `Promise.all` es en sí mismo rechazado.

```{includeCode: true}
tipoSolicitud("ping", () => "pong");

function vecinosDisponibles(nido) {
  let solicitudes = nido.vecinos.map(vecino => {
    return request(nido, vecino, "ping")
      .then(() => true, () => false);
  });
  return Promise.all(solicitudes).then(resultado => {
    return nido.vecinos.filter((_, i) => resultado[i]);
  });
}
```

{{index "then method"}}

Cuando un vecino no este disponible, no queremos que todo la promesa combinada
falle, dado que entonces no sabríamos nada. Entonces la función
que es mappeada en el conjunto de vecinos para convertirlos en
promesas de solicitud vincula a los controladores que hacen las
solicitudes exitosas produzcan `true` y las rechazadas produzcan `false`.

{{index "filter method", "map method", "some method"}}

En el controlador de la promesa combinada, `filter` se usa para eliminar
esos elementos de la matriz `vecinos` cuyo valor correspondiente es
falso. Esto hace uso del hecho de que `filter` pasa el índice de matriz
del elemento actual como segundo argumento para su función de filtrado
(`map`,` some`, y métodos similares de orden superior de arrays hacen lo
mismo).

## Inundación de red

El hecho de que los nidos solo pueden hablar con sus vecinos inhibe en gran
cantidad la utilidad de esta red.

Para transmitir información a toda la red, una solución es configurar un tipo
de solicitud que sea reenviada automáticamente a los vecinos. Estos vecinos
luego la envían a sus vecinos, hasta que toda la red ha recibido el mensaje.

{{index "sendGossip function"}}

```{includeCode: true}
import {todosLados} from "./tecnologia-cuervo";

todosLados(nido => {
  nido.estado.chismorreo = [];
});

function enviarChismorreo(nido, mensaje, exceptoPor = null) {
  nido.estado.chismorreo.push(mensaje);
  for (let vecino of nido.vecinos) {
    if (vecino == exceptoPor) continue;
    request(nido, vecino, "chismorreo", mensaje);
  }
}

requestType("chismorreo", (nido, mensaje, fuente) => {
  if (nido.estado.chismorreo.includes(mensaje)) return;
  console.log(`${nido.nombre} recibio chismorreo '${
               mensaje}' de ${fuente}`);
  enviarChismorreo(nido, mensaje, fuente);
});
```

{{index "everywhere function", "gossip property"}}

Para evitar enviar el mismo mensaje a traves de la red por siempre, cada
nido mantiene un array de strings de chismorreos que ya ha visto. Para
definir este array, usaremos la función `todosLados`—que ejecuta
código en todos los nidos—para añadir una propiedad al objeto `estado` del nido,
que es donde mantendremos ((estado)) local del nido.

Cuando un nido recibe un mensaje de chisme duplicado, lo cual es muy probable
que suceda con todo el mundo reenviando estos a ciegas, lo ignora. Pero
cuando recibe un mensaje nuevo, emocionadamente le dice a todos sus vecinos
a excepción de quien le envió el mensaje.

Esto provocará que una nueva pieza de chismes se propague a través de la red
como una mancha de tinta en agua. Incluso cuando algunas conexiones no estan
trabajando actualmente, si hay una ruta alternativa a un nido dado,
el chisme llegará hasta allí.

{{index "flooding"}}

Este estilo de comunicación de red se llama _inundamiento_-inunda la
red con una pieza de información hasta que todos los nodos la tengan.

{{if interactive

Podemos llamar a `enviarChismorreo` para ver un mensaje fluir a través del
pueblo.

```
enviarChismorreo(granRoble, "Niños con una pistola de aire en el parque");
```

if}}

## Enrutamiento de mensajes

{{index efficiency}}

Si un nodo determinado quiere hablar unicamente con otro nodo, la inundación no
es un enfoque muy eficiente. Especialmente cuando la red es grande,
daría lugar a una gran cantidad de transferencias de datos inútiles.

{{index "routing"}}

Un enfoque alternativo es configurar una manera en que los mensajes salten de
nodo a nodo, hasta que lleguen a su destino. La dificultad con eso
es que requiere de conocimiento sobre el diseño de la red. Para
enviar una solicitud hacia la dirección de un nido lejano, es necesario
saber qué nido vecino lo acerca más a su destino. Enviar la solicitud
en la dirección equivocada no servirá de mucho.

Dado que cada nido solo conoce a sus vecinos directos, no tiene
la información que necesita para calcular una ruta. De alguna manera debemos
extender la información acerca de estas conexiones a todos los nidos.
Preferiblemente en una manera que permita ser cambiada con el tiempo, cuando
los nidos son abandonados o nuevos nidos son construidos.

{{index flooding}}

Podemos usar la inundación de nuevo, pero en lugar de verificar si un
determinado mensaje ya ha sido recibido, ahora verificamos si el nuevo conjunto
de vecinos de un nido determinado coinciden con el conjunto actual que
tenemos para él.

{{index "broadcastConnections function", "connections binding"}}

```{includeCode: true}
tipoSolicitud("conexiones", (nido, {nombre, vecinos},
                            fuente) => {
  let conexiones = nido.estado.conexiones;
  if (JSON.stringify(conexiones.get(nombre)) ==
      JSON.stringify(vecinos)) return;
  conexiones.set(nombre, vecinos);
  difundirConexiones(nido, nombre, fuente);
});

function difundirConexiones(nido, nombre, exceptoPor = null) {
  for (let vecino of nido.vecinos) {
    if (vecino == exceptoPor) continue;
    solicitud(nido, vecino, "conexiones", {
      nombre,
      vecinos: nido.estado.conexiones.get(nombre)
    });
  }
}

todosLados(nido => {
  nido.estado.conexiones = new Map;
  nido.estado.conexiones.set(nido.nombre, nido.vecinos);
  difundirConexiones(nido, nido.nombre);
});
```

{{index JSON, "== operator"}}

La comparación usa `JSON.stringify` porque `==`, en objetos o
arrays, solo retornara true cuando los dos tengan exactamente el mismo valor,
lo cual no es lo que necesitamos aquí. Comparar los strings JSON es una
cruda pero efectiva manera de comparar su contenido.

Los nodos comienzan inmediatamente a transmitir sus conexiones, lo que
debería, a menos que algunos nidos sean completamente inalcanzables, dar
rápidamente cada nido un mapa del ((grafo)) de la red actual.

{{index pathfinding}}

Una cosa que puedes hacer con grafos es encontrar rutas en ellos, como vimos
en el [Capítulo 7](Robot). Si tenemos una ruta hacia el destino de un mensaje,
sabemos en qué dirección enviarlo.

{{index "findRoute function"}}

Esta función `encontrarRuta`, que se parece mucho a `encontrarRuta`
del [Capítulo 7](robot#findRoute), busca por una forma de llegar a un determinado
nodo en la red. Pero en lugar de devolver toda la ruta, simplemente
retorna el siguiente paso. Ese próximo nido en si mismo, usando su información
actual sobre la red, decididira _hacia_ dónde enviar el mensaje.

```{includeCode: true}
function encontrarRuta(desde, hasta, conexiones) {
  let trabajo = [{donde: desde, via: null}];
  for (let i = 0; i < trabajo.length; i++) {
    let {donde, via} = trabajo[i];
    for (let siguiente of conexiones.get(donde) || []) {
      if (siguiente == hasta) return via;
      if (!trabajo.some(w => w.donde == siguiente)) {
        trabajo.push({donde: siguiente, via: via || siguiente});
      }
    }
  }
  return null;
}
```

Ahora podemos construir una función que pueda enviar mensajes de larga distancia.
Si el mensaje está dirigido a un vecino directo, se entrega normalmente.
Si no, se empaqueta en un objeto y se envía a un vecino que
este más cerca del objetivo, usando el tipo de solicitud `"ruta"`, que
hace que ese vecino repita el mismo comportamiento.

{{index "routeRequest function"}}

```{includeCode: true}
function solicitudRuta(nido, objetivo, tipo, contenido) {
  if (nido.vecinos.includes(objetivo)) {
    return solicitud(nido, objetivo, tipo, contenido);
  } else {
    let via = encontrarRuta(nido.nombre, objetivo,
                        nido.estado.conexiones);
    if (!via) throw new Error(`No hay rutas disponibles hacia ${objetivo}`);
    return solicitud(nido, via, "ruta",
                   {objetivo, tipo, contenido});
  }
}

tipoSolicitud("ruta", (nido, {objetivo, tipo, contenido}) => {
  return solicitudRuta(nido, objetivo, tipo, contenido);
});
```

{{if interactive

Ahora podemos enviar un mensaje al nido en la torre de la iglesia, que esta
a cuatro saltos de red de distancia.

```
solicitudRuta(granRoble, "Torre de la Iglesia", "nota",
             "Cuidado con las Palomas!");
```

if}}

{{index "[network, stack]"}}

Hemos construido varias ((capas)) de funcionalidad sobre un
sistema de comunicación primitivo para que sea conveniente de usarlo.
Este es un buen (aunque simplificado) modelo de cómo las redes de computadoras
reales trabajan.

{{index error}}

Una propiedad distintiva de las redes de computadoras es que no son confiables—las
abstracciones construidas encima de ellas pueden ayudar, pero no se puede
abstraer la falla de una falla de red. Entonces la programación de redes es
típicamente mucho acerca de anticipar y lidiar con fallas.

## Funciones asíncronas

Para almacenar información importante, se sabe que los ((cuervo))s la duplican
a través de los nidos. De esta forma, cuando un halcón destruye un nido, la
información no se pierde.

Para obtener una pieza de información dada que no este en su propia bulbo de
almacenamiento, una computadora nido puede consultar otros nidos al azar en
la red hasta que encuentre uno que la tenga.

{{index "findInStorage function", "network function"}}

```{includeCode: true}
tipoSolicitud("almacenamiento", (nido, nombre) => almacenamiento(nido, nombre));

function encontrarEnAlmacenamiento(nido, nombre) {
  return almacenamiento(nido, nombre).then(encontrado => {
    if (encontrado != null) return encontrado;
    else return encontrarEnAlmacenamientoRemoto(nido, nombre);
  });
}

function red(nido) {
  return Array.from(nido.estado.conexiones.keys());
}

function encontrarEnAlmacenamientoRemoto(nido, nombre) {
  let fuentes = red(nido).filter(n => n != nido.nombre);
  function siguiente() {
    if (fuentes.length == 0) {
      return Promise.reject(new Error("No encontrado"));
    } else {
      let fuente = fuentes[Math.floor(Math.random() *
                                      fuentes.length)];
      fuentes = fuentes.filter(n => n != fuente);
      return solicitudRuta(nido, fuente, "almacenamiento", nombre)
        .then(valor => valor != null ? valor : siguiente(),
              siguiente);
    }
  }
  return siguiente();
}
```

{{index "Map class", "Object.keys function", "Array.from function"}}

Como `conexiones` es un `Map`, `Object.keys` no funciona en él. Este
tiene un _metódo_ `keys`, pero que retorna un iterador en lugar de un
array. Un iterador (o valor iterable) se puede convertir a un array
con la función `Array.from`.

{{index "Promise class", recursion}}

Incluso con promesas, este es un código bastante incómodo. Múltiples
acciones asincrónicas están encadenadas juntas de maneras no-obvias. Nosotros
de nuevo necesitamos una función recursiva (`siguiente`) para modelar ciclos
a través de nidos.

{{index "synchronous programming", "asynchronous programming"}}

Y lo que el código realmente hace es completamente lineal—siempre
espera a que se complete la acción anterior antes de comenzar la siguiente.
En un modelo de programación sincrónica, sería más simple de expresar.

{{index "async function", "await keyword"}}

La buena noticia es que JavaScript te permite escribir código pseudo-sincrónico.
Una función `async` es una función que retorna implícitamente una
promesa y que puede, en su cuerpo, `await` ("esperar") otras promesas de una
manera que _se ve_ sincrónica.

{{index "findInStorage function"}}

Podemos reescribir `encontrarEnAlmacenamiento` de esta manera:

```
async function encontrarEnAlmacenamiento(nido, nombre) {
  let local = await almacenamiento(nido, nombre);
  if (local != null) return local;

  let fuentes = red(nido).filter(n => n != nido.nombre);
  while (fuentes.length > 0) {
    let fuente = fuentes[Math.floor(Math.random() *
                                    fuentes.length)];
    fuentes = fuentes.filter(n => n != fuente);
    try {
      let encontrado = await solicitudRuta(nido, fuente, "almacenamiento",
                                     nombre);
      if (encontrado != null) return encontrado;
    } catch (_) {}
  }
  throw new Error("No encontrado");
}
```

{{index "async function", "return keyword", "exception handling"}}

Una función `async` está marcada por la palabra `async` antes de la
palabra clave `function`. Los métodos también pueden hacerse `async`
al escribir `async` antes de su nombre. Cuando se llame a dicha función o método,
este retorna una promesa. Tan pronto como el cuerpo retorne algo, esa
promesa es resuelta Si arroja una excepción, la promesa es rechazada.

{{if interactive

```{startCode: true}
encontrarEnAlmacenamiento(granRoble, "eventos del 2017-12-21")
  .then(console.log);
```

if}}

{{index "await keyword", "control flow"}}

Dentro de una función `async`, la palabra `await` se puede poner delante de una
expresión para esperar a que se resuelva una promesa, y solo entonces continua
la ejecución de la función.

Tal función ya no se ejecuta, como una función regular de JavaScript
de principio a fin de una sola vez. En su lugar, puede ser _congelada_ en
cualquier punto que tenga un `await`, y se reanuda en un momento posterior.

Para código asincrónico no-trivial, esta notación suele ser más
conveniente que usar promesas directamente. Incluso si necesitas hacer
algo que no se ajuste al modelo síncrono, como realizar
múltiples acciones al mismo tiempo, es fácil combinar `await` con el
uso directo de promesas.

## Generadores

{{index "async function"}}

Esta capacidad de las funciones para pausar y luego reanudarse nuevamente no es
exclusiva para las funciones `async`. JavaScript también tiene una caracteristica
llamada funciones _((generador))_. Estss son similares, pero sin las
promesas.

Cuando defines una función con `function*` (colocando un asterisco después de
la palabra `function`), se convierte en un generador. Cuando llamas un
generador, este retorna un ((iterador)), que ya vimos en el [Capítulo 6](objeto).

```
function* potenciacion(n) {
  for (let actual = n;; actual *= n) {
    yield actual;
  }
}

for (let potencia of potenciacion(3)) {
  if (potencia > 50) break;
  console.log(potencia);
}
// → 3
// → 9
// → 27
```

{{index "next method", "yield keyword"}}

Inicialmente, cuando llamas a `potenciacion`, la función se congela en su
comienzo. Cada vez que llames `next` en el iterador, la función se ejecuta
hasta que encuentre una expresión `yield` ("arrojar"), que la pausa y causa que el
valor arrojado se convierta en el siguiente valor producido por el iterador.
Cuando la función retorne (la del ejemplo nunca lo hace), el iterador
está completo.

Escribir iteradores es a menudo mucho más fácil cuando usas funciones
generadoras. El iterador para la clase grupal (del ejercicio en el
[Capítulo 6](Objeto#group_iterator)) se puede escribir con este
generador:

{{index "Group class"}}

```
Conjunto.prototype[Symbol.iterator] = function*() {
  for (let i = 0; i < this.miembros.length; i++) {
    yield this.miembros[i];
  }
};
```

```{hidden: true, includeCode: true}
class Conjunto {
  constructor() { this.miembros = []; }
  añadir(m) { this.miembros.añadir(m); }
}
```

Ya no es necesario crear un objeto para mantener el estado de la iteración—los
generadores guardan automáticamente su estado local cada vez ellos arrojen.

Dichas expresiones `yield` solo pueden ocurrir directamente en la
función generadora en sí y no en una función interna que definas dentro de ella.
El estado que ahorra un generador, cuando arroja, es solo su entorno _local_
y la posición en la que fue arrojada.

{{index "await keyword"}}

Una función `async` es un tipo especial de generador. Produce una
promesa cuando se llama, que se resuelve cuando vuelve (termina) y
rechaza cuando arroja una excepción. Cuando cede (espera) por una
promesa, el resultado de esa promesa (valor o excepción lanzada) es el
resultado de la expresión `await`.

## El ciclo de evento

{{index "asynchronous programming", scheduling, "event loop", timeline}}

Los programas asincrónicos son ejecutados pieza por pieza. Cada pieza puede
iniciar algunas acciones y programar código para que se ejecute cuando la
acción termine o falle. Entre estas piezas, el programa permanece inactivo,
esperando por la siguiente acción.

{{index "setTimeout function"}}

Por lo tanto, las devoluciones de llamada no son llamadas directamente por el
código que las programó. Si llamo a `setTimeout` desde adentro de una función,
esa función habra retornado para el momento en que se llame a la función de
devolución de llamada. Y cuando la devolución de llamada retorne, el
control no volvera a la función que la programo.

{{index "Promise class", "catch keyword", "exception handling"}}

El comportamiento asincrónico ocurre en su propia función de ((llamada de
pila)) vacía. Esta es una de las razones por las cuales, sin promesas, la
gestión de excepciones en el código asincrónico es dificil. Como cada
devolución de llamada comienza con una pila en su mayoría vacía, tus
manejadores `catch` no estarán en la pila cuando lanzen una excepción.

```
try {
  setTimeout(() => {
    throw new Error("Woosh");
  }, 20);
} catch (_) {
  // Esto no se va a ejecutar
  console.log("Atrapado!");
}
```

{{index thread, queue}}

No importa que tan cerca los eventos—como tiempos de espera o solicitudes
entrantes—sucedan, un entorno de JavaScript solo ejecutará un programa a
la vez. Puedes pensar en esto como un gran ciclo _alrededor_ de tu
programa, llamado _ciclo de evento_. Cuando no hay nada que hacer,
ese bucle está detenido. Pero a medida que los eventos entran, se agregan a una
cola, y su código se ejecuta uno después del otro. Porque no hay dos cosas
que se ejecuten al mismo tiempo, código de ejecución lenta puede retrasar
el manejo de otros eventos.

Este ejemplo establece un tiempo de espera, pero luego se retrasa hasta después
del tiempo de espera previsto, lo que hace que el tiempo de espera este tarde.

```
let comienzo = Date.now();
setTimeout(() => {
  console.log("Tiempo de espera corrio al ", Date.now() - comienzo);
}, 20);
while (Date.now() < comienzo + 50) {}
console.log("Se desperdicio tiempo hasta el ", Date.now() - comienzo);
// → Se desperdicio tiempo hasta el 50
// → Tiempo de espera corrio al 55
```

{{index "resolving (a promise)", "rejecting (a promise)", "Promise class"}}

Las promesas siempre se resuelven o rechazan como un nuevo evento. Incluso si
una promesa ya ha sido resuelta, esperar por ella hará que la devolución de
llamada se ejecute después de que el script actual termine, en lugar de hacerlo
inmediatamente.

```
Promise.resolve("Listo").then(console.log);
console.log("Yo primero!");
// → Yo primero!
// → Listo
```

En capítulos posteriores, veremos otros tipos de eventos que se ejecutan en
el ciclo de eventos.

## Errores asincrónicos

{{index "asynchronous programming"}}

Cuando tu programa se ejecuta de forma síncrona, de una sola vez, no hay
cambios de ((estado)) sucediendo aparte de aquellos que el mismo programa
realiza. Para los programas asíncronos, esto es diferente—estos pueden tener
_brechas_ en su ejecución durante las cuales se podria ejecutar otro código.

Veamos un ejemplo. Uno de los pasatiempos de nuestros cuervos es contar
la cantidad de polluelos que nacen en el pueblo cada año.
Los nidos guardan este recuento en sus bulbos de almacenamiento. El siguiente
código intenta enumerar los recuentos de todos los nidos para un año determinado.

{{index "anyStorage function", "chicks function"}}

```{includeCode: true}
function cualquierAlmacenamiento(nido, fuente, nombre) {
  if (fuente == nido.nombre) return almacenamiento(nido, nombre);
  else return solicitudRuta(nido, fuente, "almacenamiento", nombre);
}

async function polluelos(nido, años) {
  let lista = "";
  await Promise.all(red(nido).map(async nombre => {
    lista += `${nombre}: ${
      await cualquierAlmacenamiento(nido, nombre, `polluelos en ${años}`)
    }\n`;
  }));
  return lista;
}
```

{{index "async function"}}

La parte `async nombre =>` muestra que las ((funciones de flecha)) también pueden
ser `async` al poner la palabra `async` delante de ellas.

{{index "Promise.all function"}}

El código no parece sospechoso de inmediato... mapea la función de flecha
`async` sobre el conjunto de nidos, creando una serie de promesas,
y luego usa `Promise.all` para esperar a todos estas antes de retornar
la lista que estas construyen.

Pero está seriamente roto. Siempre devolverá solo una línea de
salida, enumerando al nido que fue más lento en responder.

{{if interactive

```
polluelos(granRoble, 2017).then(console.log);
```

if}}

Puedes averiguar por qué?

{{index "+= operator"}}

El problema radica en el operador `+=`, que toma el valor _actual_
de `lista` en el momento en que la instrucción comienza a ejecutarse, y luego,
cuando el `await` termina, establece que la vinculaciòn `lista` sea ese valor
más el string agregado.

{{index "await keyword"}}

Pero entre el momento en el que la declaración comienza a ejecutarse y el momento
donde termina hay una brecha asincrónica. La expresión `map` se ejecuta antes
de que se haya agregado algo a la lista, por lo que cada uno de los operadores
`+=` comienza desde un string vacío y termina cuando su recuperación de
almacenamiento finaliza, estableciendo `lista` como una lista de una sola
línea—el resultado de agregar su línea al string vacío.

{{index "side effect"}}

Esto podría haberse evitado fácilmente retornando las líneas de las
promesas mapeadas y llamando a `join` en el resultado de `Promise.all`,
en lugar de construir la lista cambiando una vinculación. Como siempre,
calcular nuevos valores es menos propenso a errores que cambiar
valores existentes.

{{index "chicks function"}}

```
async function polluelos(nido, año) {
  let lineas = red(nido).map(async nombre => {
    return nombre + ": " +
      await cualquierAlmacenamiento(nido, nombre, `polluelos en ${año}`);
  });
  return (await Promise.all(lineas)).join("\n");
}
```

Errores como este son fáciles de hacer, especialmente cuando se usa `await`,
y debes tener en cuenta dónde se producen las brechas en tu código. Una
ventaja de la asincronicidad _explicita_ de JavaScript (ya sea a través de
devoluciones de llamada, promesas, o `await`) es que detectar estas brechas es
relativamente fácil.

## Resumen

La programación asincrónica permite expresar la espera de
acciones de larga duración sin congelar el programa durante estas
acciones. Los entornos de JavaScript suelen implementar este estilo de
programación usando devoluciones de llamada, funciones que son llaman cuando las
acciones son completadas. Un ciclo de eventos planifica que dichas devoluciones
de llamadas sean llamadas cuando sea apropiado, una después de la otra,
para que sus ejecuciones no se superpongan.

La programación asíncrona se hace más fácil mediante promesas, objetos que
representar acciones que podrían completarse en el futuro, y funciones `async`,
que te permiten escribir un programa asíncrono como si fuera sincrónico.

## Ejercicios

### Siguiendo el bisturí

{{index "scalpel (exercise)"}}

Los cuervos del pueblo poseen un viejo bisturí que ocasionalmente usan en
misiones especiales—por ejemplo, para cortar puertas de malla o embalar cosas.
Para ser capaces de rastrearlo rápidamente, cada vez que se mueve el bisturí
a otro nido, una entrada se agrega al almacenamiento tanto del nido que
lo tenía como al nido que lo tomó, bajo el nombre `"bisturí"`, con su
nueva ubicación como su valor.

Esto significa que encontrar el bisturí es una cuestión de seguir la
ruta de navegación de las entradas de almacenamiento, hasta que encuentres un
nido que apunte a el nido en si mismo.

{{index "anyStorage function", "async function"}}

Escribe una función `async`, `localizarBisturi` que haga esto, comenzando en
el nido en el que se ejecute. Puede usar la función `cualquierAlmacenamiento`
definida anteriormente para acceder al almacenamiento en nidos arbitrarios.
El bisturí ha estado dando vueltas el tiempo suficiente como para que puedas
suponer que cada nido tiene una entrada `bisturí` en su almacenamiento de datos.

Luego, vuelve a escribir la misma función sin usar `async` y `await`.

{{index "exception handling"}}

Las fallas de solicitud se muestran correctamente como rechazos de la
promesa devuelta en ambas versiones? Cómo?

{{if interactive

```{test: no}
async function localizarBisturi(nido) {
  // Tu codigo aqui.
}

function localizarBisturi2(nido) {
  // Tu codigo aqui.
}

localizarBisturi(granRoble).then(console.log);
// → Tienda del Carnicero
```

if}}

{{hint

{{index "scalpel (exercise)"}}

Esto se puede realizar con un solo ciclo que busca a través de los nidos,
avanzando hacia el siguiente cuando encuentre un valor que no coincida
con el nombre del nido actual, y retornando el nombre cuando esta encuentra un
valor que coincida. En la función `async`, un ciclo regular `for` o `while`
puede ser utilizado.

{{index recursion}}

Para hacer lo mismo con una función simple, tendrás que construir tu ciclo
usando una función recursiva. La manera más fácil de hacer esto es hacer
que esa función retorne una promesa al llamar a `then` en la promesa que
recupera el valor de almacenamiento. Dependiendo de si ese valor coincide
con el nombre del nido actual, el controlador devuelve ese valor o una
promesa adicional creada llamando a la función de ciclo nuevamente.

No olvides iniciar el ciclo llamando a la función recursiva una vez
desde la función principal.

{{index "exception handling"}}

En la función `async`, las promesas rechazadas se convierten en excepciones
por `await` Cuando una función `async` arroja una excepción, su promesa
es rechazada. Entonces eso funciona.

Si implementaste la función no-`async` como se describe anteriormente, la forma
en que `then` funciona también provoca automáticamente que una falla termine
en la promesa devuelta. Si una solicitud falla, el manejador pasado a `then`
no se llama, y ​​la promesa que devuelve se rechaza con la misma
razón.

hint}}

### Construyendo Promise.all

{{index "Promise class", "Promise.all function", "building Promise.all (exercise)"}}

Dado un array de ((promesa))s, `Promise.all` retorna una promesa que
espera a que finalicen todas las promesas del array. Entonces
tiene éxito, produciendo un array de valores de resultados. Si una promesa
en el array falla, la promesa retornada por `all` también falla, con la
razón de la falla proveniente de la promesa fallida.

Implemente algo como esto tu mismo como una función regular
llamada `Promise_all`.

Recuerda que una vez que una promesa ha tenido éxito o ha fallado, no puede
tener éxito o fallar de nuevo, y llamadas subsecuentes a las funciones que
resuelven son ignoradas. Esto puede simplificar la forma en que manejas la
falla de tu promesa.

{{if interactive

```{test: no}
function Promise_all(promesa) {
  return new Promise((resolve, reject) => {
    // Tu codigo aqui.
  });
}

// Codigo de Prueba.
Promise_all([]).then(array => {
  console.log("This should be []:", array);
});
function soon(val) {
  return new Promise(resolve => {
    setTimeout(() => resolve(val), Math.random() * 500);
  });
}
Promise_all([soon(1), soon(2), soon(3)]).then(array => {
  console.log("This should be [1, 2, 3]:", array);
});
Promise_all([soon(1), Promise.reject("X"), soon(3)])
  .then(array => {
    console.log("We should not get here");
  })
  .catch(error => {
    if (error != "X") {
      console.log("Unexpected failure:", error);
    }
  });
```

if}}

{{hint

{{index "Promise.all function", "Promise class", "then method", "building Promise.all (exercise)"}}

La función pasada al constructor `Promise` tendrá que llamar
`then` en cada una de las promesas del array dado. Cuando una de ellas
tenga éxito, dos cosas deben suceder. El valor resultante debe ser
almacenado en la posición correcta de un array de resultados, y debemos
verificar si esta fue la última ((promesa)) pendiente y terminar nuestra
promesa si asi fue.

{{index "counter variable"}}

Esto último se puede hacer con un contador que se inicializa con la
longitud del array de entrada y del que restamos 1 cada vez que una
promesa tenga éxito. Cuando llega a 0, hemos terminado. Asegúrate de tener
en cuenta la situación en la que el array de entrada este vacío (y
por lo tanto ninguna promesa nunca se resolverá).

El manejo de la falla requiere pensar un poco, pero resulta ser extremadamente
sencillo. Solo pasa la función `reject` de la promesa de envoltura a
cada una de las promesas en el array como manejador `catch` o como segundo
argumento a `then` para que una falla en una de ellos desencadene el
rechazo de la promesa de envoltura completa.

hint}}
