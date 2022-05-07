# Expresiones Regulares

{{quote {author: "Jamie Zawinski", chapter: true}

Algunas personas, cuando confrontadas con un problema, piensan 'Ya sé, usaré
expresiones regulares.' Ahora tienen dos problemas.

quote}}

{{index "Zawinski, Jamie"}}

{{if interactive

{{quote {author: "Master Yuan-Ma", title: "The Book of Programming", chapter: true}

Yuan-Ma dijo: 'Cuando cortas contra el grano de la madera, mucha
fuerza se necesita. Cuando programas contra el grano del problema,
mucho código se necesita.

quote}}

if}}

{{figure {url: "img/chapter_picture_9.jpg", alt: "A railroad diagram", chapter: "square-framed"}}}

{{index evolution, adoption, integration}}

Las ((herramientas)) y técnicas de la programación sobreviven y se propagan de
una forma caótica y evolutiva. No siempre son los bonitas o las brillantes
las que ganan, sino más bien las que funcionan lo suficientemente bien
dentro del nicho correcto o que sucede se integran con otra pieza exitosa de
tecnología.

{{index "domain-specific language"}}

En este capítulo, discutiré una de esas herramientas, _((expresiones regulare))s_.
Las expresiones regulares son una forma de describir ((patrones)) en datos
de tipo string. Estas forman un lenguaje pequeño e independiente que es parte de
JavaScript y de muchos otros lenguajes y sistemas.

{{index [interface, design]}}

Las expresiones regulares son terriblemente incómodas y extremadamente útiles.
Su sintaxis es críptica, y la ((interfaz)) de programación que JavaScript
proporciona para ellas es torpe. Pero son una poderosa ((herramienta))  para
inspeccionar y procesar cadenas. Entender apropiadamente a las expresiones
regulares te hará un programador más efectivo.

## Creando una expresión regular

{{index ["regular expression", creation], "RegExp class", "literal expression", "slash character"}}

Una expresión regular es un tipo de objeto. Puede ser construido
con el constructor `RegExp` o escrito como un valor literal al
envolver un patrón en caracteres de barras diagonales (`/`).

```
let re1 = new RegExp("abc");
let re2 = /abc/;
```

Ambos objetos de expresión regular representan el mismo
((patrón)): un carácter _a_ seguido por una _b_ seguida de una _c_.

{{index "backslash character", "RegExp class"}}

Cuando se usa el constructor `RegExp`, el patrón se escribe como un
string normal, por lo que las reglas habituales se aplican a las barras
invertidas.

{{index ["regular expression", escaping], [escaping, "in regexps"], "slash character"}}

La segunda notación, donde el patrón aparece entre caracteres de barras diagonales,
trata a las barras invertidas de una forma diferente. Primero, dado que una
barra diagonal termina el patrón, tenemos que poner una barra invertida antes
de cualquier barra diagonal que queremos sea _parte_ del patrón. En adición,
las barras invertidas que no sean parte de códigos especiales de caracteres
(como `\n`) seran _preservadas_, en lugar de ignoradas, ya que están en strings, y
cambian el significado del patrón. Algunos caracteres, como los signos de
interrogación pregunta y los signos de adición, tienen significados especiales
en las expresiones regulares y deben ir precedidos por una barra inversa
si se pretende que representen al caracter en sí mismo.

```
let dieciochoMas = /dieciocho\+/;
```

## Probando por coincidencias

{{index matching, "test method", ["regular expression", methods]}}

Los objetos de expresión regular tienen varios métodos. El más simple
es `test` ("probar"). Si le pasas un string, retornar un ((Booleano))
diciéndote si el string contiene una coincidencia del patrón en la
expresión.

```
console.log(/abc/.test("abcde"));
// → true
console.log(/abc/.test("abxde"));
// → false
```

{{index pattern}}

Una ((expresión regular)) que consista solamente de caracteres no especiales
simplemente representara esa secuencia de caracteres. Si _abc_ ocurre
en cualquier parte del string con la que estamos probando (no solo al comienzo),
`test` retornara `true`.

## Conjuntos de caracteres

{{index "regular expression", "indexOf method"}}

Averiguar si un string contiene _abc_ bien podría hacerse con una llamada a
`indexOf`. Las expresiones regulares nos permiten expresar ((patrones)) más
complicados.

Digamos que queremos encontrar cualquier ((número)). En una expresión regular,
poner un ((conjunto)) de caracteres entre corchetes hace que esa parte de la
expresión coincida con cualquiera de los caracteres entre los corchetes.

Ambas expresiones coincidiran con todas los strings que contengan un ((dígito)):

```
console.log(/[0123456789]/.test("en 1992"));
// → true
console.log(/[0-9]/.test("en 1992"));
// → true
```

{{index "dash character"}}

Dentro de los corchetes, un guion (`-`) entre dos caracteres puede ser
utilizado para indicar un ((rango)) de caracteres, donde el orden es
determinado por el número ((Unicode)) del carácter. Los caracteres 0 a 9
estan uno al lado del otro en este orden (códigos 48 a 57), por lo que
`[0-9]` los cubre a todos y coincide con cualquier ((dígito)).

{{index whitespace, "alphanumeric character", "period character"}}

Un numero de caracteres comunes tienen sus propios atajos incorporados.
Los dígitos son uno de ellos: `\d` significa lo mismo que `[0-9]`.

{{index "newline character"}}

{{table {cols: [1, 5]}}}

| `\d`    | Cualquier caracter ((dígito))
| `\w`    | Un caracter alfanumérico
| `\s`    | Cualquier carácter de espacio en blanco (espacio, tabulación, nueva línea y similar)
| `\D`    | Un caracter que _no_ es un dígito
| `\W`    | Un caracter no alfanumérico
| `\S`    | Un caracter que no es un espacio en blanco
| `.`     | Cualquier caracter a excepción de una nueva línea

Por lo que podrías coincidir con un formato de ((fecha)) y ((hora)) como 30-01-2003
15:20 con la siguiente expresión:

```
let fechaHora = /\d\d-\d\d-\d\d\d\d \d\d:\d\d/;
console.log(fechaHora.test("30-01-2003 15:20"));
// → true
console.log(fechaHora.test("30-jan-2003 15:20"));
// → false
```

{{index "backslash character"}}

