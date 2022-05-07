{{meta {load_files: ["code/packages_chapter_10.js", "code/chapter/07_robot.js"]}}}

# Módulos

{{quote {author: "Tef", title: "Programming is Terrible", chapter: true}

Escriba código que sea fácil de borrar, no fácil de extender.

quote}}

{{index "Yuan-Ma", "Book of Programming"}}

{{figure {url: "img/chapter_picture_10.jpg", alt: "Picture of a building built from modular pieces", chapter: framed}}}

{{index organization, "code structure"}}

El programa ideal tiene una estructura cristalina. La forma en que funciona es
fácil de explicar, y cada parte juega un papel bien definido.

{{index "organic growth"}}

Un típico programa real crece orgánicamente. Nuevas piezas de funcionalidad
se agregan a medida que surgen nuevas necesidades. Estructurar—y preservar la
estructura—es trabajo adicional, trabajo que solo valdra la pena en el
futuro, la _siguiente_ vez que alguien trabaje en el programa. Así que es
tentador descuidarlo, y permitir que las partes del programa se vuelvan
profundamente enredadas.

{{index readability, reuse, isolation}}

Esto causa dos problemas prácticos. En primer lugar, entender tal sistema
es difícil. Si todo puede tocar todo lo demás, es difícil ver a cualquier
pieza dada de forma aislada. Estas obligado a construir un
entendimiento holístico de todo el asunto. En segundo lugar, si quieres
usar cualquiera de las funcionalidades de dicho programa en otra situación,
reescribirla podria resultar más fácil que tratar de desenredarla de su
contexto.

El término "((gran bola de barro))" se usa a menudo para tales programas
grandes, sin estructura. Todo se mantiene pegado, y cuando intentas
sacar una pieza, todo se desarma y tus manos se ensucian.

## Módulos

{{index dependency}}

Los _módulos_ son un intento de evitar estos problemas. Un ((módulo)) es una
pieza del programa que especifica en qué otras piezas este depende (
sus _dependencias_) y qué funcionalidad proporciona para que otros módulos
usen (su _((interfaz))_).

{{index "big ball of mud"}}

