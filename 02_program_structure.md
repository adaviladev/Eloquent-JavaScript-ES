# Estructura de Programa

{{quote {author: "_why", title: "Why's (Poignant) Guide to Ruby", chapter: true}

Y mi corazón brilla de un color rojo brillante bajo mi piel transparente y
translúcida, y tienen que administrarme 10cc de JavaScript para conseguir que
regrese. (respondo bien a las toxinas en la sangre.) ¡Hombre, esa cosa es
increíble!

quote}}

{{index why, "Poignant Guide"}}

{{figure {url: "img/chapter_picture_2.jpg", alt: "Foto de tentáculos sosteniendo objetos", chapter: framed}}}

En este capítulo, comenzaremos a hacer cosas que realmente se pueden llamar
_programación_. Expandiremos nuestro dominio del lenguaje JavaScript
más allá de los sustantivos y fragmentos de oraciones que hemos visto hasta
ahora, al punto donde podemos expresar prosa significativa.

## Expresiones y declaraciones

{{index grammar, syntax, [code, "structure of"], grammar, [JavaScript, syntax]}}

En el [Capítulo 1](valores), creamos valores y les aplicamos operadores a ellos
para obtener nuevos valores. Crear valores de esta manera es la sustancia
principal de cualquier programa en JavaScript. Pero esa sustancia tiene que
enmarcarse en una estructura más grande para poder ser útil. Así que eso es
lo que veremos a continuación.

{{index "literal expression"}}

Un fragmento de código que produce un valor se llama una
_((expresión))_. Cada valor que se escribe literalmente (como `22`
o `"psicoanálisis"`) es una expresión. Una expresión entre
((paréntesis)) también es una expresión, como lo es un ((operador binario))
aplicado a dos expresiones o un ((operador unario)) aplicado a una sola.

{{index [nesting, "of expressions"], "human language"}}

Esto demuestra parte de la belleza de una interfaz basada en un lenguaje.
Las expresiones pueden contener otras expresiones de una manera muy similar
a como las sub-oraciones en los lenguajes humanos están anidadas, una
sub-oración puede contener sus propias sub-oraciones, y así sucesivamente.
Esto nos permite construir expresiones que describen cálculos arbitrariamente
complejos.

{{index statement, semicolon, program}}

Si una expresión corresponde al fragmento de una oración, una _declaración_
en JavaScript corresponde a una oración completa. Un programa es una lista de
declaraciones.

{{index syntax}}

El tipo más simple de declaración es una expresión con un punto y coma después
ella. Esto es un programa:

```
1;
!false;
```

Sin embargo, es un programa inútil. Una ((expresión)) puede estar feliz solo con
producir un valor, que luego pueda ser utilizado por el código circundante. Una
((declaración)) es independiente por si misma, por lo que equivale a
algo solo si afecta al mundo. Podría mostrar algo en la pantalla, eso
cuenta como cambiar el mundo, o podría cambiar el estado interno de
la máquina en una manera que afectará a las declaraciones que vengan después de
ella. Estos cambios se llaman _((efecto secundario))s_. Las declaraciones en el
ejemplo anterior solo producen los valores `1` y `true` y luego
inmediatamente los tira a la basura. Esto no deja ninguna huella en el mundo.
Cuando ejecutes este programa, nada observable ocurre.

{{index "programming style", "automatic semicolon insertion", semicolon}}

En algunos casos, JavaScript te permite omitir el punto y coma al final
de una declaración. En otros casos, tiene que estar allí, o la próxima
((línea)) serán tratada como parte de la misma declaración. Las reglas para
saber cuando se puede omitir con seguridad son algo complejas y propensas a
errores. Asi que en este libro, cada declaración que necesite un punto y coma
siempre tendra uno. Te recomiendo que hagas lo mismo, al menos hasta que hayas
aprendido más sobre las sutilezas de los puntos y comas que puedan ser
omitidos.

## Vinculaciones

{{indexsee variable, binding}}
{{index syntax, [binding, definition], "side effect", memory}}

Cómo mantiene un programa un ((estado)) interno? Cómo recuerda
cosas? Hasta ahora hemos visto cómo producir nuevos valores a partir de
valores anteriores, pero esto no cambia los valores anteriores, y el nuevo valor
tiene que ser usado inmediatamente o se disipará nuevamente. Para atrapar y
mantener valores, JavaScript proporciona una cosa llamada _vinculación_, o
_variable_:

```
let atrapado = 5 * 5;
```

{{index "let keyword"}}

Ese es un segundo tipo de ((declaración)). La palabra especial
(_((palabra clave))_) `let` indica que esta oración va a definir
una vinculación. Le sigue el nombre de la vinculación y, si queremos
darle un valor inmediatamente, un operador `=` y una expresión.

La declaración anterior crea una vinculación llamada `atrapado` y la usa
para capturar el número que se produce al multiplicar 5 por 5.

Después de que una vinculación haya sido definida, su nombre puede usarse como
una ((expresión)). El valor de tal expresión es el valor que la
vinculación mantiene actualmente. Aquí hay un ejemplo:

```
let diez = 10;
console.log(diez * diez);
// → 100
```