Eso se ve completamente horrible, no? La mitad de la expresión son barras invertidas,
produciendo un ruido de fondo que hace que sea difícil detectar el ((patrón))
real que queremos expresar. Veremos una versión ligeramente mejorada de esta
expresión [más tarde](regexp#date_regexp_counted).

{{index [escaping, "in regexps"], "regular expression", set}}

Estos códigos de barra invertida también pueden usarse dentro de ((corchetes)).
Por ejemplo, `[\d.]` representa cualquier dígito o un carácter de punto.
Pero el punto en sí mismo, entre corchetes, pierde su significado especial.
Lo mismo va para otros caracteres especiales, como `+`.

{{index "square brackets", inversion, "caret character"}}

Para _invertir_ un conjunto de caracteres, es decir, para expresar que deseas
coincidir con cualquier carácter _excepto_  con los que están en el
conjunto—puedes escribir un carácter de intercalación (`^`) después del
corchete de apertura.

```
let noBinario = /[^01]/;
console.log(noBinario.test("1100100010100110"));
// → false
console.log(noBinario.test("1100100010200110"));
// → true
```

## Repitiendo partes de un patrón

{{index ["regular expression", repetition]}}

Ya sabemos cómo hacer coincidir un solo dígito. Qué pasa si queremos hacer
coincidir un número completo—una ((secuencia)) de uno o más ((dígito))s?

{{index "plus character", repetition, "+ operator"}}

Cuando pones un signo más (`+`) después de algo en una expresión regular,
este indica que el elemento puede repetirse más de
una vez. Por lo tanto, `/\d+/` coincide con uno o más caracteres de dígitos.

```
console.log(/'\d+'/.test("'123'"));
// → true
console.log(/'\d+'/.test("''"));
// → false
console.log(/'\d*'/.test("'123'"));
// → true
console.log(/'\d*'/.test("''"));
// → true
```

{{index "* operator", asterisk}}

La estrella (`*`) tiene un significado similar pero también permite que el
patrón coincida cero veces. Algo con una estrella después de el nunca evitara un
patrón de coincidirlo—este solo coincidirá con cero instancias si no
puede encontrar ningun texto adecuado para coincidir.

{{index "British English", "American English", "question mark"}}

Un signo de interrogación hace que alguna parte de un patrón sea _((opcional))_,
lo que significa que puede ocurrir cero o mas veces. En el siguiente ejemplo,
el carácter _h_ está permitido, pero el patrón también retorna verdadero
cuando esta letra no esta.

```
let reusar = /reh?usar/;
console.log(reusar.test("rehusar"));
// → true
console.log(reusar.test("reusar"));
// → true
```

{{index repetition, "curly braces"}}

Para indicar que un patrón deberia ocurrir un número preciso de veces, usa
llaves. Por ejemplo, al poner `{4}` después de un elemento, hace que requiera
que este ocurra exactamente cuatro veces. También es posible especificar un
((rango)) de esta manera: `{2,4}` significa que el elemento debe ocurrir al
menos dos veces y como máximo cuatro veces.

{{id date_regexp_counted}}

Aquí hay otra versión del patrón ((fecha)) y ((hora)) que
permite días tanto en ((dígitos)) individuales como dobles, meses y horas.
Es también un poco más fácil de descifrar.

```
let fechaHora = /\d{1,2}-\d{1,2}-\d{4} \d{1,2}:\d{2}/;
console.log(fechaHora.test("30-1-2003 8:45"));
// → true
```

También puedes especificar ((rangos)) de final abierto al usar ((llaves))
omitiendo el número después de la coma. Entonces, `{5,}` significa cinco o más
veces.

## Agrupando subexpresiones

{{index ["regular expression", grouping], grouping}}

Para usar un operador como `*` o `+` en más de un elemento a la vez,
tienes que usar ((paréntesis)). Una parte de una expresión regular que
se encierre entre paréntesis cuenta como un elemento único en cuanto a
los operadores que la siguen están preocupados.

```
let caricaturaLlorando = /boo+(hoo+)+/i;
console.log(caricaturaLlorando.test("Boohoooohoohooo"));
// → true
```

{{index crying}}

El primer y segundo caracter `+` aplican solo a la segunda _o_ en
_boo_ y _hoo_, respectivamente. El tercer `+` se aplica a la totalidad
del grupo `(hoo+)`, haciendo coincidir una o más secuencias como esa.

{{index "case sensitivity", capitalization, ["regular expression", flags]}}

La `i` al final de la expresión en el ejemplo hace que esta expresión regular
sea insensible a mayúsculas y minúsculas, lo que permite que coincida con la
letra mayúscula _B_ en el string que se le da de entrada, asi el
patrón en sí mismo este en minúsculas.

## Coincidencias y grupos

{{index ["regular expression", grouping], "exec method", array}}

El método `test` es la forma más simple de hacer coincidir una
expresión. Solo te dice si coincide y nada más.
Las expresiones regulares también tienen un método `exec` ("ejecutar") que
retorna `null` si no se encontró una coincidencia y retorna un objeto con
información sobre la coincidencia de lo contrario.

```
let coincidencia = /\d+/.exec("uno dos 100");
console.log(coincidencia);
// → ["100"]
console.log(coincidencia.index);
// → 8
```

{{index "index property", [string, indexing]}}

Un objeto retornado por `exec` tiene una propiedad `index` ("indice") que nos
dice _donde_ en el string comienza la coincidencia exitosa. Aparte de eso,
el objeto parece (y de hecho es) un array de strings, cuyo
primer elemento es el string que coincidio—en el ejemplo anterior,
esta es la secuencia de ((dígito))s que estábamos buscando.

{{index [string, methods], "match method"}}

Los valores de tipo string tienen un método `match` que se comporta de
manera similar.

```
console.log("uno dos 100".match(/\d+/));
// → ["100"]
```

{{index grouping, "capture group", "exec method"}}

Cuando la expresión regular contenga subexpresiones agrupadas con
paréntesis, el texto que coincida con esos grupos también aparecerá en
el array. La coincidencia completa es siempre el primer elemento. El siguiente
elemento es la parte que coincidio con el primer grupo (el que abre
paréntesis primero en la expresión), luego el segundo grupo, y
asi sucesivamente.

```
let textoCitado = /'([^']*)'/;
console.log(textoCitado.exec("ella dijo 'hola'"));
// → ["'hola'", "hola"]
```

{{index "capture group"}}

Cuando un grupo no termina siendo emparejado en absoluto (por ejemplo, cuando
es seguido de un signo de interrogación), su posición en el array de salida
sera `undefined`. Del mismo modo, cuando un grupo coincida multiples veces,
solo la ultima coincidencia termina en el array.

```
console.log(/mal(isimo)?/.exec("mal"));
// → ["mal", undefined]
console.log(/(\d)+/.exec("123"));
// → ["123", "3"]
```

{{index "exec method", ["regular expression", methods], extraction}}

Los grupos pueden ser útiles para extraer partes de un string. Si no solo
queremos verificar si un string contiene una ((fecha)) pero también
extraerla y construir un objeto que la represente, podemos envolver
paréntesis alrededor de los patrones de dígitos y tomar directamente la fecha
del resultado de `exec`.

Pero primero, un breve desvío, en el que discutiremos la forma incorporada de
representar valores de fecha y ((hora)) en JavaScript.

## La clase Date ("Fecha")

{{index constructor, "Date class"}}

JavaScript tiene una clase estándar para representar ((fecha))s—o mejor dicho,
puntos en el ((tiempo)). Se llama `Date`. Si simplemente creas un objeto
fecha usando `new`, obtienes la fecha y hora actual.

```{test: no}
console.log(new Date());
// → Mon Nov 13 2017 16:19:11 GMT+0100 (CET)
```

{{index "Date class"}}

También puedes crear un objeto para un tiempo específico.

```
console.log(new Date(2009, 11, 9));
// → Wed Dec 09 2009 00:00:00 GMT+0100 (CET)
console.log(new Date(2009, 11, 9, 12, 59, 59, 999));
// → Wed Dec 09 2009 12:59:59 GMT+0100 (CET)
```

{{index "zero-based counting", [interface, design]}}

JavaScript usa una convención en donde los números de los meses comienzan en
cero (por lo que Diciembre es 11), sin embargo, los números de los días
comienzan en uno. Esto es confuso y tonto. Ten cuidado.

Los últimos cuatro argumentos (horas, minutos, segundos y milisegundos)
son opcionales y se toman como cero cuando no se dan.

{{index "getTime method"}}

Las marcas de tiempo se almacenan como la cantidad de milisegundos desde el
inicio de 1970, en la ((zona horaria)) UTC. Esto sigue una convención
establecida por el "((Tiempo Unix))", el cual se inventó en ese momento.
Puedes usar números negativos para los tiempos anteriores a 1970. Usar el método
`getTime` ("obtenerTiempo") en un objeto fecha retorna este número.
Es bastante grande, como te puedes imaginar.

```
console.log(new Date(2013, 11, 19).getTime());
// → 1387407600000
console.log(new Date(1387407600000));
// → Thu Dec 19 2013 00:00:00 GMT+0100 (CET)
```

{{index "Date.now function", "Date class"}}

Si le das al constructor `Date` un único argumento, ese argumento sera
tratado como un conteo de milisegundos. Puedes obtener el recuento
de milisegundos actual creando un nuevo objeto `Date` y llamando
`getTime` en él o llamando a la función `Date.now`.

{{index "getFullYear method", "getMonth method", "getDate method", "getHours method", "getMinutes method", "getSeconds method", "getYear method"}}

Los objetos de fecha proporcionan métodos como `getFullYear`
("obtenerAñoCompleto"), `getMonth` ("obtenerMes"), `getDate` ("obtenerFecha"),
`getHours` ("obtenerHoras"), `getMinutes` ("obtenerMinutos"), y `getSeconds`
("obtenerSegundos") para extraer sus componentes. Además de `getFullYear`,
también existe `getYear` ("obtenerAño"), que te da como resultado un valor
de año de dos dígitos bastante inútil (como `93` o `14`).

{{index "capture group", "getDate function"}}

Al poner ((paréntesis)) alrededor de las partes de la expresión en las que
estamos interesados, ahora podemos crear un objeto de fecha a partir de un
string.

```
function obtenerFecha(string) {
  let [_, dia, mes, año] =
    /(\d{1,2})-(\d{1,2})-(\d{4})/.exec(string);
  return new Date(año, mes - 1, dia);
}
console.log(obtenerFecha("30-1-2003"));
// → Thu Jan 30 2003 00:00:00 GMT+0100 (CET)
```

{{index destructuring, "underscore character"}}

La vinculación `_` (guion bajo) es ignorada, y solo se usa para omitir el
elemento de coincidencia completa en el array retornado por `exec`.

## Palabra y límites de string

{{index matching, ["regular expression", boundary]}}

Desafortunadamente, `obtenerFecha` felizmente también extraerá la absurda
fecha 00-1-3000 del string `"100-1-30000"`. Una coincidencia puede suceder
en cualquier lugar del string, por lo que en este caso, esta simplemente
comenzará en el segundo carácter y terminara en el penúltimo carácter.

{{index boundary, "caret character", "dollar sign"}}

Si queremos hacer cumplir que la coincidencia deba abarcar el string completamente,
puedes agregar los marcadores `^` y `$`. El signo de intercalación ("^")
coincide con el inicio del string de entrada, mientras que el signo de dólar
coincide con el final.  Entonces, `/^\d+$/` coincide con un string compuesto
por uno o más dígitos, `/^!/` coincide con cualquier string que comience con
un signo de exclamación, y `/x^/` no coincide con ningun string
(no puede haber una _x_ antes del inicio del string).

{{index "word boundary", "word character"}}

Si, por el otro lado, solo queremos asegurarnos de que la fecha comience y
termina en un límite de palabras, podemos usar el marcador `\b`. Un límite de
palabra puede ser el inicio o el final del string o cualquier punto en el
string que tenga un carácter de palabra (como en `\w`) en un lado y un
carácter de no-palabra en el otro.

```
console.log(/cat/.test("concatenar"));
// → true
console.log(/\bcat\b/.test("concatenar"));
// → false
```

{{index matching}}

Ten en cuenta que un marcador de límite no coincide con un carácter real. Solo
hace cumplir que la expresión regular coincida solo cuando una cierta
condición se mantenga en el lugar donde aparece en el patrón.

## Patrones de elección

{{index branching, ["regular expression", alternatives], "farm example"}}

Digamos que queremos saber si una parte del texto contiene no solo un número
pero un número seguido de una de las palabras _cerdo_, _vaca_, o _pollo_,
o cualquiera de sus formas plurales.

Podríamos escribir tres expresiones regulares y probarlas a su vez, pero
hay una manera más agradable. El ((carácter de tubería)) (`|`) denota una
((elección)) entre el patrón a su izquierda y el patrón a su
derecha. Entonces puedo decir esto:

```
let conteoAnimales = /\b\d+ (cerdo|vaca|pollo)s?\b/;
console.log(conteoAnimales.test("15 cerdo"));
// → true
console.log(conteoAnimales.test("15 cerdopollos"));
// → false
```

{{index parentheses}}

Los paréntesis se pueden usar para limitar la parte del patrón a la que aplica
el operador de tuberia, y puedes poner varios de estos operadores unos a los
lados de los otros para expresar una elección entre más de dos alternativas.

## Las mecánicas del emparejamiento

{{index ["regular expression", matching], [matching, algorithm], searching}}

Conceptualmente, cuando usas `exec` o `test` el motor de la expresión regular
busca una coincidencia en tu string al tratar de hacer coincidir la
expresión primero desde el comienzo del string, luego desde el segundo
caracter, y así sucesivamente hasta que encuentra una coincidencia o llega al
final del string. Retornara la primera coincidencia que se puede encontrar o
fallara en encontrar cualquier coincidencia.

{{index ["regular expression", matching], [matching, algorithm]}}

Para realmente hacer la coincidencia, el motor trata una expresión regular
algo así como un ((diagrama de flujo)). Este es el diagrama para la
expresión de ganado en el ejemplo anterior:

{{figure {url: "img/re_pigchickens.svg", alt: "Visualization of /\\b\\d+ (pig|cow|chicken)s?\\b/"}}}

{{index traversal}}

Nuestra expresión coincide si podemos encontrar un camino desde el lado
izquierdo del diagrama al lado derecho. Mantenemos una posición actual en el
string, y cada vez que nos movemos a través de una caja, verificaremos que la
parte del string después de nuestra posición actual coincida con esa
caja.

Entonces, si tratamos de coincidir `"los 3 cerdos"` desde la posición 4,
nuestro progreso a través del diagrama de flujo se vería así:

- En la posición 4, hay un ((límite)) de palabra, por lo que podemos pasar
  la primera caja.

- Aún en la posición 4, encontramos un dígito, por lo que también podemos pasar
  la segunda caja.

- En la posición 5, una ruta regresa a antes de la segunda caja (dígito),
  mientras que la otro se mueve hacia adelante a través de la caja que contiene
  un caracter de espacio simple. Hay un espacio aquí, no un dígito, asi que
  debemos tomar el segundo camino.

- Ahora estamos en la posición 6 (el comienzo de "cerdos") y en el camino de
  tres vías en el diagrama. No vemos "vaca" o "pollo" aquí, pero
  vemos "cerdo", entonces tomamos esa rama.

- En la posición 9, después de la rama de tres vías, un camino se salta la
  caja _s_ y va directamente al límite de la palabra final, mientras que la
  otra ruta coincide con una _s_. Aquí hay un carácter _s_, no una palabra
  límite, por lo que pasamos por la caja _s_.

- Estamos en la posición 10 (al final del string) y solo podemos hacer coincidir
  una palabra ((límite)). El final de un string cuenta como un límite de palabra,
  así que pasamos por la última caja y hemos emparejado con éxito este string.

{{id backtracking}}

## Retrocediendo

{{index ["regular expression", backtracking], "binary number", "decimal number", "hexadecimal number", "flow diagram", [matching, algorithm], backtracking}}

La expresión regular `/\b([01]+b|[\da-f]+h|\d+)\b/` coincide con un
número binario seguido de una _b_, un número hexadecimal (es decir, en base
16, con las letras _a_ a _f_ representando los dígitos 10 a 15)
seguido de una _h_, o un número decimal regular sin caracter de sufijo.
Este es el diagrama correspondiente:

{{figure {url: "img/re_number.svg", alt: "Visualization of /\\b([01]+b|\\d+|[\\da-f]+h)\\b/"}}}

{{index branching}}

Al hacer coincidir esta expresión, a menudo sucederá que la rama superior
(binaria) sea ingresada aunque la entrada en realidad no contenga un número
binario. Al hacer coincidir el string `"103"`, por ejemplo, queda claro solo en
el 3 que estamos en la rama equivocada. El string _si_ coincide con la
expresión, pero no con la rama en la que nos encontramos actualmente.

{{index backtracking, searching}}

Entonces el "emparejador" _retrocede_. Al ingresar a una rama, este recuerda su
posición actual (en este caso, al comienzo del string, justo después
del primer cuadro de límite en el diagrama) para que pueda retroceder e intentar
otra rama si la actual no funciona. Para el string `"103"`, después de
encontrar los 3 caracteres, comenzará a probar la
rama para números hexadecimales, que falla nuevamente porque no hay
_h_ después del número. Por lo tanto, intenta con la rama de número decimal.
Esta encaja, y se informa de una coincidencia después de todo.

{{index [matching, algorithm]}}

El emparejador se detiene tan pronto como encuentra una coincidencia completa.
Esto significa que si múltiples ramas podrían coincidir con un string, solo
la primera (ordenado por donde las ramas aparecen en la expresión regular) es
usada.

El retroceso también ocurre para ((repetición)) de operadores como + y `*`.
Si hace coincidir `/^.*x/` contra `"abcxe"`, la parte `.*` intentará primero
consumir todo el string. El motor entonces se dará cuenta de que
necesita una _x_ para que coincida con el patrón. Como no hay _x_ al pasar
el final del string, el operador de estrella intenta hacer coincidir un
caracter menos. Pero el emparejador tampoco encuentra una _x_ después de `abcx`,
por lo que retrocede nuevamente, haciendo coincidir el operador de estrella con
`abc`. _Ahora_ encuentra una _x_ donde lo necesita e informa de una
coincidencia exitosa de las posiciones 0 a 4.

{{index performance, complexity}}

Es posible escribir expresiones regulares que harán un _monton_ de
retrocesos. Este problema ocurre cuando un patrón puede coincidir con una
pieza de entrada en muchas maneras diferentes. Por ejemplo, si nos
confundimos mientras escribimos una expresión regular de números binarios,
podríamos accidentalmente escribir algo como `/([01]+)+b/`.

{{figure {url: "img/re_slow.svg", alt: "Visualization of /([01]+)+b/",width: "6cm"}}}

{{index "inner loop", [nesting, "in regexps"]}}

Si intentas hacer coincidir eso con algunas largas series de ceros y unos sin
un caracter _b_ al final, el emparejador primero pasara por el ciclo interior
hasta que se quede sin dígitos. Entonces nota que no hay _b_, asi que
retrocede una posición, atraviesa el ciclo externo una vez, y
se da por vencido otra vez, tratando de retroceder fuera del ciclo interno una
vez más. Continuará probando todas las rutas posibles a través de estos dos
bucles. Esto significa que la cantidad de trabajo se _duplica_ con cada
caracter. Incluso para unas pocas docenas de caracters, la coincidencia
resultante tomará prácticamente para siempre.

## El método replace

{{index "replace method", "regular expression"}}

Los valores de string tienen un método `replace` ("reemplazar") que se puede
usar para reemplazar parte del string con otro string.

```
console.log("papa".replace("p", "m"));
// → mapa
```

{{index ["regular expression", flags], ["regular expression", global]}}

El primer argumento también puede ser una expresión regular, en cuyo caso ña
primera coincidencia de la expresión regular es reemplazada. Cuando una opción
`g` (para _global_) se agrega a la expresión regular, _todas_ las coincidencias
en el string será reemplazadas, no solo la primera.

```
console.log("Borobudur".replace(/[ou]/, "a"));
// → Barobudur
console.log("Borobudur".replace(/[ou]/g, "a"));
// → Barabadar
```

{{index [interface, design], argument}}

Hubiera sido sensato si la elección entre reemplazar una coincidencia
o todas las coincidencias se hiciera a través de un argumento adicional en
`replace` o proporcionando un método diferente, `replaceAll` ("reemplazarTodas").
Pero por alguna desafortunada razón, la elección se basa en una propiedad de
los expresiones regulares en su lugar.

{{index grouping, "capture group", "dollar sign", "replace method", ["regular expression", grouping]}}

El verdadero poder de usar expresiones regulares con `replace` viene del
hecho de que podemos referirnos a grupos coincidentes en la string de reemplazo.
Por ejemplo, supongamos que tenemos una gran string que contenga los nombres de
personas, un nombre por línea, en el formato `Apellido, Nombre`. Si
deseamos intercambiar estos nombres y eliminar la coma para obtener un
formato `Nombre Apellido`, podemos usar el siguiente código:

```
console.log(
  "Liskov, Barbara\nMcCarthy, John\nWadler, Philip"
    .replace(/(\w+), (\w+)/g, "$2 $1"));
// → Barbara Liskov
//   John McCarthy
//   Philip Wadler
```

Los `$1` y `$2` en el string de reemplazo se refieren a los grupos entre
paréntesis del patrón. `$1` se reemplaza por el texto que coincide
con el primer grupo, `$2` por el segundo, y así sucesivamente, hasta `$9`.
Puedes hacer referencia a la coincidencia completa con `$&`.

{{index [function, "higher-order"], grouping, "capture group"}}

Es posible pasar una función, en lugar de un string, como segundo
argumento para `replace`. Para cada reemplazo, la función será
llamada con los grupos coincidentes (así como con la coincidencia completa)
como argumentos, y su valor de retorno se insertará en el nuevo string.

Aquí hay un pequeño ejemplo:

```
let s = "la cia y el fbi";
console.log(s.replace(/\b(fbi|cia)\b/g,
            str => str.toUpperCase()));
// → la CIA y el FBI
```

Y aquí hay uno más interesante:

```
let almacen = "1 limon, 2 lechugas, y 101 huevos";
function menosUno(coincidencia, cantidad, unidad) {
  cantidad = Number(cantidad) - 1;
  if (cantidad == 1) { // solo queda uno, remover la 's'
    unidad = unidad.slice(0, unidad.length - 1);
  } else if (cantidad == 0) {
    cantidad = "sin";
  }
  return cantidad + " " + unidad;
}
console.log(almacen.replace(/(\d+) (\w+)/g, menosUno));
// → sin limon, 1 lechuga, y 100 huevos
```

Esta función toma un string, encuentra todas las ocurrencias de un número
seguido de una palabra alfanumérica, y retorna un string en la que cada
ocurrencia es decrementada por uno.

El grupo `(\d+)` termina como el argumento `cantidad` para la función,
y el grupo `(\w+)` se vincula a `unidad`. La función convierte
`cantidad` a un número—lo que siempre funciona, ya que coincidio con `\d+`—y
realiza algunos ajustes en caso de que solo quede uno o cero.

## Codicia

{{index greed, "regular expression"}}

Es posible usar `replace` para escribir una función que elimine todo los
((comentario))s de un fragmento de ((código)) JavaScript. Aquí hay un primer
intento:

```{test: wrap}
function removerComentarios(codigo) {
  return codigo.replace(/\/\/.*|\/\*[^]*\*\//g, "");
}
console.log(removerComentarios("1 + /* 2 */3"));
// → 1 + 3
console.log(removerComentarios("x = 10;// ten!"));
// → x = 10;
console.log(removerComentarios("1 /* a */+/* b */ 1"));
// → 1  1
```

{{index "period character", "slash character", "newline character", "empty set", "block comment", "line comment"}}

La parte anterior al operador _o_ coincide con dos caracteres de barra inclinada
seguido de cualquier número de caracteres que no sean nuevas lineas. La parte
para los comentarios de líneas múltiples es más complicado. Usamos `[^]`
(cualquier caracter que no está en el conjunto de caracteres vacíos)
como una forma de unir cualquier caracter. No podemos simplemente usar un
punto aquí porque los comentarios de bloque pueden continuar en una nueva
línea, y el carácter del período no coincide con caracteres de nuevas lineas.

Pero la salida de la última línea parece haber salido mal. Por qué?

{{index backtracking, greed, "regular expression"}}

La parte `[^]*` de la expresión, como describí en la sección
retroceder, primero coincidirá tanto como sea posible. Si eso causa un falo en
la siguiente parte del patrón, el emparejador retrocede un caracter
e intenta nuevamente desde allí. En el ejemplo, el emparejador primero intenta
emparejar el resto del string y luego se mueve hacia atrás desde allí. Este
encontrará una ocurrencia de `*/` después de retroceder cuatro caracteres y
emparejar eso. Esto no es lo que queríamos, la intención era hacer coincidir un
solo comentario, no ir hasta el final del código y encontrar
el final del último comentario de bloque.

Debido a este comportamiento, decimos que los operadores de repetición (`+`, `*`,
`?` y `{}`) son _ ((codiciosos)), lo que significa que coinciden con tanto como
pueden y retroceden desde allí. Si colocas un ((signo de interrogación)) después
de ellos (`+?`, `*?`, `??`, `{}?`), se vuelven no-codiciosos y comienzan a
hacer coincidir lo menos posible, haciendo coincidir más solo cuando el
patrón restante no se ajuste a la coincidencia más pequeña.

Y eso es exactamente lo que queremos en este caso. Al hacer que la estrella
coincida con el tramo más pequeño de caracteres que nos lleve a un `*/`,
consumimos un comentario de bloque y nada más.

```{test: wrap}
function removerComentarios(codigo) {
  return codigo.replace(/\/\/.*|\/\*[^]*?\*\//g, "");
}
console.log(removerComentarios("1 /* a */+/* b */ 1"));
// → 1 + 1
```

Una gran cantidad de ((errores)) en los programas de ((expresiones regulares))
se pueden rastrear a intencionalmente usar un operador codicioso, donde uno
que no sea codicioso trabajaria mejor. Al usar un operador de ((repetición)),
considera la variante no-codiciosa primero.

## Creando objetos RegExp dinámicamente

{{index ["regular expression", creation], "underscore character", "RegExp class"}}

Hay casos en los que quizás no sepas el ((patrón)) exacto
necesario para coincidir cuando estes escribiendo tu código. Imagina
que quieres buscar el nombre del usuario en un texto y encerrarlo en
caracteres de subrayado para que se destaque. Como solo sabrás el
nombrar una vez que el programa se está ejecutando realmente, no puedes
usar la notación basada en barras.

Pero puedes construir un string y usar el ((constructor)) `RegExp` en el.
Aquí hay un ejemplo:

```
let nombre = "harry";
let texto = "Harry es un personaje sospechoso.";
let regexp = new RegExp("\\b(" + nombre + ")\\b", "gi");
console.log(texto.replace(regexp, "_$1_"));
// → _Harry_ es un personaje sospechoso.
```

{{index ["regular expression", flags], "backslash character"}}

Al crear los marcadores de ((límite)) `\b`, tenemos que usar dos
barras invertidas porque las estamos escribiendo en un string normal, no en una
expresión regular contenida en barras. El segundo argumento para el
constructor `RegExp` contiene las opciones para la expresión regular—en este
caso, `"gi"` para global e insensible a mayúsculas y minúsculas.

Pero, y si el nombre es `"dea+hl[]rd"` porque nuestro usuario es un ((nerd))
adolescente? Eso daría como resultado una expresión regular sin sentido que
en realidad no coincidirá con el nombre del usuario.

{{index "backslash character", [escaping, "in regexps"], ["regular expression", escaping]}}

Para solucionar esto, podemos agregar barras diagonales inversas antes de
cualquier caracter que tenga un significado especial.

```
let nombre = "dea+hl[]rd";
let texto = "Este sujeto dea+hl[]rd es super fastidioso.";
let escapados = nombre.replace(/[\\[.+*?(){|^$]/g, "\\$&");
let regexp = new RegExp("\\b" + escapados + "\\b", "gi");
console.log(texto.replace(regexp, "_$&_"));
// → Este sujeto _dea+hl[]rd_ es super fastidioso.
```

## El método search

{{index searching, ["regular expression", methods], "indexOf method", "search method"}}

El método `indexOf` en strings no puede invocarse con una expresión regular.
Pero hay otro método, `search` ("buscar"), que espera una expresión regular.
Al igual que `indexOf`, retorna el primer índice en
que se encontró la expresión, o -1 cuando no se encontró.

```
console.log("  palabra".search(/\S/));
// → 2
console.log("    ".search(/\S/));
// → -1
```

Desafortunadamente, no hay forma de indicar que la coincidencia debería comenzar
a partir de un desplazamiento dado (como podemos con el segundo argumento para
`indexOf`), que a menudo sería útil.

## La propiedad lastIndex

{{index "exec method", "regular expression"}}

De manera similar el método `exec` no proporciona una manera conveniente de
comenzar buscando desde una posición dada en el string. Pero proporciona una
manera *in*conveniente.

{{index ["regular expression", matching], matching, "source property", "lastIndex property*"}}

Los objetos de expresión regular tienen propiedades. Una de esas propiedades es
`source` ("fuente"), que contiene el string de donde se creó la expresión.
Otra propiedad es `lastIndex` ("ultimoIndice"), que controla, en algunas
circunstancias limitadas, donde comenzará la siguiente coincidencia.

{{index [interface, design], "exec method", ["regular expression", global]}}

Esas circunstancias son que la expresión regular debe tener la opción global
(`g`) o adhesiva (`y`) habilitada, y la coincidencia debe suceder
a través del método `exec`. De nuevo, una solución menos confusa hubiese
sido permitir que un argumento adicional fuera pasado a `exec`, pero
la confusión es una característica esencial de la interfaz de las
expresiones regulares de JavaScript.

```
let patron = /y/g;
patron.lastIndex = 3;
let coincidencia = patron.exec("xyzzy");
console.log(coincidencia.index);
// → 4
console.log(patron.lastIndex);
// → 5
```

{{index "side effect", "lastIndex property"}}

Si la coincidencia fue exitosa, la llamada a `exec` actualiza automáticamente
a la propiedad `lastIndex` para que apunte después de la coincidencia. Si no
se encontraron coincidencias, `lastIndex` vuelve a cero, que es también el
valor que tiene un objeto de expresión regular recién construido.

La diferencia entre las opciones globales y las adhesivas es que, cuandoa
adhesivo está habilitado, la coincidencia solo tendrá éxito si comienza
directamente en `lastIndex`, mientras que con global, buscará una
posición donde pueda comenzar una coincidencia.

```
let global = /abc/g;
console.log(global.exec("xyz abc"));
// → ["abc"]
let adhesivo = /abc/y;
console.log(adhesivo.exec("xyz abc"));
// → null
```

{{index bug}}

Cuando se usa un valor de expresión regular compartido para múltiples llamadas
a `exec`, estas actualizaciones automáticas a la propiedad `lastIndex` pueden
causar problemas. Tu expresión regular podría estar accidentalmente comenzando
en un índice que quedó de una llamada anterior.

```
let digito = /\d/g;
console.log(digito.exec("aqui esta: 1"));
// → ["1"]
console.log(digito.exec("y ahora: 1"));
// → null
```

{{index ["regular expression", global], "match method"}}

Otro efecto interesante de la opción global es que cambia la
forma en que funciona el método `match` en strings. Cuando se llama con una
expresión global, en lugar de retornar un matriz similar al retornado por
`exec`,` match` encontrará _todas_ las coincidencias del patrón en el string
y retornar un array que contiene los strings coincidentes.

```
console.log("Banana".match(/an/g));
// → ["an", "an"]
```

Por lo tanto, ten cuidado con las expresiones regulares globales. Los casos
donde son necesarias—llamadas a `replace` y lugares donde deseas
explícitamente usar `lastIndex`—son generalmente los únicos lugares donde
querras usarlas.

### Ciclos sobre coincidencias

{{index "lastIndex property", "exec method", loop}}

Una cosa común que hacer es escanear todas las ocurrencias de un patrón
en un string, de una manera que nos de acceso al objeto de coincidencia en el
cuerpo del ciclo. Podemos hacer esto usando `lastIndex` y `exec`.

```
let entrada = "Un string con 3 numeros en el... 42 y 88.";
let numero = /\b\d+\b/g;
let coincidencia;
while (coincidencia = numero.exec(entrada)) {
  console.log("Se encontro", coincidencia[0], "en", coincidencia.index);
}
// → Se encontro 3 en 14
//   Se encontro 42 en 33
//   Se encontro 88 en 38
```

{{index "while loop", "= operator"}}

Esto hace uso del hecho de que el valor de una expresión de ((asignación)) (`=`)
es el valor asignado. Entonces al usar `coincidencia = numero.exec(entrada)`
como la condición en la declaración `while`,
realizamos la coincidencia al inicio de cada iteración, guardamos
su resultado en una ((vinculación)), y terminamos de repetir cuando no se
encuentran más coincidencias.

{{id ini}}
## Análisis de un archivo INI

{{index comment, "file format", "enemies example", "INI file"}}

Para concluir el capítulo, veremos un problema que requiere de
((expresiones regulares)). Imagina que estamos escribiendo un programa para
recolectar automáticamente información sobre nuestros enemigos de el
((Internet)). (No escribiremos ese programa aquí, solo la
parte que lee el archivo de ((configuración)). Lo siento.) El archivo
de configuración se ve así:

```{lang: "text/plain"}
motordebusqueda=https://duckduckgo.com/?q=$1
malevolencia=9.7

; los comentarios estan precedidos por un punto y coma...
; cada seccion contiene un enemigo individual

[larry]
nombrecompleto=Larry Doe
tipo=bravucon del preescolar
sitioweb=http://www.geocities.com/CapeCanaveral/11451

[davaeorn]
nombrecompleto=Davaeorn
tipo=hechizero malvado
directoriosalida=/home/marijn/enemies/davaeorn
```

{{index grammar}}

Las reglas exactas para este formato (que es un formato ampliamente utilizado,
usualmente llamado un archivo _INI_) son las siguientes:

- Las líneas en blanco y líneas que comienzan con punto y coma se ignoran.

- Las líneas envueltas en `[` y `]` comienzan una nueva ((sección)).

- Líneas que contienen un identificador alfanumérico seguido de un
  carácter `=` agregan una configuración a la sección actual.

- Cualquier otra cosa no es válida.

Nuestra tarea es convertir un string como este en un objeto cuyas
propiedades contengas strings para configuraciones sin sección y sub-objetos
para secciones, con esos subobjetos conteniendo la configuración de la sección.

{{index "carriage return", "line break", "newline character"}}

Dado que el formato debe procesarse ((línea)) por línea, dividir
el archivo en líneas separadas es un buen comienzo. Usamos `string.split("\n")`
para hacer esto en el [Capítulo 4](datos#split). Algunos sistemas operativos,
sin embargo, usan no solo un carácter de nueva línea para separar lineas
sino un carácter de retorno de carro seguido de una nueva línea
(`"\r\n"`). Dado que el método `split` también permite una expresión
regular como su argumento, podemos usar una expresión regular como
`/\r?\n/` para dividir el string de una manera que permita tanto `"\n"` como
`"\r\n"` entre líneas.

```{startCode: true}
function analizarINI(string) {
  // Comenzar con un objeto para mantener los campos de nivel superior
  let resultado = {};
  let seccion = resultado;
  string.split(/\r?\n/).forEach(linea => {
    let coincidencia;
    if (coincidencia = linea.match(/^(\w+)=(.*)$/)) {
      seccion[coincidencia[1]] = coincidencia[2];
    } else if (coincidencia = linea.match(/^\[(.*)\]$/)) {
      seccion = resultado[coincidencia[1]] = {};
    } else if (!/^\s*(;.*)?$/.test(linea)) {
      throw new Error("Linea '" + linea + "' no es valida.");
    }
  });
  return resultado;
}

console.log(analizarINI(`
nombre=Vasilis
[direccion]
ciudad=Tessaloniki`));
// → {nombre: "Vasilis", direccion: {ciudad: "Tessaloniki"}}
```

{{index "parseINI function", parsing}}

El código pasa por las líneas del archivo y crea un objeto.
Las propiedades en la parte superior se almacenan directamente en ese objeto,
mientras que las propiedades que se encuentran en las secciones se almacenan
en un objeto de sección separado. La vinculación `sección` apunta al
objeto para la sección actual.

Hay dos tipos de de líneas significativas—encabezados de seccion o lineas de
propiedades. Cuando una línea es una propiedad regular, esta se almacena en la
sección actual. Cuando se trata de un encabezado de sección, se crea un nuevo
objeto de sección, y `seccion` se configura para apuntar a él.

{{index "caret character", "dollar sign", boundary}}

Nota el uso recurrente de `^` y `$` para asegurarse de que la expresión
coincida con toda la línea, no solo con parte de ella. Dejando afuera
estos resultados en código que funciona principalmente, pero que se comporta
de forma extraña para algunas entradas, lo que puede ser un error difícil de
rastrear.

{{index "if keyword", assignment, "= operator"}}

El patrón `if (coincidencia = string.match (...))` es similar al truco
de usar una asignación como condición para `while`. A menudo no estas
seguro de que tu llamada a `match` tendrá éxito, para que puedas acceder al
objeto resultante solo dentro de una declaración `if` que pruebe esto. Para
no romper la agradable cadena de las formas `else if`, asignamos el resultado
de la coincidencia a una vinculación e inmediatamente usamos esa asignación
como la prueba para la declaración `if`.

Si una línea no es un encabezado de sección o una propiedad, la función verifica
si es un comentario o una línea vacía usando la expresión
`/^\s*(;.*)?$/`. Ves cómo funciona? La parte entre los ((paréntesis))
coincidirá con los comentarios, y el `?` asegura que también
coincida con líneas que contengan solo espacios en blanco. Cuando una línea
no coincida con cualquiera de las formas esperadas, la función arroja una
excepción.

## Caracteres internacionales

{{index internationalization, Unicode, ["regular expression", internationalization]}}

Debido a la simplista implementación inicial de JavaScript y al hecho de
que este enfoque simplista fue luego establecido en piedra como comportamiento
((estándar)), las expresiones regulares de JavaScript son bastante tontas
acerca de los caracteres que no aparecen en el idioma inglés. Por ejemplo, en
cuanto a las expresiones regulares de JavaScript, una "((palabra
caracter))" es solo uno de los 26 caracteres en el alfabeto latino
(mayúsculas o minúsculas), dígitos decimales, y, por alguna razón, el
carácter de guion bajo. Cosas como _é_ o _β_, que definitivamente
son caracteres de palabras, no coincidirán con `\w` (y _si_
coincidiran con `\W` mayúscula, la categoría no-palabra).