Las interfaces de los módulos tienen mucho en común con las interfaces de
objetos, como las vimos en el [Capítulo 6](objeto#interface). Estas hacen parte
del módulo disponible para el mundo exterior y mantienen el resto privado. Al
restringir las formas en que los módulos interactúan entre sí, el
el sistema se parece más a un juego de ((LEGOS)), donde las piezas interactúan
a través de conectores bien definidos, y menos como barro, donde todo se mezcla
con todo.

{{index dependency}}

Las relaciones entre los módulos se llaman _dependencias_. Cuando un módulo
necesita una pieza de otro módulo, se dice que depende de ese
módulo. Cuando este hecho está claramente especificado en el módulo en sí,
puede usarse para descubrir qué otros módulos deben estar presentes para poder
ser capaces de usar un módulo dado y cargar dependencias automáticamente.

Para separar módulos de esa manera, cada uno necesita su propio alcance privado.

Simplemente poner todo tu código JavaScript en diferentes ((archivo))s no
satisface estos requisitos. Los archivos aún comparten el mismo espacio de
nombres global. Pueden, intencionalmente o accidentalmente, interferir con
las vinculaciones de cada uno. Y la estructura de dependencia sigue sin
estar clara. Podemos hacerlo mejor, como veremos más adelante en el capítulo.

{{index design}}

Diseñar una estructura de módulo ajustada para un programa puede ser difícil.
En la fase en la que todavía estás explorando el problema, intentando
cosas diferentes para ver que funciona, es posible que desees no preocuparte
demasiado por eso, ya que puede ser una gran distracción. Una vez que tengas
algo que se sienta sólido, es un buen momento para dar un paso atrás y
organizarlo.

## Paquetes

{{index bug, dependency, structure, reuse}}

Una de las ventajas de construir un programa a partir de piezas separadas,
y ser capaces de ejecutar esas piezas por si mismas, es que tú
podrías ser capaz de aplicar la misma pieza en diferentes programas.

{{index "parseINI function"}}

Pero cómo se configura esto? Digamos que quiero usar la función `analizarINI`
del [Capítulo 9](regexp#ini) en otro programa. Si está claro de qué
depende la función (en este caso, nada), puedo copiar todo el
código necesario en mi nuevo proyecto y usarlo. Pero luego, si encuentro un
error en ese código, probablemente lo solucione en el programa
en el que estoy trabajando en ese momento y me olvido de arreglarlo en el otro
programa.

{{index duplication, "copy-paste programming"}}

Una vez que comience a duplicar código, rápidamente te encontraras perdiendo
tiempo y energía moviendo las copias alrededor y manteniéndolas actualizadas.

Ahí es donde los _((paquete))s_ entran. Un paquete es un pedazo de código que
puede ser distribuido (copiado e instalado). Puede contener uno o más
módulos, y tiene información acerca de qué otros paquetes depende.
Un paquete también suele venir con documentación que explica qué es
lo que hace, para que las personas que no lo escribieron todavía puedan hacer
uso de el.

Cuando se encuentra un problema en un paquete, o se agrega una nueva
característica, el el paquete es actualizado. Ahora los programas que dependen
de él (que también pueden ser otros paquetes) pueden actualizar a la
nueva ((versión)).

{{id modules_npm}}

{{index installation, upgrading, "package manager", download, reuse}}

Trabajar de esta manera requiere ((infraestructura)). Necesitamos un lugar para
almacenar y encontrar paquetes, y una forma conveniente de instalar y
actualizarlos. En el mundo de JavaScript, esta infraestructura es provista por
((NPM)) ([_npmjs.org_](https://npmjs.org)).

NPM es dos cosas: un servicio en línea donde uno puede descargar (y
subir) paquetes, y un programa (incluido con Node.js) que te ayuda a
instalar y administrarlos.

{{index "ini package"}}

Al momento de escribir esto, hay más de medio millón de paquetes diferentes
disponibles en NPM. Una gran parte de ellos son basura, debería mencionar,
pero casi todos los paquetes útiles, disponibles públicamente,
se puede encontrar allí. Por ejemplo, un analizador de archivos INI, similar al
uno que construimos en el [Capítulo 9](regexp), está disponible bajo el
nombre de paquete `ini`.

{{index "command line"}}

En el [Capítulo 20](node) veremos cómo instalar dichos paquetes de forma local
utilizando el programa de línea de comandos `npm`.

Tener paquetes de calidad disponibles para descargar es extremadamente valioso.
Significa que a menudo podemos evitar tener que reinventar un programa que cien
personas han escrito antes, y obtener una implementación sólida y bien probado
con solo presionar algunas teclas.

{{index maintenance}}

El software es barato de copiar, por lo que una vez lo haya escrito alguien,
distribuirlo a otras personas es un proceso eficiente. Pero escribirlo
en el primer lugar, _es_ trabajo y responder a las personas que han
encontrado problemas en el código, o que quieren proponer nuevas
características, es aún más trabajo.

Por defecto, tu posees el ((copyright)) del código que escribes, y otras
personas solo pueden usarlo con tu permiso. Pero ya que algunas personas
son simplemente agradables, y porque la publicación de un buen software
puede ayudarte a hacerte un poco famoso entre los programadores, se publican
muchos paquetes bajo una ((licencia)) que explícitamente permite a otras
personas usarlos.

La mayoría del código en ((NPM)) esta licenciado de esta manera. Algunas
licencias requieren que tu publiques también el código bajo la
misma licencia del paquete que estas usando. Otras son menos exigentes,
solo requieren que guardes la licencia con el código cuando lo distribuyas.
La comunidad de JavaScript principalmente usa ese último tipo de licencia.
Al usar paquetes de otras personas, asegúrete de conocer su licencia.

## Módulos improvisados

Hasta 2015, el lenguaje JavaScript no tenía un sistema de módulos incorporado.
Sin embargo, la gente había estado construyendo sistemas grandes en JavaScript
durante más de una década y ellos _necesitaban_ ((módulos)).

{{index [function, scope]}}

Así que diseñaron sus propios ((sistema de módulos)) arriba del lenguaje.
Puedes usar funciones de JavaScript para crear alcances locales, y
((objeto))s para representar las ((interfaces)) de los módulos.

{{index "Date class", "weekDay module"}}

Este es un módulo para ir entre los nombres de los días y números (como son
retornados por el método `getDay` de `Date`). Su interfaz consiste en
`diaDeLaSemana.nombre` y `diaDeLaSemana.numero`, y oculta su vinculación
local `nombres` dentro del alcance de una expresión de función que se invoca
inmediatamente.

```
const diaDeLaSemana = function() {
  const nombres = ["Domingo", "Lunes", "Martes", "Miercoles",
                 "Jueves", "Viernes", "Sabado"];
  return {
    nombre(numero) { return nombres[numero]; },
    numero(nombre) { return nombres.indexOf(nombre); }
  };
}();

console.log(diaDeLaSemana.nombre(diaDeLaSemana.numero("Domingo")));
// → Domingo
```

{{index dependency}}

Este estilo de módulos proporciona ((aislamiento)), hasta cierto punto, pero
no declara dependencias. En cambio, simplemente pone su ((interfaz)) en el
((alcance global)) y espera que sus dependencias, si hay alguna, hagan lo mismo.
Durante mucho tiempo, este fue el enfoque principal utilizado en la
programación web, pero ahora está mayormente obsoleto.

Si queremos hacer que las relaciones de dependencia sean parte del código,
tendremos que tomar el control de las dependencias que deben ser cargadas.
Hacer eso requiere que seamos capaces de ejecutar strings como código.
JavaScript puede hacer esto.

{{id eval}}

## Evaluando datos como código

{{index evaluation, interpretation}}

Hay varias maneras de tomar datos (un string de código) y ejecutarlos como
una parte del programa actual.

{{index isolation, eval}}

La forma más obvia es usar el operador especial `eval`, que ejecuta un
string en el ((alcance)) _actual_. Esto usualmente es una mala idea
porque rompe algunas de las propiedades que normalmente tienen los alcances,
tal como fácilmente predecir a qué vinculación se refiere un nombre dado.

```
const x = 1;
function evaluarYRetornarX(codigo) {
  eval(codigo);
  return x;
}

console.log(evaluarYRetornarX("var x = 2"));
// → 2
```

{{index "Function constructor"}}

Una forma menos aterradora de interpretar datos como código es usar el
constructor `Function`. Este toma dos argumentos: un string que contiene una
lista de nombres de argumentos separados por comas y un string que contiene el
cuerpo de la función.

```
let masUno = Function("n", "return n + 1;");
console.log(masUno(4));
// → 5
```

Esto es precisamente lo que necesitamos para un sistema de módulos. Podemos
envolver el código del módulo en una función y usar el alcance de esa función
como el ((alcance)) del módulo.

## CommonJS

{{id commonjs}}

{{index "CommonJS modules"}}

El enfoque más utilizado para incluir módulos en JavaScript es
llamado _módulos CommonJS_. ((Node.js)) lo usa, y es el sistema utilizado
por la mayoría de los paquetes en ((NPM)).

{{index "require function"}}

El concepto principal en los módulos CommonJS es una función llamada `require`
("requerir"). Cuando la llamas con el nombre del módulo de una dependencia,
esta se asegura de que el módulo sea cargado y retorna su ((interfaz)).

{{index "exports object"}}

Debido a que el cargador envuelve el código del módulo en una función, los
módulos obtienen automáticamente su propio alcance local. Todo lo que tienen que
hacer es llamar a `require` para acceder a sus dependencias, y poner su
interfaz en el objeto vinculado a `exports` ("exportaciones").

{{index "formatDate module", "Date class", "ordinal package", "date-names package"}}

Este módulo de ejemplo proporciona una función de formateo de fecha. Utiliza dos
((paquete))s de NPM—`ordinal` para convertir números a strings como
`"1st"` y `"2nd"`, y `date-names` para obtener los nombres en inglés de los
días de la semana y meses. Este exporta una sola función, `formatDate`, que
toma un objeto `Date` y un string ((plantilla)).

El string de plantilla puede contener códigos que dirigen el formato, como
`YYYY` para todo el año y `Do` para el día ordinal del mes.
Podrías darle un string como `"MMMM Do YYYY"` para obtener resultados como
"November 22nd 2017".

```
const ordinal = require("ordinal");
const {days, months} = require("date-names");

exports.formatDate = function(date, format) {
  return format.replace(/YYYY|M(MMM)?|Do?|dddd/g, tag => {
    if (tag == "YYYY") return date.getFullYear();
    if (tag == "M") return date.getMonth();
    if (tag == "MMMM") return months[date.getMonth()];
    if (tag == "D") return date.getDate();
    if (tag == "Do") return ordinal(date.getDate());
    if (tag == "dddd") return days[date.getDay()];
  });
};
```

{{index "destructuring binding"}}

La interfaz de `ordinal` es una función única, mientras que `date-names`
exporta un objeto que contiene varias cosas—los dos valores que usamos son
arrays de nombres. La desestructuración es muy conveniente cuando creamos
vinculaciones para interfaces importadas.

El módulo agrega su función de interfaz a `exports`, de modo que los módulos
que dependen de el tengan acceso a el. Podríamos usar el módulo de esta
manera:

```
const {formatDate} = require("./format-date");

console.log(formatDate(new Date(2017, 9, 13),
                       "dddd the Do"));
// → Friday the 13th
```

{{index "require function", "CommonJS modules", "readFile function"}}

{{id require}}

Podemos definir `require`, en su forma más mínima, así:

```{test: wrap, sandbox: require}
require.cache = Object.create(null);

function require(nombre) {
  if (!(nombre in require.cache)) {
    let codigo = leerArchivo(nombre);
    let modulo = {exportaciones: {}};
    require.cache[nombre] = modulo;
    let envolvedor = Function("require, exportaciones, modulo", codigo);
    envolvedor(require, modulo.exportaciones, modulo);
  }
  return require.cache[nombre].exportaciones;
}
```

En este código, `leerArchivo` es una función inventada que lee un archivo y
retorna su contenido como un string. El estándar de JavaScript no ofrece tal
funcionalidad—pero diferentes entornos de JavaScript, como el
navegador y Node.js, proporcionan sus propias formas de acceder a ((archivos)).
El ejemplo solo pretende que `leerArchivo` existe.

{{index cache, "Function constructor"}}

Para evitar cargar el mismo módulo varias veces, `require` mantiene un
(caché) almacenado de módulos que ya han sido cargados. Cuando se llama,
primero verifica si el módulo solicitado ya ha sido cargado y, si no, lo carga.
Esto implica leer el código del módulo, envolverlo en una función y
llamárla.

{{index "ordinal package", "exports object", "module object"}}

La ((interfaz)) del paquete `ordinal` que vimos antes no es un
objeto, sino una función. Una peculiaridad de los módulos CommonJS es que,
aunque el sistema de módulos creará un objeto de interfaz vacío para ti
(vinculado a `exports`), puedes reemplazarlo con cualquier valor al
sobrescribir `module.exports`. Esto lo hacen muchos módulos para exportar un
valor único en lugar de un objeto de interfaz.

Al definir `require`, `exportaciones` y `modulo` como ((parametros)) para
la función de envoltura generada (y pasando los valores apropiados
al llamarla), el cargador se asegura de que estas vinculaciones esten
disponibles en el ((alcance)) del módulo.

{{index resolution, "relative path"}}

La forma en que el string dado a `require` se traduce a un nombre de archivo
real o dirección web difiere en diferentes sistemas. Cuando comienza con
`"./"` o `"../"`, generalmente se interpreta como relativo al
nombre del archivo actual. Entonces `"./format-date"` sería el archivo
llamado `format-date.js` en el mismo directorio.

Cuando el nombre no es relativo, Node.js buscará por un paquete instalado
con ese nombre. En el código de ejemplo de este capítulo,
interpretaremos esos nombres como referencias a paquetes de NPM.
Entraremos en más detalles sobre cómo instalar y usar los módulos de NPM en
el [Capítulo 20](node).

{{id modules_ini}}

{{index "ini package"}}

Ahora, en lugar de escribir nuestro propio analizador de archivos INI,
podemos usar uno de ((NPM)):

```
const {parse} = require("ini");

console.log(parse("x = 10\ny = 20"));
// → {x: "10", y: "20"}
```

## Módulos ECMAScript

Los ((módulos CommonJS)) funcionan bastante bien y, en combinación con NPM,
han permitido que la comunidad de JavaScript comience a compartir
código en una gran escala.

{{index "exports object", linter}}

Pero siguen siendo un poco de un ((truco)) con cinta adhesiva. La ((notación)) es
ligeramente incomoda—las cosas que agregas a `exports` no están disponibles en
el ((alcance)) local, por ejemplo. Y ya que `require` es una llamada de función
normal tomando cualquier tipo de argumento, no solo un string literal,
puede ser difícil de determinar las dependencias de un módulo sin
correr su código primero.

{{index "import keyword", dependency, "ES modules"}}

{{id es}}

Esta es la razón por la cual el estándar de JavaScript introdujo su propio,
sistema de módulos diferente a partir de 2015. Por lo general es llamado
_((módulos ES))_, donde _ES_ significa ((ECMAScript)). Los principales
conceptos de dependencias e interfaces siguen siendo los mismos,
pero los detalles difieren. Por un lado,
la notación está ahora integrada en el lenguaje. En lugar de llamar a una
función para acceder a una dependencia, utilizas una palabra clave
`import` ("importar") especial.

```
import ordinal from "ordinal";
import {days, months} from "date-names";

export function formatDate(date, format) { /* ... */ }
```

{{index "export keyword", "formatDate module"}}

Similarmente, la palabra clave `export` se usa para exportar cosas. Puede
aparecer delante de una función, clase o definición de vinculación (`let`,
`const`, o `var`).

La interfaz de un módulo ES no es un valor único, sino un conjunto de
((vinculaciones)) con nombres. El módulo anterior vincula `formatDate`
a una función. Cuando importas desde otro módulo, importas la _vinculación_,
no el valor, lo que significa que un módulo exportado puede cambiar el
valor de la vinculación en cualquier momento, y que los módulos que la importen
verán su nuevo valor.

{{index "default export"}}

Cuando hay una vinculación llamada `default`, esta se trata como el
principal valor del módulo exportado. Si importas un módulo como `ordinal` en
el ejemplo, sin llaves alrededor del nombre de la vinculación, obtienes su
vinculación `default`. Dichos módulos aún pueden exportar otras vinculaciones
bajo diferentes nombres ademas de su exportación por `default`.

Para crear una exportación por default, escribe `export default` antes de una
expresión, una declaración de función o una declaración de clase.

```
export default ["Invierno", "Primavera", "Verano", "Otoño"];
```

Es posible renombrar la vinculación importada usando la palabra `as` ("como").

```
import {days as nombresDias} from "date-names";

console.log(nombresDias.length);
// → 7
```

Al momento de escribir esto, la comunidad de JavaScript está en proceso de
adoptar este estilo de módulos. Pero ha sido un proceso lento. Tomó
algunos años, después de que se haya especificado el formato paraq que los
navegadores y Node.js comenzaran a soportarlo. Y a pesar de que lo soportan
mayormente ahora, este soporte todavía tiene problemas, y la discusión sobre
cómo dichos módulos deberían distribuirse a través de ((NPM)) todavía está en
curso.

Muchos proyectos se escriben usando módulos ES y luego se convierten
automáticamente a algún otro formato cuando son publicados. Estamos en
período de transición en el que se utilizan dos sistemas de módulos diferentes
uno al lado del otro, y es útil poder leer y escribir código en
cualquiera de ellos.

## Construyendo y empaquetando

{{index compilation, "type checking"}}

De hecho, muchos proyectos de JavaScript ni siquiera están, técnicamente,
escritos en JavaScript. Hay extensiones, como el ((dialecto)) de comprobación
de tipos mencionado en el [Capítulo 7](error#typing), que son ampliamente
usados. Las personas también suelen comenzar a usar extensiones planificadas
para el lenguaje mucho antes de que estas hayan sido agregadas a las plataformas
que realmente corren JavaScript.

Para que esto sea posible, ellos _compilan_ su código, traduciéndolo del
dialecto de JavaScript que eligieron a JavaScript simple y antiguo—o incluso a
una versión anterior de JavaScript, para que ((navegadores)) antiguos puedan
ejecutarlo.

{{index latency, performance}}

Incluir un programa modular que consiste de 200 archivos diferentes
en una ((página web)) produce sus propios problemas. Si buscar un
solo ((archivo)) sobre la ((red)) tarda 50 milisegundos, cargar
todo el programa tardaria 10 segundos, o tal vez la mitad si puedes
cargar varios archivos simultáneamente. Eso es mucho tiempo perdido.
Ya que buscar un solo archivo grande tiende a ser más rápido que buscar muchos
archivos pequeños, los programadores web han comenzado a usar herramientas que
convierten sus programas (los cuales cuidadosamente estan dividos en módulos)
de nuevo en un único archivo grande antes de publicarlo en la Web. Tales
herramientas son llamado _((empaquetadores))_.

{{index "file size"}}

Y podemos ir más allá. Además de la cantidad de archivos, el _tamaño_ de
los archivos también determina qué tan rápido se pueden transferir a través de la
red. Por lo tanto, la comunidad de JavaScript ha inventado _((minificadores))_.
Estas son herramientas que toman un programa de JavaScript y lo hacen más pequeño
al eliminar automáticamente los comentarios y espacios en blanco, cambia el nombre
de las vinculaciones, y reemplaza piezas de código con código equivalente
que ocupa menos espacio.

{{index pipeline, tool}}

Por lo tanto, no es raro que el código que encuentres en un paquete de NPM o
que se ejecute en una página web haya pasado por _multiples_ etapas de
transformación: conversión de JavaScript moderno a JavaScript histórico,
del formato de módulos ES a CommonJS, empaquetado y minificado.
No vamos a entrar en los detalles de estas herramientas en este libro, ya que
tienden a ser aburridos y cambian rápidamente. Solo ten en cuenta que
el código JavaScript que ejecutas a menudo no es el código tal y como fue
escrito.

## Diseño de módulos

{{index [module, design], interface, "code structure"}}

La estructuración de programas es uno de los aspectos más sutiles de la
programación. Cualquier pieza de funcionalidad no trivial se puede modelar de
varias maneras.

Un buen diseño de programa es subjetivo—hay ventajas/desventajas involucradas, y
cuestiones de gusto. La mejor manera de aprender el valor de una buena
estructura de diseño es leer o trabajar en muchos programas y notar lo que
funciona y lo qué no. No asumas que un desastroso doloroso es
"solo la forma en que las cosas son ". Puedes mejorar la estructura de casi
todo al ponerle mas pensamiento.

Un aspecto del diseño de módulos es la facilidad de uso. Si estás diseñando
algo que está destinado a ser utilizado por varias personas—o incluso por
ti mismo, en tres meses cuando ya no recuerdes los detalles de
lo que hiciste—es útil si tu ((interfaz)) es simple y predicible.

{{index "ini package", JSON}}

Eso puede significar seguir convenciones existentes. Un buen ejemplo es el
paquete `ini`. Este módulo imita el objeto estándar `JSON` al
proporcionar las funciones `parse` y `stringify` (para escribir un archivo INI),
y, como `JSON`, convierte entre strings y objetos simples. Entonces
la interfaz es pequeña y familiar, y después de haber trabajado con ella una
vez, es probable que recuerdes cómo usarla.

{{index "side effect", "hard disk", composability}}

Incluso si no hay una función estándar o un paquete ampliamente utilizado para
imitar, puedes mantener tus módulos predecibles mediante el uso de
estructuras de datos simples y haciendo una cosa única y enfocada. Muchos de
los módulos de análisis de archivos INI en NPM proporcionan una función que
lee directamente tal archivo del disco duro y lo analiza, por ejemplo. Esto
hace que sea imposible de usar tales módulos en el navegador, donde no tenemos
acceso directo al sistema de archivos, y agrega una complejidad que habría sido
mejor abordada al _componer_ el módulo con alguna función de lectura de
archivos.

{{index "pure function"}}

Lo que apunta a otro aspecto útil del diseño de módulos—la facilidad con la
qué algo se puede componer con otro código. Módulos enfocados que
que computan valores son aplicables en una gama más amplia de programas
que módulos mas grandes que realizan acciones complicadas con efectos
secundarios. Un lector de archivos INI que insista en leer el archivo desde el
disco es inútil en un escenario donde el contenido del archivo provenga de
alguna otra fuente.

{{index "object-oriented programming"}}

Relacionadamente, los objetos con estado son a veces útiles e incluso necesarios,
pero si se puede hacer algo con una función, usa una función. Varios
de los lectores de archivos INI en NPM proporcionan un estilo de interfaz que
requiere que primero debes crear un objeto, luego cargar el archivo en tu objeto,
y finalmente usar métodos especializados para obtener los resultados. Este tipo
de cosas es común en la tradición orientada a objetos, y es terrible.
En lugar de hacer una sola llamada de función y seguir adelante,
tienes que realizar el ritual de mover tu objeto a través de diversos
estados. Y ya que los datos ahora están envueltos en un objeto de
tipo especializado, todo el código que interactúa con él tiene que saber
sobre ese tipo, creando interdependencias innecesarias.

A menudo no se puede evitar la definición de nuevas estructuras de datos—solo
unas pocas básicas son provistos por el estándar de lenguaje, y muchos
tipos de datos tienen que ser más complejos que un ((array)) o un mapa.
Pero cuando el array es suficiente, usa un array.

Un ejemplo de una estructura de datos un poco más compleja es el grafo de el
[Capítulo 7](robot). No hay una sola manera obvia de representar un
((grafo)) en JavaScript. En ese capítulo, usamos un objeto cuya propiedades
contenian arrays de strings—los otros nodos accesibles desde ese
nodo.

Hay varios paquetes de busqueda de rutas diferentes en ((NPM)), pero ninguno
de ellos usa este formato de grafo. Por lo general, estos permiten que los
bordes del grafo tengan un peso, el costo o la distancia asociada a ellos,
lo que no es posible en nuestra representación.

{{index "Dijkstra, Edsger", pathfinding, "Dijkstra's algorithm", "dijkstrajs package"}}

Por ejemplo, está el paquete `dijkstrajs`. Un enfoque bien conocido
par la busqueda de rutas, bastante similar a nuestra función `encontrarRuta`,
se llama el _algoritmo de Dijkstra_, después de Edsger Dijkstra, quien fue
el primero que lo escribió. El sufijo `js` a menudo se agrega a los nombres de
los paquetes para indicar el hecho de que están escritos en JavaScript.
Este paquete `dijkstrajs` utiliza un formato de grafo similar al nuestro,
pero en lugar de arrays, utiliza objetos cuyos valores de propiedad son
números—los pesos de los bordes.

Si quisiéramos usar ese paquete, tendríamos que asegurarnos de que nuestro
grafo fue almacenado en el formato que este espera.

```
const {find_path} = require("dijkstrajs");

let grafo = {};
for (let node of Object.keys(roadGraph)) {
  let edges = graph[node] = {};
  for (let dest of roadGraph[node]) {
    edges[dest] = 1;
  }
}

console.log(find_path(grafo, "Oficina de Correos", "Cabaña"));
// → ["Oficina de Correos", "Casa de Alice", "Cabaña"]
```

Esto puede ser una barrera para la composición—cuando varios paquetes están
usando diferentes estructuras de datos para describir cosas similares,
combinarlos es difícil. Por lo tanto, si deseas diseñar para la compibilidad,
averigua qué ((estructura de datos)) están usando otras personas y, cuando
sea posible, sigue su ejemplo.

## Resumen

Los módulos proporcionan de estructura a programas más grandes al separar el
código en piezas con interfaces y dependencias claras. La interfaz es
la parte del módulo que es visible desde otros módulos, y
las dependencias son los otros módulos este que utiliza.

Debido a que históricamente JavaScript no proporcionó un sistema de módulos,
el sistema CommonJS fue construido encima. Entonces, en algún momento,
_consiguio_ un sistema incorporado, que ahora coexiste incomodamente
con el sistema CommonJS.

Un paquete es una porción de código que se puede distribuir por sí misma. NPM
es un repositorio de paquetes de JavaScript. Puedes descargar todo tipo de
paquetes útiles (e inútiles) de él.

## Ejercicios

### Un robot modular

{{index "modular robot (exercise)", module, robot, NPM}}

{{id modular_robot}}

Estas son las vinculaciones que el proyecto del [Capítulo 7](robot) crea:

```{lang: "text/plain"}
caminos
construirGrafo
grafoCamino
EstadoPueblo
correrRobot
eleccionAleatoria
robotAleatorio
rutaCorreo
robotRuta
encontrarRuta
robotOrientadoAMetas
```

Si tuvieras que escribir ese proyecto como un programa modular, qué módulos
crearías? Qué módulo dependería de qué otro módulo, y
cómo se verían sus interfaces?

Qué piezas es probable que estén disponibles pre-escritas en NPM?
Preferirias usar un paquete de NPM o escribirlas tu mismo?

{{hint

{{index "modular robot (exercise)"}}

Aqui esta lo que habría hecho (pero, una vez más, no hay una sola forma
_correcta_ de diseñar un módulo dado):

{{index "dijkstrajs package"}}

El código usado para construir el camino de grafo vive en el módulo `grafo`.
Ya que prefiero usar `dijkstrajs` de NPM en lugar de nuestro propio código de busqueda
de rutas, haremos que este construya el tipo de datos de grafos que `dijkstajs`
espera. Este módulo exporta una sola función, `construirGrafo`. Haria que
`construirGrafo` acepte un array de arrays de dos elementos, en lugar de
strings que contengan guiones, para hacer que el módulo sea menos
dependiente del formato de entrada.

El módulo `caminos` contiene los datos en bruto del camino (el array `caminos`)
y la vinculación `grafoCamino`. Este módulo depende de `./grafo` y exporta
el grafo del camino.

{{index "random-item package"}}

La clase `EstadoPueblo` vive en el módulo `estado`. Depende del
módulo `./caminos`, porque necesita poder verificar que un
camino dado existe. También necesita `eleccionAleatoria`. Dado que eso es una
función de tres líneas, podríamos simplemente ponerla en el módulo `estado`
como una función auxiliar interna. Pero `robotAleatorio` también la necesita.
Entonces tendriamos que duplicarla o ponerla en su propio módulo.
Dado que esta función existe en NPM en el paquete `random-item`, una buena
solución es hacer que ambos módulos dependan de el. Podemos agregar
la función `correrRobot` a este módulo también, ya que es pequeña y
estrechamente relacionada con la gestión de estado. El módulo exporta tanto
la clase `EstadoPueblo` como la función `correrRobot`.

Finalmente, los robots, junto con los valores de los que dependen, como
`mailRoute`, podrían ir en un módulo `robots-ejemplo`, que depende
de `./caminos` y exporta las funciones de robot. Para que sea posible
que el `robotOrientadoAMetas` haga busqueda de rutas, este módulo también
depende de `dijkstrajs`.

Al descargar algo de trabajo a los módulos de ((NPM)), el código se volvió un
poco mas pequeño. Cada módulo individual hace algo bastante simple, y puede
ser leído por sí mismo. La división del código en módulos también sugiere
a menudo otras mejoras para el diseño del programa. En este caso, parece un
poco extraño que `EstadoPueblo` y los robots dependan de un grafo de caminos.
Podría ser una mejor idea hacer del grafo un argumento para
el constructor del estado y hacer que los robots lo lean del objeto
estado—esto reduce las dependencias (lo que siempre es bueno) y hace
posible ejecutar simulaciones en diferentes mapas (lo cual es aún mejor).

Es una buena idea usar módulos de NPM para cosas que podríamos haber
escrito nosotros mismos? En principio, sí—para cosas no triviales como la
función de busqueda de rutas es probable que cometas errores y pierdas el tiempo
escribiendola tú mismo. Para pequeñas funciones como `eleccionAleatoria`,
escribirla por ti mismo es lo suficiente fácil. Pero agregarlas
donde las necesites tiende a desordenar tus módulos.

Sin embargo, tampoco debes subestimar el trabajo involucrado en
_encontrar_ un paquete apropiado de NPM. E incluso si encuentras uno, este
podría no funcionar bien o faltarle alguna característica que necesitas.
Ademas de eso, depender de los paquetes de NPM, significa  que debes asegurarte
de que están instalados, tienes que distribuirlos con tu
programa, y podrías tener que actualizarlos periódicamente.

Entonces, de nuevo, esta es una solución con compromisos, y tu puedes decidir de
una u otra manera dependiendo sobre cuánto te ayuden los paquetes.

hint}}

### Módulo de Caminos

{{index "roads module (exercise)"}}

Escribe un ((módulo CommonJS)), basado en el ejemplo del [Capítulo 7](robot),
que contenga el array de caminos y exporte la estructura de datos
grafo que los representa como `grafoCamino`. Debería depender de un
modulo `./grafo`, que exporta una función `construirGrafo` que se usa
para construir el grafo. Esta función espera un array de arrays de
dos elementos (los puntos de inicio y final de los caminos).

{{if interactive

```{test: no}
// Añadir dependencias y exportaciones

const caminos = [
  "Casa de Alicia-Casa de Bob",        "Casa de Alicia-Cabaña",
  "Casa de Alicia-Oficina de Correos", "Casa de Bob-Ayuntamiento",
  "Casa de Daria-Casa de Ernie",       "Casa de Daria-Ayuntamiento",
  "Casa de Ernie-Casa de Grete",       "Casa de Grete-Granja",
  "Casa de Grete-Tienda",              "Mercado-Granja",
  "Mercado-Oficina de Correos",        "Mercado-Tienda",
  "Mercado-Ayuntamiento",              "Tienda-Ayuntamiento"
];
```

if}}

{{hint

{{index "roads module (exercise)", "destructuring binding", "exports object"}}

Como este es un ((módulo CommonJS)), debes usar `require` para
importar el módulo grafo. Eso fue descrito como exportar una
función `construirGrafo`, que puedes sacar de su objeto de interfaz
con una declaración `const` de desestructuración.

Para exportar `grafoCamino`, agrega una propiedad al objeto `exports`.
Ya que `construirGrafo` toma una estructura de datos que no empareja
precisamente `caminos`, la división de los strings de los caminis
debe ocurrir en tu módulo.

hint}}

### Dependencias circulares

{{index dependency, "circular dependency", "require function"}}

Una dependencia circular es una situación en donde el módulo A depende de B, y
B también, directa o indirectamente, depende de A. Muchos sistemas de módulos
simplemente prohíbne esto porque cualquiera que sea el orden que elijas
para cargar tales módulos, no puedes asegurarse de que las dependencias de
cada módulo han sido cargadas antes de que se ejecuten.

Los ((modulos CommonJS)) permiten una forma limitada de dependencias cíclicas.
Siempre que los módulos no reemplacen a su objeto `exports` predeterminado, y
no accedan a la interfaz de las demás hasta que terminen de cargar,
las dependencias cíclicas están bien.

La función `require` dada [anteriormente en este capítulo](modulos#require)
es compatible con este tipo de ciclo de dependencias. Puedes
ver cómo maneja los ciclos? Qué iría mal cuando un módulo en un
ciclo _reemplace_ su objeto `exports` por defecto?

{{hint

{{index overriding, "circular dependency", "exports object"}}

El truco es que `require` agrega módulos a su caché _antes_ de
comenzar a cargar el módulo. De esa forma, si se realiza una llamada `require`
mientras está ejecutando el intento de cargarlo, ya es conocido y la
interfaz actual sera retornada, en lugar de comenzar a cargar el módulo
una vez más (lo que eventualmente desbordaría la pila).

Si un módulo sobrescribe su valor `module.exports`, cualquier otro módulo
que haya recibido su valor de interfaz antes de que termine de cargarse
ha conseguido el objeto de interfaz predeterminado (que es probable que este
vacío), en lugar del valor de interfaz previsto.

hint}}