{{index "= operator", assignment, [binding, assignment]}}

Cuando una vinculación señala a un valor, eso no significa que esté atada a
ese valor para siempre. El operador `=` puede usarse en cualquier momento en
vinculaciones existentes para desconectarlas de su valor actual y hacer que
ellas apuntan a uno nuevo:

```
let humor = "ligero";
console.log(humor);
// → ligero
humor = "oscuro";
console.log(humor);
// → oscuro
```

{{index [binding, "model of"], "tentacle (analogy)"}}

Deberías imaginar a las vinculaciones como tentáculos, en lugar de cajas.
Ellas no _contienen_ valores; ellas los _agarran_, dos vinculaciones pueden
referirse al mismo valor. Un programa solo puede acceder a los valores que
todavía pueda referenciar. Cuando necesitas recordar algo, creces un
tentáculo para aferrarte a él o vuelves a conectar uno de tus tentáculos
existentes a ese algo.

Veamos otro ejemplo. Para recordar la cantidad de dólares que
Luigi aún te debe, creas una vinculación. Y luego, cuando él te
pague de vuelta $35, le das a esta vinculación un nuevo valor:

```
let deudaLuigi = 140;
deudaLuigi = deudaLuigi - 35;
console.log(deudaLuigi);
// → 105
```

{{index undefined}}

Cuando defines una vinculación sin darle un valor, el tentáculo no tiene
nada que agarrar, por lo que termina en solo aire. Si pides el valor de
una vinculación vacía, obtendrás el valor `undefined`.

{{index "let keyword"}}

Una sola declaración `let` puede definir múltiples vinculaciones.
Las definiciones deben estar separadas por comas.

```
let uno = 1, dos = 2;
console.log(uno + dos);
// → 3
```

Las palabras `var` y `const` también pueden ser usadas para crear
vinculaciones, en una manera similar a `let`.

```
var nombre = "Ayda";
const saludo = "Hola ";
console.log(saludo + nombre);
// → Hola Ayda
```

{{index "var keyword"}}

La primera, `var` (abreviatura de "variable"), es la forma en la que se
declaraban las vinculaciones en JavaScript previo al 2015. Volveremos a la forma
precisa en que difiere de `let` en el [próximo capítulo](funciones). Por ahora,
recuerda que generalmente hace lo mismo, pero raramente la usaremos
en este libro porque tiene algunas propiedades confusas.

{{index "const keyword", naming}}

La palabra `const` representa una _((constante))_. Define una vinculación
constante, que apunta al mismo valor por el tiempo que viva. Esto
es útil para vinculaciones que le dan un nombre a un valor para que
fácilmente puedas consultarlo más adelante.

## Nombres vinculantes

{{index "underscore character", "dollar sign", [binding, naming]}}

Los nombres de las vinculaciones pueden ser cualquier palabra. Los dígitos pueden
ser parte de los nombres de las vinculaciones pueden, `catch22` es un nombre
válido, por ejemplo, pero el nombre no debe comenzar con un dígito.
El nombre de una vinculación puede incluir signos de dólar (`$`) o
caracteres de subrayado (`_`), pero no otros signos de puntuación o
caracteres especiales.

{{index syntax, "implements (reserved word)", "interface (reserved word)", "package (reserved word)", "private (reserved word)", "protected (reserved word)", "public (reserved word)", "static (reserved word)", "void operator", "yield (reserved word)", "enum (reserved word)", "reserved word", [binding, naming]}}

Las palabras con un significado especial, como `let`, son _((palabras clave))s_, y
no pueden usarse como nombres vinculantes. También hay una cantidad de
palabras que están "reservadas para su uso" en ((futuras)) versiones de
JavaScript, que tampoco pueden ser usadas como nombres vinculantes.
La lista completa de palabras clave y palabras reservadas es bastante larga:

```{lang: "text/plain"}
break case catch class const continue debugger default
delete do else enum export extends false finally for
function if implements import interface in instanceof let
new package private protected public return static super
switch this throw true try typeof var void while with yield
```

No te preocupes por memorizarlas. Cuando crear una vinculación produzca
un ((error de sintaxis)) inesperado, observa si estas tratando de definir una
palabra reservada.

## El entorno

{{index "standard environment"}}

La colección de vinculaciones y sus valores que existen en un momento dado
se llama _((entorno))_. Cuando se inicia un programa, est
entorno no está vacío. Siempre contiene vinculaciones que son parte del
((estándar)) del lenguaje, y la mayoría de las veces, también tiene vinculaciones
que proporcionan formas de interactuar con el sistema circundante. Por
ejemplo, en el ((navegador)), hay funciones para interactuar con el
sitio web actualmente cargado y para leer entradas del ((mouse)) y ((teclado)).

## Funciones

{{indexsee "application (of functions)", "function application"}}
{{indexsee "invoking (of functions)", "function application"}}
{{indexsee "calling (of functions)", "function application"}}
{{index output, function, [function, application]}}