{{index whitespace}}

Por un extraño accidente histórico, `\s` (espacio en blanco) no tiene este
problema y coincide con todos los caracteres que el estándar Unicode considera
espacios en blanco, incluyendo cosas como el (espacio de no separación) y el
((Separador de vocales Mongol)).

Otro problema es que, de forma predeterminada, las expresiones regulares
funcionan en unidades del código, como se discute en el
[Capítulo 5](orden_superior#unidades_del_codigo), no en
caracteres reales. Esto significa que los caracteres que estan compustos
de dos unidades de código se comportan de manera extraña.

```
console.log(/🍎{3}/.test("🍎🍎🍎"));
// → false
console.log(/<.>/.test("<🌹>"));
// → false
console.log(/<.>/u.test("<🌹>"));
// → true
```

El problema es que la 🍎 en la primera línea se trata como dos unidades de
código, y la parte `{3}` se aplica solo a la segunda. Del mismo modo,
el punto coincide con una sola unidad de código, no con las dos que componen
al ((emoji)) de rosa.

Debe agregar una opción `u` (para ((Unicode))) a tu expresión regular
para hacerla tratar a tales caracteres apropiadamente. El comportamiento
incorrecto sigue siendo el predeterminado, desafortunadamente, porque cambiarlo
podría causar problemas en código ya existente que depende de él.

{{index "character category", [Unicode, property]}}

Aunque esto solo se acaba de estandarizar y aun no es, al momento de escribir
este libro, ampliamente compatible con muchs nabegadores, es posible usar
`\p` en una expresión regular (que debe tener habilitada la opción Unicode)
para que coincida con todos los caracteres a los que el estándar Unicode
lis asigna una propiedad determinada.

```{test: never}
console.log(/\p{Script=Greek}/u.test("α"));
// → true
console.log(/\p{Script=Arabic}/u.test("α"));
// → false
console.log(/\p{Alphabetic}/u.test("α"));
// → true
console.log(/\p{Alphabetic}/u.test("!"));
// → false
```

Unicode define una cantidad de propiedades útiles, aunque encontrar la
que necesitas puede no ser siempre trivial. Puedes usar la notación
`\p{Property=Value}` para hacer coincidir con cualquier carácter que tenga el
valor dado para esa propiedad. Si el nombre de la propiedad se deja afuera,
como en `\p{Name}`, se asume que el nombre es una propiedad binaria como
`Alfabético` o una categoría como `Número`.

{{id summary_regexp}}

## Resumen

Las expresiones regulares son objetos que representan patrones en strings.
Ellas usan su propio lenguaje para expresar estos patrones.

{{table {cols: [1, 5]}}}

| `/abc/`     | Una secuencia de caracteres
| `/[abc]/`   | Cualquier caracter de un conjunto de caracteres
| `/[^abc]/`  | Cualquier carácter que _no_ este en un conjunto de caracteres
| `/[0-9]/`   | Cualquier caracter en un rango de caracteres
| `/x+/`      | Una o más ocurrencias del patrón `x`
| `/x+?/`     | Una o más ocurrencias, no codiciosas
| `/x*/`      | Cero o más ocurrencias
| `/x?/`      | Cero o una ocurrencia
| `/x{2,4}/`  | De dos a cuatro ocurrencias
| `/(abc)/`   | Un grupo
| `/a|b|c/`   | Cualquiera de varios patrones
| `/\d/`      | Cualquier caracter de digito
| `/\w/`      | Un caracter alfanumérico ("carácter de palabra")
| `/\s/`      | Cualquier caracter de espacio en blanco
| `/./`       | Cualquier caracter excepto líneas nuevas
| `/\b/`      | Un límite de palabra
| `/^/`       | Inicio de entrada
| `/$/`       | Fin de la entrada

Una expresión regular tiene un método `test` para probar si una determinada
string coincide cn ella. También tiene un método `exec` que, cuando una
coincidencia es encontrada, retorna un array que contiene todos los grupos
que coincidieron. Tal array tiene una propiedad `index` que indica en
dónde comenzó la coincidencia.

Los strings tienen un método `match` para coincidirlas con una expresión
regular y un método `search` para buscar por una, retornando solo la
posición inicial de la coincidencia. Su método `replace` puede reemplazar
coincidencias de un patrón con un string o función de reemplazo.

Las expresiones regulares pueden tener opciones, que se escriben después de la
barra que cierra la expresión. La opción `i` hace que la coincidencia no
distinga entre mayúsculas y minúsculas. La opción `g` hace que la expresión sea
_global_, que, entre otras cosas, hace que el método `replace` reemplace todas
las instancias en lugar de solo la primera. La opción `y` la hace adhesivo,
lo que significa que hará que no busque con anticipación y omita la parte del
string cuando busque una coincidencia. La opción `u` activa el modo Unicode,
lo que soluciona varios problemas alrededor del manejo de caracteres que
toman dos unidades de código.

Las expresiones regulares son ((herramientas)) afiladas con un manejo incómodo.
Ellas simplifican algunas tareas enormemente, pero pueden volverse inmanejables
rápidamente cuando se aplican a problemas complejos. Parte de saber cómo
usarlas es resistiendo el impulso de tratar de calzar cosas que no pueden ser
expresadas limpiamente en ellas.

## Ejercicios

{{index debugging, bug}}

Es casi inevitable que, durante el curso de trabajar en estos
ejercicios, te sentiras confundido y frustrado por el ((comportamiento))
inexplicable de alguna regular expresión. A veces ayuda ingresar
tu expresión en una herramienta en línea como
[_debuggex.com_](https://www.debuggex.com/) para ver si
su visualización corresponde a lo que pretendías y a ((experimentar))
con la forma en que responde a varios strings de entrada.

### Golf Regexp

{{index "program size", "code golf", "regexp golf (exercise)"}}

_Golf de Codigo_ es un término usado para el juego de intentar expresar un
programa particular con el menor número de caracteres posible. Similarmente,
_Golf de Regexp_ es la práctica de escribir una expresión regular tan pequeña
como sea posible para que coincida con un patrón dado, y _sólo_ con ese patrón.

{{index boundary, matching}}

Para cada uno de los siguientes elementos, escribe una ((expresión regular)) para
probar si alguna de las substrings dadas ocurre en un string. La
expresión regular debe coincidir solo con strings que contengan una de las
substrings descritas. No te preocupes por los límites de palabras a menos que
sean explícitamente mencionados. Cuando tu expresión funcione, ve si puedes
hacerla más pequeña.

 1. _car_ y _cat_
 2. _pop_ y _prop_
 3. _ferret_, _ferry_, y _ferrari_
 4. Cualquier palabra que termine _ious_
 5. Un carácter de espacio en blanco seguido de un punto, coma, dos puntos o punto y coma
 6. Una palabra con mas de seis letras
 7. Una palabra sin la letra _e_ (o _E_)

Consulta la tabla en el [resumen del capítulo](regexp#summary_regexp) para
ayudarte. Pruebe cada solución con algunos strings de prueba.

{{if interactive
```
// Llena con las expresiones regulares

verificar(/.../,
       ["my car", "bad cats"],
       ["camper", "high art"]);

verificar(/.../,
       ["pop culture", "mad props"],
       ["plop", "prrrop"]);

verificar(/.../,
       ["ferret", "ferry", "ferrari"],
       ["ferrum", "transfer A"]);

verificar(/.../,
       ["how delicious", "spacious room"],
       ["ruinous", "consciousness"]);

verificar(/.../,
       ["bad punctuation ."],
       ["escape the period"]);

verificar(/.../,
       ["hottentottententen"],
       ["no", "hotten totten tenten"]);

verificar(/.../,
       ["red platypus", "wobbling nest"],
       ["earth bed", "learning ape", "BEET"]);


function verificar(regexp, si, no) {
  // Ignora ejercicios sin terminar
  if (regexp.source == "...") return;
  for (let str of si) if (!regexp.test(str)) {
    console.log(`Fallo al coincidir '${str}'`);
  }
  for (let str of no) if (regexp.test(str)) {
    console.log(`Coincidencia inesperada para '${str}'`);
  }
}
```

if}}

### Estilo entre comillas

{{index "quoting style (exercise)", "single-quote character", "double-quote character"}}

Imagina que has escrito una historia y has utilizado ((comillas))s simples
en todas partes para marcar piezas de diálogo. Ahora quieres reemplazar todas
las comillas de diálogo con comillas dobles, pero manteniendo las comillas
simples usadas en contracciones como _aren't_.

{{index "replace method"}}

Piensa en un patrón que distinga de estos dos tipos de uso de citas y crea
una llamada al método `replace` que haga el reemplazo apropiado.

{{if interactive
```{test: no}
let texto = "'I'm the cook,' he said, 'it's my job.'";
// Cambia esta llamada
console.log(texto.replace(/A/g, "B"));
// → "I'm the cook," he said, "it's my job."
```
if}}

{{hint

{{index "quoting style (exercise)", boundary}}

La solución más obvia es solo reemplazar las citas con una palabra no
personaje en al menos un lado. Algo como `/\W'|'\W/`. Pero
también debes tener en cuenta el inicio y el final de la línea.

{{index grouping, "replace method"}}

Además, debes asegurarte de que el reemplazo también incluya los
caracteres que coincidieron con el patrón `\W` para que estos no sean
dejados. Esto se puede hacer envolviéndolos en ((paréntesis)) e
incluyendo sus grupos en la cadena de reemplazo (`$1`,`$2`). Los grupos
que no están emparejados serán reemplazados por nada.

hint}}

### Números otra vez

{{index sign, "fractional number", syntax, minus, "plus character", exponent, "scientific notation", "period character"}}

Escribe una expresión que coincida solo con el estilo de ((número))s en
JavaScript. Esta debe admitir un signo opcional menos _o_ más delante del número,
el punto decimal, y la notación de exponente—`5e-3` o `1E10`— nuevamente con un
signo opcional en frente del exponente. También ten en cuenta que no es
necesario que hayan dígitos delante o detrás del punto, pero el
el número no puede ser solo un punto. Es decir, `.5` y `5.` son numeros válidos
de JavaScript, pero solo un punto _no_ lo es.

{{if interactive
```{test: no}
// Completa esta expresión regular.
let numero = /^...$/;

// Pruebas:
for (let str of ["1", "-1", "+15", "1.55", ".5", "5.",
                 "1.3e2", "1E-4", "1e+12"]) {
  if (!numero.test(str)) {
    console.log(`Fallo al coincidir '${str}'`);
  }
}
for (let str of ["1a", "+-1", "1.2.3", "1+1", "1e4.5",
                 ".5.", "1f5", "."]) {
  if (numero.test(str)) {
    console.log(`Incorrectamente acepto '${str}'`);
  }
}
```

if}}

{{hint

{{index ["regular expression", escaping], "backslash character"}}

Primero, no te olvides de la barra invertida delante del punto.

Coincidir el ((signo)) opcional delante de el ((número)), así como
delante del ((exponente)), se puede hacer con `[+\-]?` o `(\+|-|)`
(más, menos o nada).

{{index "pipe character"}}

La parte más complicada del ejercicio es el problema hacer coincidir
ambos `"5."` y `".5"` sin también coincidir coincidir con `"."`. Para esto,
una buena solución es usar el operador `|` para separar los dos casos—ya sea
uno o más dígitos seguidos opcionalmente por un punto y cero o más
dígitos _o_ un punto seguido de uno o más dígitos.

{{index exponent, "case sensitivity", ["regular expression", flags]}}

Finalmente, para hacer que la _e_ pueda ser mayuscula o minuscula,
agrega una opción `i` a la expresión regular o usa `[eE]`.

hint}}