Muchos de los valores proporcionados por el entorno predeterminado tienen el
tipo _((función))_. Una función es una pieza de programa envuelta en un valor.
Dichos valores pueden ser _aplicados_ para ejecutar el programa envuelto. Por
ejemplo, en un entorno ((navegador)), la vinculación `prompt` sostiene una
función que muestra un pequeño ((cuadro de diálogo)) preguntando por entrada
del usuario. Esta se usa así:

```
prompt("Introducir contraseña");
```

{{figure {url: "img/prompt.png", alt: "A prompt dialog", width: "8cm"}}}

{{index parameter, [function, application]}}

Ejecutar una función también se conoce como _invocarla_, _llamarla_, o
_aplicarla_. Puedes llamar a una función poniendo ((paréntesis)) después de una
expresión que produzca un valor de función. Usualmente usarás directamente
el nombre de la vinculación que contenga la función. Los valores entre
los paréntesis se dan al programa dentro de la función. En el
ejemplo, la función `prompt` usa el string que le damos como el
texto a mostrar en el cuadro de diálogo. Los valores dados a las funciones
se llaman _((argumento))s_. Diferentes funciones pueden necesitar un número
diferente o diferentes tipos de argumentos

La función `prompt` no se usa mucho en la programación web moderna,
sobre todo porque no tienes control sobre la forma en como se ve la caja de
diálogo resultante, pero puede ser útil en programas de juguete y experimentos.

## La función console.log

{{index "JavaScript console", "developer tools", "Node.js", "console.log", output}}

En los ejemplos, utilicé `console.log` para dar salida a los valores.
La mayoría de los sistemas de JavaScript (incluidos todos los ((navegadore))s
web modernos y Node.js) proporcionan una función `console.log` que escribe
sus argumentos en _algun_ dispositivo de salida de texto. En los navegadores,
esta salida aterriza en la ((consola de JavaScript)). Esta parte de la interfaz
del navegador está oculta por defecto, pero la mayoría de los navegadores
la abren cuando presionas F12 o, en Mac, Command-Option-I. Si eso no funciona,
busca en los menús un elemento llamado "herramientas de desarrollador" o
algo similar.

{{if interactive

Al ejecutar los ejemplos (o tu propio código) en las páginas de este
libro, la salida de `console.log` se mostrará después del ejemplo, en lugar de
en la consola de JavaScript del navegador.

```
let x = 30;
console.log("el valor de x es", x);
// → el valor de x es 30
```

if}}

{{index object}}

Aunque los nombres de las vinculaciones no puedan contener
((carácteres de punto))s, `console.log` tiene uno. Esto es porque `console.log`
no es un vinculación simple. En realidad, es una expresión que obtiene la
((propiedad)) `log` del valor mantenido por la vinculación `console`.
Averiguaremos qué significa esto exactamente en el
[Capítulo 4](datos#propiedades).

{{id return_values}}
## Valores de retorno

{{index [comparison, "of numbers"], "return value", "Math.max function", maximum}}

Mostrar un cuadro de diálogo o escribir texto en la pantalla es un _((efecto
secundario))_. Muchas funciones son útiles debido a los efectos secundarios
que ellas producen. Las funciones también pueden producir valores, en cuyo caso
no necesitan tener un efecto secundario para ser útil. Por ejemplo, la
función `Math.max` toma cualquier cantidad de argumentos numéricos y devuelve
el mayor de ellos.

```
console.log(Math.max(2, 4));
// → 4
```

{{index [function, application], minimum, "Math.min function"}}

Cuando una función produce un valor, se dice que _retorna_ ese valor.
Todo lo que produce un valor es una ((expresión)) en JavaScript,
lo que significa que las llamadas a funciones se pueden usar dentro de
expresiones más grandes. aquí una llamada a `Math.min`, que es lo opuesto
a `Math.max`, se usa como parte de una expresión de adición:

```
console.log(Math.min(2, 4) + 100);
// → 102
```

El [próximo capítulo](funciones) explica cómo escribir tus propias
funciones.

## Flujo de control

{{index "execution order", program, "control flow"}}

Cuando tu programa contiene más de una ((declaración)), las declaraciones
se ejecutan como si fueran una historia, de arriba a abajo. Este programa de
ejemplo tiene dos declaraciones. La primera le pide al usuario un número,
y la segunda, que se ejecuta después de la primera, muestra el
((cuadrado)) de ese número.

```
let elNumero = Number(prompt("Elige un numero"));
console.log("Tu número es la raiz cuadrada de " +
            elNumero * elNumero);
```

{{index [number, "conversion to"], "type coercion", "Number function", "String function", "Boolean function", [Boolean, "conversion to"]}}

La función `Número` convierte un valor a un número. Necesitamos esa
conversión porque el resultado de `prompt` es un valor de string, y nosotros
queremos un numero. Hay funciones similares llamadas `String` y
`Boolean` que convierten valores a esos tipos.

Aquí está la representación esquemática (bastante trivial) de un flujo de
control en línea recta:

{{figure {url: "img/controlflow-straight.svg", alt: "Trivial control flow", width: "4cm"}}}

## Ejecución condicional

{{index Boolean, "control flow"}}

No todos los programas son caminos rectos. Podemos, por ejemplo, querer
crear un camino de ramificación, donde el programa toma la rama adecuada
basadandose en la situación en cuestión. Esto se llama _((ejecución
condicional))_.

{{figure {url: "img/controlflow-if.svg", alt: "Conditional control flow",width: "4cm"}}}

{{index syntax, "Number function", "if keyword"}}

La ejecución condicional se crea con la palabra clave `if` en JavaScript.
En el caso simple, queremos que se ejecute algún código si, y solo si,
una cierta condición se cumple. Podríamos, por ejemplo, solo querer mostrar el
cuadrado de la entrada si la entrada es realmente un número.

```{test: wrap}
let elNumero = Number(prompt("Elige un numero"));
if (!Number.isNaN(elNumero)) {
  console.log("Tu número es la raiz cuadrada de " +
              elNumero * elNumero);
}
```

Con esta modificación, si ingresas la palabra "loro", no se mostrara
ninguna salida.

La palabra clave `if` ejecuta u omite una declaración dependiendo del valor
de una expresión booleana. La expresión decisiva se escribe después de la
palabra clave, entre ((paréntesis)), seguida de la declaración a
ejecutar.

{{index "Number.isNaN function"}}

La función `Number.isNaN` es una función estándar de JavaScript que
retorna `true` solo si el argumento que se le da es `NaN`. Resulta que
la función `Number` devuelve `NaN` cuando le pasas un string que
no representa un número válido. Por lo tanto, la condición se traduce a
"a menos que `elNumero` no sea un número, haz esto".

{{index grouping, "{} (block)"}}

La declaración debajo del `if` está envuelta en ((llaves)) (`{`y
`}`) en este ejemplo. Estos pueden usarse para agrupar cualquier cantidad de
declaraciones en una sola declaración, llamada un _((bloque))_. Podrías
también haberlas omitido en este caso, ya que solo tienes una sola
declaración, pero para evitar tener que pensar si se necesitan
o no, la mayoría de los programadores en JavaScript las usan en cada una
de sus declaraciones envueltas como esta. Seguiremos esta convención
en la mayoria de este libro, a excepción de la ocasional declaración
de una sola linea.

```
if (1 + 1 == 2) console.log("Es verdad");
// → Es verdad
```

{{index "else keyword"}}

A menudo no solo tendrás código que se ejecuta cuando una condición es
verdadera, pero también código que maneja el otro caso. Esta ruta alternativa
está representado por la segunda flecha en el diagrama. La palabra clave `else`
se puede usar, junto con `if`, para crear dos caminos de ejecución
alternativos, de una manera separada.

```{test: wrap}
let elNumero = Number(prompt("Elige un numero"));
if (!Number.isNaN(elNumero)) {
  console.log("Tu número es la raiz cuadrada de " +
              elNumero * elNumero);
} else {
  console.log("Ey. Por qué no me diste un número?");
}
```

{{index ["if keyword", chaining]}}

Si tenemos más de dos rutas a elegir, múltiples pares de `if`/`else`
se pueden "encadenar". Aquí hay un ejemplo:

```
let numero = Number(prompt("Elige un numero"));

if (numero < 10) {
  console.log("Pequeño");
} else if (numero < 100) {
  console.log("Mediano");
} else {
  console.log("Grande");
}
```

El programa primero comprobará si `numero` es menor que 10. Si lo es,
eligira esa rama, mostrara `"Pequeño"`, y está listo. Si no es así,
toma la rama `else`, que a su vez contiene un segundo `if`. Si
la segunda condición (`< 100`) es verdadera, eso significa que el número está
entre 10 y 100, y `"Mediano"` se muestra. Si no es así, la segunda y última
la rama `else` es elegida.

El esquema de este programa se ve así:

{{figure {url: "img/controlflow-nested-if.svg", alt: "Nested if control flow", width: "4cm"}}}

{{id loops}}
## Ciclos while y do

Considera un programa que muestra todos los ((números pare))s de 0 a 12. Una
forma de escribir esto es la siguiente:

```
console.log(0);
console.log(2);
console.log(4);
console.log(6);
console.log(8);
console.log(10);
console.log(12);
```

{{index "control flow"}}

Eso funciona, pero la idea de escribir un programa es hacer de algo
_menos_ trabajo, no más. Si necesitáramos todos los números pares menores a
1.000, este enfoque sería poco práctico. Lo que necesitamos es una forma de
ejecutar una pieza de código multiples veces. Esta forma de flujo de control
es llamada un _((ciclo))_ (o "loop"):

{{figure {url: "img/controlflow-loop.svg", alt: "Loop control flow",width: "4cm"}}}

{{index syntax, "counter variable"}}

El flujo de control de ciclos nos permite regresar a algún punto del programa
en donde estábamos antes y repetirlo con nuestro estado del programa actual. Si
combinamos esto con una vinculación que cuenta, podemos hacer algo como
esta:

```
let numero = 0;
while (numero <= 12) {
  console.log(numero);
  numero = numero + 2;
}
// → 0
// → 2
//   … etcetera
```

{{index "while loop", Boolean}}

Una ((declaración)) que comienza con la palabra clave `while` crea un ciclo.
La palabra `while` es seguida por una ((expresión)) en ((paréntesis)) y
luego por una declaración, muy similar a `if`. El bucle sigue ingresando a
esta declaración siempre que la expresión produzca un valor que dé `true`
cuando sea convertida a Boolean.

{{index comparison, state}}

La vinculación `numero` demuestra la forma en que una ((vinculación)
puede seguir el progreso de un programa. Cada vez que el ciclo se repite,
`numero` obtiene un valor que es 2 más que su valor anterior. Al comienzo de
cada repetición, se compara con el número 12 para decidir si
el trabajo del programa está terminado.

{{index exponentiation}}

Como un ejemplo que realmente hace algo útil, ahora podemos escribir un
programa que calcula y muestra el valor de 2^10^ (2 a la 10).
Usamos dos vinculaciones: una para realizar un seguimiento de nuestro resultado
y una para contar cuántas veces hemos multiplicado este resultado por 2.
El ciclo prueba si la segunda vinculación ha llegado a 10 todavía y, si no,
actualiza ambas vinculaciones.

```
let resultado = 1;
let contador = 0;
while (contador < 10) {
  resultado = resultado * 2;
  contador = contador + 1;
}
console.log(resultado);
// → 1024
```

El contador también podría haber comenzado en `1` y chequear para `<= 10`,
pero, por razones que serán evidentes en el [Capítulo 4](datos#array_indexing),
es una buena idea ir acostumbrandose a contar desde 0.

{{index "loop body", "do loop", "control flow"}}

Un ciclo `do` es una estructura de control similar a un ciclo `while`.
Difiere solo en un punto: un ciclo `do` siempre ejecuta su cuerpo
al menos una vez, y comienza a chequear si debe detenerse solo después de
esa primera ejecución. Para reflejar esto, la prueba aparece después del cuerpo
del ciclo:

```
let tuNombre;
do {
  tuNombre = prompt("Quien eres?");
} while (!tuNombre);
console.log(tuNombre);
```

{{index [Boolean, "conversion to"], "! operator"}}

Este programa te obligará a ingresar un nombre. Preguntará de nuevo y
de nuevo hasta que obtenga algo que no sea un string vacío. Aplicar
el operador `!` convertirá un valor a tipo Booleano antes de negarlo
y todos los strings, excepto `""` seran convertidas a `true`. Esto significa que
el ciclo continúa dando vueltas hasta que proporciones un nombre no-vacío.

## Indentando Código

{{index "code structure", whitespace, "programming style"}}

En los ejemplos, he estado agregando espacios adelante de declaraciones que
son parte de una declaración más grande. Estos no son necesarios, la computadora
aceptará el programa normalmente sin ellos. De hecho, incluso las nuevas
((líneas)) en los programas son opcionales. Podrías escribir un
programa en una sola línea inmensa si asi quisieras.

El rol de esta ((indentación)) dentro de los ((bloques)) es hacer que
la estructura del código se destaque. En código donde se abren nuevos bloques
dentro de otros bloques, puede ser difícil ver dónde termina un bloque
y donde comienza el otro. Con la indentación apropiada, la forma visual de un
programa corresponde a la forma de los bloques dentro de él. Me gusta
usar dos espacios para cada bloque abierto, pero los gustos varían, algunas
personas usan cuatro espacios, y algunas personas usan
((carácteres de tabulación)). Lo cosa importante es que cada bloque
nuevo agregue la misma cantidad de espacio.

```
if (false != true) {
  console.log("Esto tiene sentido.");
  if (1 < 2) {
    console.log("Ninguna sorpresa alli.");
  }
}
```

La mayoría de los ((editores)) de código [(incluido el de este libro)]{if
interactive} ayudaran indentar automáticamente las nuevas líneas con la
cantidad adecuada.

## Ciclos for

{{index syntax, "while loop", "counter variable"}}

Muchos ciclos siguen el patrón visto en los ejemplos de `while`. Primero una
vinculación "contador" se crea para seguir el progreso del ciclo. Entonces
viene un ciclo `while`, generalmente con una expresión de prueba que verifica si
el contador ha alcanzado su valor final. Al final del cuerpo del ciclo, el
el contador se actualiza para mantener un seguimiento del progreso.

{{index "for loop", loop}}

Debido a que este patrón es muy común, JavaScript y otros lenguajes similares
proporcionan una forma un poco más corta y más completa, el ciclo `for`:

```
for (let numero = 0; numero <= 12; numero = numero + 2) {
  console.log(numero);
}
// → 0
// → 2
//   … etcetera
```

{{index "control flow", state}}

Este programa es exactamente equivalente al ejemplo
[anterior](estructura_de_programa#ciclos) de impresión de números pares.
El único cambio es que todos las ((declaraciónes)) que están relacionadas
con el "estado" del ciclo estan agrupadas después del `for`.

Los ((paréntesis)) después de una palabra clave `for` deben contener dos
((punto y coma))s. La parte antes del primer punto y coma _inicializa_ el
cicloe, generalmente definiendo una ((vinculación)). La segunda parte es la
((expresión)) que _chequea_ si el ciclo debe continuar. La parte final
_actualiza_ el estado del ciclo después de cada iteración. En la mayoría de
los casos, esto es más corto y conciso que un constructo `while`.

{{index exponentiation}}

Este es el código que calcula 2^10^, usando `for` en lugar de `while`:

```{test: wrap}
let resultado = 1;
for (let contador = 0; contador < 10; contador = contador + 1) {
  resultado = resultado * 2;
}
console.log(resultado);
// → 1024
```

## Rompiendo un ciclo

{{index [loop, "termination of"], "break keyword"}}

Hacer que la condición del ciclo produzca `false` no es la única forma en que
el ciclo puede terminar. Hay una declaración especial llamada `break`
("romper") que tiene el efecto de inmediatamente saltar afuera del ciclo
circundante.

Este programa ilustra la declaración `break`. Encuentra el primer número
que es a la vez mayor o igual a 20 y divisible por 7.

```
for (let actual = 20; ; actual = actual + 1) {
  if (actual % 7 == 0) {
    console.log(actual);
    break;
  }
}
// → 21
```

{{index "remainder operator", "% operator"}}

Usar el operador restante (`%`) es una manera fácil de probar si
un número es divisible por otro número. Si lo es, el residuo de
su división es cero.

{{index "for loop"}}

El constructo `for` en el ejemplo no tiene una parte que verifique
cuando finalizar el ciclo. Esto significa que el ciclo nunca se detendrá
a menos que se ejecute la declaración `break` dentro de el.

Si eliminases esa declaración `break` o escribieras accidentalmente
una condición final que siempre produciera `true`, tu programa estaria
atrapado en un _((ciclo infinito))_. Un programa atrapado en un ciclo infinito
nunca terminará de ejecutarse, lo que generalmente es algo malo.

{{if interactive

Si creas un ciclo infinito en alguno de los ejemplos en estas páginas,
generalmente se te preguntará si deseas detener el script después de
algunos segundos. Si eso falla, tendrás que cerrar la pestaña en la que estás
trabajando, o en algunos navegadores, cerrar todo tu navegador, para poder
recuperarse.

if}}

{{index "continue keyword"}}

La palabra clave `continue` ("continuar") es similar a `break`, en que influye
el progreso de un ciclo. Cuando `continue` se encuentre en el cuerpo de un
ciclo, el control salta afuera del cuerpo y continúa con la siguiente
iteración del ciclo.

## Actualizando vinculaciones de manera sucinta

{{index assignment, "+= operator", "-= operator", "/= operator", "*= operator*", state, "side effect"}}

Especialmente cuando realices un ciclo, un programa a menudo necesita
"actualizar" una vinculación para mantener un valor basadandose
en el valor anterior de esa vinculación.

```{test: no}
contador = contador + 1;
```

JavaScript provee de un atajo para esto:

```{test: no}
contador += 1;
```

Atajos similares funcionan para muchos otros operadores, como `resultado *= 2`
para duplicar `resultado` o `contador -= 1` para contar hacia abajo.

Esto nos permite acortar un poco más nuestro ejemplo de conteo.

```
for (let numero = 0; numero <= 12; numero += 2) {
  console.log(numero);
}
```

{{index "++ operator", "-- operator"}}

Para `contador += 1` y `contador -= 1`, hay incluso equivalentes más cortos:
`contador++` y `contador --`.

## Despachar en un valor con switch

{{index syntax, "conditional execution", dispatching, ["if keyword", chaining]}}

No es poco común que el código se vea así:

```{test: no}
if (x == "valor1") accion1();
else if (x == "valor2") accion2();
else if (x == "valor3") accion3();
else accionPorDefault();
```

{{index "colon character", "switch keyword"}}

Existe un constructo llamado `switch` que está destinada a expresar tales
"despachos" de una manera más directa. Desafortunadamente, la sintaxis que
JavaScript usa para esto (que heredó de la línea lenguajes de programación
C/Java) es algo incómoda, una cadena de declaraciones `if` podria llegar a verse
mejor. Aquí hay un ejemplo:

```
switch (prompt("Como esta el clima?")) {
  case "lluvioso":
    console.log("Recuerda salir con un paraguas.");
    break;
  case "soleado":
    console.log("Vistete con poca ropa.");
  case "nublado":
    console.log("Ve afuera.");
    break;
  default:
    console.log("Tipo de clima desconocido!");
    break;
}
```

{{index fallthrough, comparison, "break keyword", "case keyword", "default keyword"}}

Puedes poner cualquier número de etiquetas de `case` dentro del bloque
abierto por `switch`. El programa comenzará a ejecutarse en la etiqueta que
corresponde al valor que se le dio a `switch`, o en `default` si no se
encuentra ningún valor que coincida. Continuará ejecutándose, incluso a
través de otras etiquetas, hasta que llegue a una declaración `break`.
En algunos casos, como en el caso `"soleado"` del ejemplo, esto se puede usar
para compartir algo de código entre casos (recomienda salir para ambos
climas soleado y nublado). Pero ten cuidado, es fácil olvidarse de
`break`, lo que hará que el programa ejecute código que no quieres que sea
ejecutado.

## Capitalización

{{index capitalization, [binding, naming], whitespace}}

Los nombres de vinculaciones no pueden contener espacios, sin embargo,
a menudo es útil usar múltiples palabras para describir claramente lo que
representa la vinculación. Estas son más o menos tus opciones para escribir el
nombre de una vinculación con varias palabras en ella:

```{lang: null}
pequeñatortugaverde
pequeña_tortuga_verde
PequeñaTortugaVerde
pequeñaTortugaVerde
```

{{index "camel case", "programming style", "underscore character"}}

El primer estilo puede ser difícil de leer. Me gusta mucho el aspecto del
estilo con los guiones bajos, aunque ese estilo es algo fastidioso de escribir.
Las funciones ((estándar)) de JavaScript, y la mayoría de los programadores de
JavaScript, siguen el estilo de abajo: capitalizan cada palabra excepto la
primera. No es difícil acostumbrarse a pequeñas cosas así, y programar con
estilos de nombres mixtos pueden ser algo discordante para leer, así que
seguiremos esta ((convención)).

{{index "Number function", constructor}}

En algunos casos, como en la función `Number`, la primera letra de
la vinculación también está en mayúscula. Esto se hizo para marcar esta
función como un constructor. Lo que es un constructor quedará claro en el
[Capítulo 6](objeto#constructores). Por ahora, lo importante es no
ser molestado por esta aparente falta de ((consistencia)).

## Comentarios

{{index readability}}

A menudo, el código en si mismo no transmite toda la información que deseas
que un programa transmita a los lectores humanos, o lo transmite de una
manera tan críptica que la gente quizás no lo entienda. En otras ocasiones,
podrías simplemente querer incluir algunos pensamientos relacionados como
parte de tu programa. Esto es para lo qué son los _((comentario))s_.

{{index "slash character", "line comment"}}

Un comentario es una pieza de texto que es parte de un programa pero que es
completamente ignorado por la computadora. JavaScript tiene dos formas de
escribir comentarios. Para escribir un comentario de una sola línea,
puede usar dos caracteres de barras inclinadas (`//`) y luego el texto del
comentario después.

```{test: no}
let balanceDeCuenta = calcularBalance(cuenta);
// Es un claro del bosque donde canta un río
balanceDeCuenta.ajustar();
// Cuelgan enloquecidamente de las hierbas harapos de plata
let reporte = new Reporte();
// Donde el sol de la orgullosa montaña luce:
añadirAReporte(balanceDeCuenta, reporte);
// Un pequeño valle espumoso de luz.
```

{{index "block comment"}}

Un comentario `//` va solo haste el final de la línea. Una sección de texto
entre `/*` y `*/` se ignorará en su totalidad, independientemente de
si contiene saltos de línea. Esto es útil para agregar bloques de
información sobre un archivo o un pedazo de programa.

```
/*
  Primero encontré este número garabateado en la parte posterior de
  un viejo cuaderno. Desde entonces, a menudo lo he visto,
  apareciendo en números de teléfono y en los números de serie de
  productos que he comprado. Obviamente me gusta, así que
  decidí quedármelo
*/
const miNumero = 11213;
```

## Resumen

Ahora sabes que un programa está construido a partir de declaraciones, las
cuales a veces pueden contener más declaraciones. Las declaraciones tienden a
contener expresiones, que a su vez se pueden construir a partir de
expresiones mas pequeñas.

Poner declaraciones una despues de otras te da un programa que es
ejecutado de arriba hacia abajo. Puedes introducir alteraciones en el
flujo de control usando declaraciones condicionales (`if`, `else`, y `switch`)
y ciclos (`while`, `do`, y `for`).

Las vinculaciones se pueden usar para archivar datos bajo un nombre, y son
utiles para el seguimiento de estado en tu programa. El entorno es el conjunto
de vinculaciones que se definen. Los sistemas de JavaScript siempre
incluyen por defecto un número de vinculaciones estándar útiles en tu entorno.

Las funciones son valores especiales que encapsulan una parte del programa.
Puedes invocarlas escribiendo `nombreDeLaFuncion(argumento1, argumento2)`. Tal
llamada a función es una expresión, y puede producir un valor.

## Ejercicios

{{index exercises}}

Si no estas seguro de cómo probar tus soluciones para los ejercicios, consulta
la [introducción](intro).

Cada ejercicio comienza con una descripción del problema. Lee eso y trata de
resolver el ejercicio. Si tienes problemas, considera leer las pistas
[después del ejercicio]{if interactive}[en el
[final del libro](pistas)]{if book}. Las soluciones completas para los
ejercicios no estan incluidas en este libro, pero puedes encontrarlas en línea
en [_eloquentjavascript.net/code_](https://eloquentjavascript.net/code#2).
Si quieres aprender algo de los ejercicios, te recomiendo mirar a las
soluciones solo despues de que hayas resuelto el ejercicio, o al menos despues
de que lo hayas intentando resolver por un largo tiempo y tengas un
ligero dolor de cabeza.

### Ciclo de un triángulo

{{index "triangle (exercise)"}}

Escriba un ((ciclo)) que haga siete llamadas a `console.log` para generar el
siguiente triángulo:

```{lang: null}
#
##
###
####
#####
######
#######
```

{{index [string, length]}}

Puede ser útil saber que puedes encontrar la longitud de un string
escribiendo `.length` después de él:

```
let abc = "abc";
console.log(abc.length);
// → 3
```

{{if interactive

La mayoría de los ejercicios contienen una pieza de código que puedes
modificar para resolver el ejercicio. Recuerda que puedes hacer clic en los
bloques de código para editarlos.

```
// Tu código aqui.
```
if}}

{{hint

{{index "triangle (exercise)"}}

Puedes comenzar con un programa que imprima los números del 1 al 7, al que
puedes derivar haciendo algunas modificaciones al [ejemplo de impresión de
números pares](estructura_de_programa#ciclos) dado anteriormente en el
capítulo, donde se introdujo el ciclo `for`.

Ahora considera la equivalencia entre números y strings de caracteres de
numeral. Puedes ir de 1 a 2 agregando 1 (`+= 1`). Puedes ir
de `"#"` a `"##"` agregando un caracter (`+= "#"`). Por lo tanto, tu
solución puede seguir de cerca el programa de impresión de números.

hint}}

### FizzBuzz

{{index "FizzBuzz (exercise)", loop, "conditional execution"}}

Escribe un programa que use `console.log` para imprimir todos los números de
1 a 100, con dos excepciones. Para números divisibles por 3, imprime
`"Fizz"` en lugar del número, y para los números divisibles por 5 (y
no 3), imprime `"Buzz"` en su lugar.

Cuando tengas eso funcionando, modifica tu programa para imprimir `"FizzBuzz"`,
para números que sean divisibles entre 3 y 5 (y aún imprimir
`"Fizz"` o `"Buzz"` para números divisibles por solo uno de ellos).

(Esta es en realidad una ((pregunta de entrevista)) que se ha dicho
elimina un porcentaje significativo de candidatos a programadores. Así que
si la puedes resolver, tu valor en el mercado laboral acaba de subir).

{{if interactive
```
// Tu código aquí.
```
if}}

{{hint

{{index "FizzBuzz (exercise)", "remainder operator", "% operator"}}

Ir a traves de los números es claramente el trabajo de un ciclo y seleccionar
qué imprimir es una cuestión de ejecución condicional. Recuerda el truco de
usar el operador restante (`%`) para verificar si un número es
divisible por otro número (tiene un residuo de cero).

En la primera versión, hay tres resultados posibles para cada
número, por lo que tendrás que crear una cadena `if`/`else if`/`else`.

{{index "|| operator", ["if keyword", chaining]}}

La segunda versión del programa tiene una solución directa y una
inteligente. La manera simple es agregar otra "rama" condicional para
probar precisamente la condición dada. Para el método inteligente, crea un
string que contenga la palabra o palabras a imprimir e imprimir ya sea esta
palabra o el número si no hay una palabra, posiblemente haciendo un buen uso
del operador `||`.

hint}}

### Tablero de ajedrez

{{index "chess board (exercise)", loop, [nesting, "of loops"], "newline character"}}

Escribe un programa que cree un string que represente una cuadrícula de 8 × 8,
usando caracteres de nueva línea para separar las líneas. En cada posición de la
cuadrícula hay un espacio o un carácter "#". Los caracteres deberían de
formar un tablero de ajedrez.

Pasar este string a `console.log` debería mostrar algo como esto:

```{lang: null}
 # # # #
# # # #
 # # # #
# # # #
 # # # #
# # # #
 # # # #
# # # #
```

Cuando tengas un programa que genere este patrón, define una
((vinculación)) `tamaño = 8` y cambia el programa para que funcione
con cualquier `tamaño`, dando como salida una cuadrícula con el alto y ancho
dados.

{{if interactive
```
// Tu código aquí.
```
if}}

{{hint

{{index "chess board (exercise)"}}

El string se puede construir comenzando con un string vacío (`""`) y
repetidamente agregando caracteres. Un carácter de nueva línea se escribe
`"\n"`.

{{index [nesting, "of loops"]}}

Para trabajar con dos ((dimensiones)), necesitarás un ((ciclo)) dentro de un
ciclo. Coloca ((llaves)) alrededor de los cuerpos de ambos ciclos para hacer
fácil de ver dónde comienzan y terminan. Intenta indentar adecuadamente estos
cuerpos. El orden de los ciclos debe seguir el orden en el que construimos
el string (línea por línea, izquierda a derecha, arriba a abajo). Entonces el
ciclo externo maneja las líneas y el ciclo interno maneja los caracteres
en una sola linea.

{{index "counter variable", "remainder operator", "% operator"}}

Necesitará dos vinculaciones para seguir tu progreso. Para saber si debes poner
un espacio o un signo de numeral en una posición determinada, podrías probar si
la suma de los dos contadores es par (`% 2`).

Terminar una línea al agregar un carácter de nueva línea debe suceder después
de que la línea ha sido creada, entonces haz esto después del ciclo interno
pero dentro del bucle externo.

hint}}
