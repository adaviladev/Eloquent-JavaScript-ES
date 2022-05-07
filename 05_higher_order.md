{{meta {load_files: ["code/scripts.js", "code/chapter/05_higher_order.js", "code/intro.js"], zip: "node/html"}}}

# Funciones de Orden Superior

{{if interactive

{{quote {author: "Master Yuan-Ma", title: "The Book of Programming", chapter: true}

Tzu-li y Tzu-ssu estaban jactándose del tamaño de sus ultimos
programas. 'Doscientas mil líneas', dijo Tzu-li, 'sin contar los
comentarios!' Tzu-ssu respondió, 'Pssh, el mío tiene casi un *millón*
de líneas ya.' El Maestro Yuan-Ma dijo, 'Mi mejor programa tiene quinientas
líneas.' Al escuchar esto, Tzu-li y Tzu-ssu fueron iluminados.

quote}}

if}}

{{quote {author: "C.A.R. Hoare", title: "1980 ACM Turing Award Lecture", chapter: true}

{{index "Hoare, C.A.R."}}

Hay dos formas de construir un diseño de software: Una forma es
hacerlo tan simple de manera que no hayan deficiencias obvias, y
la otra es hacerlo tan complicado de manera que
obviamente no hayan deficiencias.

quote}}

{{figure {url: "img/chapter_picture_5.jpg", alt: "Letras de diferentes idiomas", chapter: true}}}

{{index "program size"}}

Un programa grande es un programa costoso, y no solo por el tiempo que
se necesita para construirlo. El tamaño casi siempre involucra ((complejidad)),
y la complejidad confunde a los programadores. A su vez, los programadores
confundidos, introducen errores en los programas. Un programa grande entonces
proporciona de mucho espacio para que estos bugs se oculten, haciéndolos
difíciles de encontrar.

{{index "summing example"}}

Volvamos rapidamente a los dos últimos programas de ejemplo en la
introducción. El primero es auto-contenido y solo tiene seis líneas de largo:

```
let total = 0, cuenta = 1;
while (cuenta <= 10) {
  total += cuenta;
  cuenta += 1;
}
console.log(total);
```

El segundo depende de dos funciones externas y tiene una línea de longitud:

```
console.log(suma(rango(1, 10)));
```

Cuál es más probable que contenga un bug?

{{index "program size"}}

Si contamos el tamaño de las definiciones de `suma` y `rango`,
el segundo programa también es grande—incluso puede que sea más grande que
el primero. Pero aún así, argumentaria que es más probable que sea correcto.

{{index abstraction, "domain-specific language"}}

Es más probable que sea correcto porque la solución se expresa en un
((vocabulario)) que corresponde al problema que se está resolviendo. Sumar un
rango de números no se trata acerca de ciclos y contadores. Se trata acerca
de rangos y sumas.

Las definiciones de este vocabulario (las funciones `suma` y `rango`)
seguirán involucrando ciclos, contadores y otros detalles incidentales. Pero
ya que expresan conceptos más simples que el programa como un conjunto,
son más fáciles de realizar correctamente.

## Abstracción

En el contexto de la programación, estos tipos de vocabularios suelen ser
llamados _((abstraccione))s_. Las abstracciones esconden detalles y nos dan la
capacidad de hablar acerca de los problemas a un nivel superior
(o más abstracto).

{{index "recipe analogy", "pea soup"}}

Como una analogía, compara estas dos recetas de sopa de guisantes:

{{quote

Coloque 1 taza de guisantes secos por persona en un recipiente. Agregue agua hasta
que los guisantes esten bien cubiertos. Deje los guisantes en agua durante al menos 12
horas. Saque los guisantes del agua y pongalos en una cacerola para cocinar.
Agregue 4 tazas de agua por persona. Cubra la sartén y mantenga los guisantes
hirviendo a fuego lento durante dos horas. Tome media cebolla por persona. Cortela en
piezas con un cuchillo. Agréguela a los guisantes. Tome un tallo de apio por
persona. Cortelo en pedazos con un cuchillo. Agréguelo a los guisantes. Tome una
zanahoria por persona. Cortela en pedazos. Con un cuchillo! Agregarla a los
guisantes. Cocine por 10 minutos más.

quote}}

Y la segunda receta:

{{quote

Por persona: 1 taza de guisantes secos, media cebolla picada, un tallo de
apio y una zanahoria.

Remoje los guisantes durante 12 horas. Cocine a fuego lento durante
2 horas en 4 tazas de agua (por persona). Picar y agregar verduras.
Cocine por 10 minutos más.

quote}}

{{index vocabulary}}

La segunda es más corta y fácil de interpretar. Pero necesitas
entender algunas palabras más relacionadas a la cocina—_remojar_,
_cocinar a fuego lento_, _picar_, y, supongo, _verduras_.

Cuando programamos, no podemos confiar en que todas las palabras que necesitaremos
estaran esperando por nosotros en el diccionario. Por lo tanto, puedes caer
en el patrón de la primera receta—resolviendo los pasos precisos que debe
realizar la computadora, uno por uno, ciego a los conceptos de orden
superior que estos expresan.

{{index abstraction}}

En la programación, es una habilidad útil, darse cuenta cuando estás trabajando
en un nivel de abstracción demasiado bajo.

## Abstrayendo la repetición

{{index array}}

Las funciones simples, como las hemos visto hasta ahora,
son una buena forma de construir abstracciones. Pero a veces se quedan cortas.

{{index "for loop"}}

Es común que un programa haga algo una determinada cantidad de veces.
Puedes escribir un ((ciclo)) `for` para eso, de esta manera:

```
for (let i = 0; i < 10; i++) {
  console.log(i);
}
```

Podemos abstraer "hacer algo _N_ veces" como una función? Bueno, es
fácil escribir una función que llame a `console.log` _N_
cantidad de veces.

```
function repetirLog(n) {
  for (let i = 0; i < n; i++) {
    console.log(i);
  }
}
```

{{index [function, "higher-order"], loop, [array, traversal], [function, "as value"]}}

{{indexsee "higher-order function", "function, higher-order"}}

Pero, y si queremos hacer algo más que loggear los números?
Ya que "hacer algo" se puede representar como una función y que las funciones
solo son valores, podemos pasar nuestra acción como un valor de función.

```{includeCode: "top_lines: 5"}
function repetir(n, accion) {
  for (let i = 0; i < n; i++) {
    accion(i);
  }
}

repetir(3, console.log);
// → 0
// → 1
// → 2
```

No es necesario que le pases una función predefinida a `repetir`. A menudo,
desearas crear un valor de función al momento en su lugar.

```
let etiquetas = [];
repetir(5, i => {
  etiquetas.push(`Unidad ${i + 1}`);
});
console.log(etiquetas);
// → ["Unidad 1", "Unidad 2", "Unidad 3", "Unidad 4", "Unidad 5"]
```

{{index "loop body", "curly braces"}}

Esto está estructurado un poco como un ciclo `for`—primero describe el
tipo de ciclo, y luego provee un cuerpo. Sin embargo, el cuerpo ahora está escrito
como un valor de función, que está envuelto en el ((paréntesis)) de la
llamada a `repetir`. Por eso es que tiene que cerrarse con el corchete de cierre
_y_ paréntesis de cierre. En casos como este ejemplo, donde el
cuerpo es una expresión pequeña y única, podrias también omitir las
llaves y escribir el ciclo en una sola línea.

## Funciones de orden superior

{{index [function, "higher-order"], [function, "as value"]}}

Las funciones que operan en otras funciones, ya sea tomándolas como
argumentos o retornandolas, se denominan _funciones de orden superior_.
Como ya hemos visto que las funciones son valores regulares, no existe
nada particularmente notable sobre el hecho de que tales funciones
existen. El término proviene de las ((matemáticas)), donde la distinción
entre funciones y otros valores se toma más en serio.

{{index abstraction}}

Las funciones de orden superior nos permiten abstraer sobre _acciones_,
no solo sobre valores. Estas vienen en varias formas. Por ejemplo, puedes tener
funciones que crean nuevas funciones.

```
function mayorQue(n) {
  return m => m > n;
}
let mayorQue10 = mayorQue(10);
console.log(mayorQue10(11));
// → true
```

Y puedes tener funciones que cambien otras funciones.

```
function ruidosa(funcion) {
  return (...argumentos) => {
    console.log("llamando con", argumentos);
    let resultado = funcion(...argumentos);
    console.log("llamada con", argumentos, ", retorno", resultado);
    return resultado;
  };
}
ruidosa(Math.min)(3, 2, 1);
// → llamando con [3, 2, 1]
// → llamada con [3, 2, 1] , retorno 1
```

Incluso puedes escribir funciones que proporcionen nuevos tipos de
((flujo de control)).

```
function aMenosQue(prueba, entonces) {
  if (!prueba) entonces();
}

repetir(3, n => {
  aMenosQue(n % 2 == 1, () => {
    console.log(n, "es par");
  });
});
// → 0 es par
// → 2 es par
```

{{index [array, methods], [array, iteration], "forEach method"}}

Hay un método de array incorporado, `forEach` que proporciona algo
como un ciclo `for`/`of` como una función de orden superior.

```
["A", "B"].forEach(letra => console.log(letra));
// → A
// → B
```

## Conjunto de datos de códigos

Un área donde brillan las funciones de orden superior es en el
procesamiento de datos. Para procesar datos, necesitaremos algunos datos reales.
Este capítulo usara un ((conjunto de datos)) acerca de
códigos—((sistema de escritura))s como Latin, Cirílico, o Arábico.

Recuerdas ((Unicode)) del [Capítulo 1](valores#unicode), el sistema que
asigna un número a cada carácter en el lenguaje escrito. La mayoría de estos
carácteres están asociados a un código específico. El estandar
contiene 140 codigos diferentes—81 de los cuales todavía están en uso hoy, y 59
que son históricos.

Aunque solo puedo leer con fluidez los caracteres en Latin, aprecio el
hecho de que las personas estan escribiendo textos en al menos 80 diferentes
sistemas de escritura, muchos de los cuales ni siquiera reconocería. Por ejemplo,
aquí está una muestra de escritura a mano en ((Tamil)).

{{figure {url: "img/tamil.png", alt: "Tamil handwriting"}}}

{{index "SCRIPTS data set"}}

El ((conjunto de datos)) de ejemplo contiene algunos piezas de información
acerca de los 140 codigos definidos en Unicode. Este esta disponible en la [caja de arena](https://eloquentjavascript.net/code#5) para este capítulo [
([_eloquentjavascript.net/code#5_](https://eloquentjavascript.net/code#5))]{if
book} como la vinculación `SCRIPTS`. La vinculación contiene un array de
objetos, cada uno de los cuales describe un codigo.


```{lang: "application/json"}
{
  name: "Coptic",
  ranges: [[994, 1008], [11392, 11508], [11513, 11520]],
  direction: "ltr",
  year: -200,
  living: false,
  link: "https://en.wikipedia.org/wiki/Coptic_alphabet"
}
```

Tal objeto te dice el nombre del codigo, los rangos de Unicode
asignados a él, la dirección en la que está escrito, la
tiempo de origen (aproximado), si todavía está en uso, y un enlace a
más información. La dirección en la que esta escrito puede ser
`"ltr"` (left-to-right) para izquierda a derecha, `"rtl"` (right-to-left)
para derecha a izquierda (la forma en que se escriben los textos en árabe
y en hebreo), o `"ttb"` (top-to-bottom) para de arriba a abajo
(como con la escritura de Mongolia).

{{index "slice method"}}

La propiedad `ranges` contiene un array de ((rango))s de caracteres Unicode,
cada uno de los cuales es un array de dos elementos que contiene límites inferior
y superior. Se asignan los códigos de caracteres dentro de estos rangos
al codigo. El ((limite)) más bajo es inclusivo (el código 994 es un carácter Copto)
y el límite superior es no-inclusivo (el código 1008 no lo es).

## Filtrando arrays

{{index [array, methods], [array, filtering], "filter method", [function, "higher-order"], "predicate function"}}

Para encontrar los codigos en el conjunto de datos que todavía están en uso,
la siguiente función podría ser útil. Filtra hacia afuera los elementos en un
array que no pasen una prueba:

```
function filtrar(array, prueba) {
  let pasaron = [];
  for (let elemento of array) {
    if (prueba(elemento)) {
      pasaron.push(elemento);
    }
  }
  return pasaron;
}

console.log(filtrar(SCRIPTS, codigo => codigo.living));
// → [{name: "Adlam", …}, …]
```

{{index [function, "as value"], [function, application]}}

La función usa el argumento llamado `prueba`, un valor de función, para llenar
una "brecha" en el cálculo—el proceso de decidir qué elementos recolectar.

{{index "filter method", "pure function", "side effect"}}

Observa cómo la función `filtrar`, en lugar de eliminar elementos del
array existente, crea un nuevo array solo con los elementos que pasan
la prueba. Esta función es _pura_. No modifica el array que se le es
dado.

Al igual que `forEach`, `filtrar` es un método de array ((estándar)), este
esta incorporado como `filter`.
El ejemplo definió la función solo para mostrar lo que hace internamente.
A partir de ahora, la usaremos así en su lugar:

```
console.log(SCRIPTS.filter(codigo => codigo.direction == "ttb"));
// → [{name: "Mongolian", …}, …]
```

{{id map}}

## Transformando con map

{{index [array, methods], "map method"}}

Digamos que tenemos un array de objetos que representan codigos, producidos al
filtrar el array `SCRIPTS` de alguna manera. Pero queremos un array de nombres,
que es más fácil de inspeccionar

{{index [function, "higher-order"]}}

El método `map` ("mapear") transforma un array al aplicar una función a todos
sus elementos y construir un nuevo array a partir de los valores retornados.
El nuevo array tendrá la misma longitud que el array de entrada, pero su
contenido ha sido _mapeado_ a una nueva forma en base a la función.

```
function map(array, transformar) {
  let mapeados = [];
  for (let elemento of array) {
    mapeados.push(transformar(elemento));
  }
  return mapeados;
}

let codigosDerechaAIzquierda = SCRIPTS.filter(codigo => codigo.direction == "rtl");
console.log(map(codigosDerechaAIzquierda, codigo => codigo.name));
// → ["Adlam", "Arabic", "Imperial Aramaic", …]
```

Al igual que `forEach` y `filter`, `map` es un método de array estándar.

## Resumiendo con reduce

{{index [array, methods], "summing example", "reduce method"}}

Otra cosa común que hacer con arrays es calcular un valor único a partir
de ellos. Nuestro ejemplo recurrente, sumar una colección de números, es
una instancia de esto. Otro ejemplo sería encontrar el codigo con
la mayor cantidad de caracteres.

{{indexsee "fold", "reduce method"}}

{{index [function, "higher-order"], "reduce method"}}

La operación de orden superior que representa este patrón se llama
_reduce_ ("reducir")—a veces también llamada _fold_ ("doblar").
Esta construye un valor al repetidamente tomar un solo elemento del
array y combinándolo con el valor actual. Al sumar números, comenzarías con el
número cero y, para cada elemento, agregas eso a la suma.

Los parámetros para `reduce` son, además del array, una función de combinación
y un valor de inicio. Esta función es un poco menos sencilla que `filter`
y `map`, así que mira atentamente:

```
function reduce(array, combinar, inicio) {
  let actual = inicio;
  for (let elemento of array) {
    actual = combinar(actual, elemento);
  }
  return actual;
}

console.log(reduce([1, 2, 3, 4], (a, b) => a + b, 0));
// → 10
```

{{index "reduce method", "SCRIPTS data set"}}

El método de array estándar `reduce`, que por supuesto corresponde a
esta función tiene una mayor comodidad. Si tu array contiene
al menos un elemento, tienes permitido omitir el argumento `inicio`.
El método tomará el primer elemento del array como su valor de inicio
y comienza a reducir a partir del segundo elemento.

```
console.log([1, 2, 3, 4].reduce((a, b) => a + b));
// → 10
```

{{index maximum, "characterCount function"}}

Para usar `reduce` (dos veces) para encontrar el codigo con la mayor
cantidad de caracteres, podemos escribir algo como esto:

```
function cuentaDeCaracteres(codigo) {
  return codigo.ranges.reduce((cuenta, [desde, hasta]) => {
    return cuenta + (hasta - desde);
  }, 0);
}

console.log(SCRIPTS.reduce((a, b) => {
  return cuentaDeCaracteres(a) < cuentaDeCaracteres(b) ? b : a;
}));
// → {name: "Han", …}
```

La función `cuentaDeCaracteres` reduce los rangos asignados a un codigo
sumando sus tamaños. Ten en cuenta el uso de la desestructuración en el parámetro
lista de la función reductora. La segunda llamada a `reduce` luego usa
esto para encontrar el codigo más grande al comparar repetidamente dos scripts
y retornando el más grande.

El codigo Han tiene más de 89,000 caracteres asignados en el
Estándar Unicode, por lo que es, por mucho, el mayor sistema de escritura en el
conjunto de datos. Han es un codigo (a veces) usado para texto chino, japonés y
coreano. Esos idiomas comparten muchos caracteres, aunque
tienden a escribirlos de manera diferente. El consorcio Unicode (con sede en EE.UU.)
decidió tratarlos como un único sistema de escritura para ahorrar
códigos de caracteres. Esto se llama _unificación Han_ y aún enoja bastante a
algunas personas.

## Composabilidad

{{index loop, maximum}}

Considera cómo habríamos escrito el ejemplo anterior (encontrar el
código más grande) sin funciones de orden superior. El código no es
mucho peor.

```{test: no}
let mayor = null;
for (let codigo of SCRIPTS) {
  if (mayor == null ||
      cuentaDeCaracteres(mayor) < cuentaDeCaracteres(codigo)) {
    mayor = codigo;
  }
}
console.log(mayor);
// → {name: "Han", …}
```

Hay algunos vinculaciones más, y el programa tiene cuatro líneas
más. Pero todavía es bastante legible.

{{index "average function", composability, [function, "higher-order"], "filter method", "map method", "reduce method"}}

{{id average_function}}

Las funciones de orden superior comienzan a brillar cuando necesitas _componer_
operaciones. Como ejemplo, vamos a escribir código que encuentre el
año de origen promedio para los codigos vivos y muertos en el conjunto de datos.

```
function promedio(array) {
  return array.reduce((a, b) => a + b) / array.length;
}

console.log(Math.round(promedio(
  SCRIPTS.filter(codigo => codigo.living).map(codigo => codigo.year))));
// → 1185
console.log(Math.round(promedio(
  SCRIPTS.filter(codigo => !codigo.living).map(codigo => codigo.year))));
// → 209
```

Entonces, los codigos muertos en Unicode son, en promedio, más antiguos que
los vivos. Esta no es una estadística terriblemente significativa o sorprendente.
Pero espero que aceptes que el código utilizado para calcularlo no es difícil
de leer. Puedes verlo como una tubería: comenzamos con todos los codigos, filtramos
los vivos (o muertos), tomamos los años de aquellos, los promediamos,
y redondeamos el resultado.

Definitivamente también podrías haber escribir este codigo como un gran ((ciclo)).

```
let total = 0, cuenta = 0;
for (let codigo of SCRIPTS) {
  if (codigo.living) {
    total += codigo.year;
    cuenta += 1;
  }
}
console.log(Math.round(total / cuenta));
// → 1185
```

Pero es más difícil de ver qué se está calculando y cómo. Y ya que
los resultados intermedios no se representan como valores coherentes, sería
mucho más trabajo extraer algo así como `promedio` en una función aparte.

{{index efficiency}}

En términos de lo que la computadora realmente está haciendo, estos dos enfoques
también son bastante diferentes. El primero creará nuevos ((arrays)) al
ejecutar `filter` y `map`, mientras que el segundo solo computa algunos
números, haciendo menos trabajo. Por lo general, puedes permitirte el
enfoque legible, pero si estás procesando arrays enormes, y haciendolo muchas
veces, el estilo menos abstracto podría ser mejor debido a la velocidad extra.

## Strings y códigos de caracteres

{{index "SCRIPTS data set"}}

Un uso del conjunto de datos sería averiguar qué código esta usando una
pieza de texto. Veamos un programa que hace esto.

Recuerda que cada codigo tiene un array de rangos para los códigos de caracteres
asociados a el. Entonces, dado un código de carácter, podríamos usar una
función como esta para encontrar el codigo correspondiente (si lo hay):

{{index "some method", "predicate function", [array, methods]}}

```{includeCode: strip_log}
function codigoCaracter(codigo_caracter) {
  for (let codigo of SCRIPTS) {
    if (codigo.ranges.some(([desde, hasta]) => {
      return codigo_caracter >= desde && codigo_caracter < hasta;
    })) {
      return codigo;
    }
  }
  return null;
}

console.log(codigoCaracter(121));
// → {name: "Latin", …}
```

El método `some` ("alguno") es otra función de orden superior. Toma una función
de prueba y te dice si esa función retorna verdadero para cualquiera de los
elementos en el array.

{{id code_units}}

Pero cómo obtenemos los códigos de los caracteres en un string?

En el [Capítulo 1](valores) mencioné que los ((strings)) de JavaScript estan
codificados como una secuencia de números de 16 bits. Estos se llaman
_((unidades de código))_. Inicialmente se suponía que un código de ((carácter))
((Unicode)) encajara dentro de esa unidad (lo que da un poco más de 65,000
caracteres). Cuando quedó claro que esto no seria suficiente, muchas 
personas se resistieron a la necesidad de usar más memoria por carácter.
Para apaciguar estas preocupaciones, ((UTF-16)), el formato utilizado por los
strings de JavaScript, fue inventado. Este describe la mayoría de los caracteres
mas comunes usando una sola unidad de código de 16 bits, pero usa un par de
dos de esas unidades para otros caracteres.

{{index error}}

Al dia de hoy UTF-16 generalmente se considera como una mala idea. Parece casi
intencionalmente diseñado para invitar a errores. Es fácil escribir programas
que pretenden que las unidades de código y caracteres son la misma cosa. Y si tu
lenguaje no usa caracteres de dos unidades, esto parecerá funcionar
simplemente bien. Pero tan pronto como alguien intente usar dicho programa con
algunos menos comunes ((caracteres chinos)), este se rompe. Afortunadamente, con
la llegada del ((emoji)), todo el mundo ha empezado a usar caracteres de
dos unidades, y la carga de lidiar con tales problemas esta
bastante mejor distribuida.

{{index [string, length], [string, indexing], "charCodeAt method"}}

Desafortunadamente, las operaciones obvias con strings de JavaScript, como
obtener su longitud a través de la propiedad `length` y acceder a su
contenido usando corchetes, trata solo con unidades de código.

```{test: no}
// Dos caracteres emoji, caballo y zapato
let caballoZapato = "🐴👟";
console.log(caballoZapato.length);
// → 4
console.log(caballoZapato[0]);
// → ((Medio-carácter inválido))
console.log(caballoZapato.charCodeAt(0));
// → 55357 (Código del medio-carácter)
console.log(caballoZapato.codePointAt(0));
// → 128052 (Código real para emoji de caballo)
```

{{index "codePointAt method"}}

El método `charCodeAt` de JavaScript te da una unidad de código, no un
código de carácter completo. El método `codePointAt`, añadido despues, si da un
carácter completo de Unicode. Entonces podríamos usarlo para obtener
caracteres de un string. Pero el argumento pasado a `codePointAt` sigue
siendo un índice en la secuencia de unidades de código. Entonces, para hacer
un ciclo a traves de todos los caracteres en un string, todavía tendríamos que
lidiar con la cuestión de si un carácter ocupa una o dos unidades de código.

{{index "for/of loop", character}}

En el [capítulo anterior](datos#for_of_loop), mencioné que el ciclo
`for`/`of` también se puede usar en strings. Como `codePointAt`, este
tipo de ciclo se introdujo en un momento en que las personas eran muy
conscientes de los problemas con UTF-16. Cuando lo usas para hacer un ciclo
a traves de un string, te da caracteres reales, no unidades de código.

```
let dragonRosa = "🐉🌹";
for (let caracter of dragonRosa) {
  console.log(caracter);
}
// → 🐉
// → 🌹
```

Si tienes un caracter (que será un string de unidades de uno o dos códigos),
puedes usar `codePointAt(0)` para obtener su código.

## Reconociendo texto

{{index "SCRIPTS data set", "countBy function", array}}

Tenemos una función `codigoCaracter` y una forma de correctamente hacer un
ciclo a traves de caracteres. El siguiente paso sería contar los caracteres
que pertenecen a cada codigo. La siguiente abstracción de conteo será útil
para eso:

```{includeCode: strip_log}
function contarPor(elementos, nombreGrupo) {
  let cuentas = [];
  for (let elemento of elementos) {
    let nombre = nombreGrupo(elemento);
    let conocido = cuentas.findIndex(c => c.nombre == nombre);
    if (conocido == -1) {
      cuentas.push({nombre, cuenta: 1});
    } else {
      cuentas[conocido].cuenta++;
    }
  }
  return cuentas;
}

console.log(contarPor([1, 2, 3, 4, 5], n => n > 2));
// → [{nombre: false, cuenta: 2}, {nombre: true, cuenta: 3}]
```

La función `contarPor` espera una colección (cualquier cosa con la que podamos
hacer un ciclo `for`/`of`) y una función que calcula un nombre de grupo para un
elemento dado. Retorna un array de objetos, cada uno de los cuales nombra un
grupo y te dice la cantidad de elementos que se encontraron en ese grupo.

{{index "findIndex method", "indexOf method"}}

Utiliza otro método de array—`findIndex` ("encontrar index"). Este método es
algo así como `indexOf`, pero en lugar de buscar un valor específico, este
encuentra el primer valor para el cual la función dada retorna verdadero.
Como `indexOf`, retorna -1 cuando no se encuentra dicho elemento.

{{index "textScripts function", "Chinese characters"}}

Usando `contarPor`, podemos escribir la función que nos dice qué codigos
se usan en una pieza de texto.

```{includeCode: strip_log, startCode: true}
function codigosTexto(texto) {
  let codigos = contarPor(texto, caracter => {
    let codigo = codigoCaracter(caracter.codePointAt(0));
    return codigo ? codigo.name : "ninguno";
  }).filter(({name}) => name != "ninguno");

  let total = codigos.reduce((n, {count}) => n + count, 0);
  if (total == 0) return "No se encontraron codigos";

  return codigos.map(({name, count}) => {
    return `${Math.round(count * 100 / total)}% ${name}`;
  }).join(", ");
}

console.log(codigosTexto('英国的狗说"woof", 俄罗斯的狗说"тяв"'));
// → 61% Han, 22% Latin, 17% Cyrillic
```

{{index "characterScript function", "filter method"}}

La función primero cuenta los caracteres por nombre, usando
`codigoCaracter` para asignarles un nombre, y recurre al
string `"ninguno"` para caracteres que no son parte de ningún codigo.
La llamada `filter` deja afuera las entrada para `"ninguno"` del array resultante,
ya que no estamos interesados ​​en esos caracteres.

{{index "reduce method", "map method", "join method", [array, methods]}}

Para poder calcular ((porcentaje))s, primero necesitamos la cantidad total
de caracteres que pertenecen a un codigo, lo que podemos calcular con
`reduce`. Si no se encuentran tales caracteres, la función retorna un
string específico. De lo contrario, transforma las entradas de conteo en
strings legibles con `map` y luego las combina con `join`.

## Resumen

Ser capaz de pasar valores de función a otras funciones es un aspecto
profundamente útil de JavaScript. Nos permite escribir funciones que
modelen calculos con "brechas" en ellas. El código que llama a estas
funciones pueden llenar estas brechas al proporcionar valores de función.

Los arrays proporcionan varios métodos útiles de orden superior. Puedes usar
`forEach` para recorrer los elementos en un array. El método `filter`
retorna un nuevo array que contiene solo los elementos que pasan una
((función de predicado)). Transformar un array al poner cada elemento
a través de una función se hace con `map`. Puedes usar `reduce` para combinar
todos los elementos en una array a un solo valor. El método `some`
prueba si algun elemento coincide con una función de predicado determinada. Y
`findIndex` encuentra la posición del primer elemento que coincide con un
predicado.

## Ejercicios

### Aplanamiento

{{index "flattening (exercise)", "reduce method", "concat method", array}}

Use el método `reduce` en combinación con el método `concat` para
"aplanar" un array de arrays en un único array que tenga todos los
elementos de los arrays originales.

{{if interactive

```{test: no}
let arrays = [[1, 2, 3], [4, 5], [6]];
// Tu código aquí.
// → [1, 2, 3, 4, 5, 6]
```
if}}

### Tu propio ciclo

{{index "your own loop (example)", "for loop"}}

Escriba una función de orden superior llamada `ciclo` que proporcione algo así
como una declaración de ciclo `for`. Esta toma un valor, una función de prueba,
una función de actualización y un cuerpo de función. En cada iteración,
primero ejecuta la función de prueba en el valor actual del ciclo y se detiene
si esta retorna falso. Luego llama al cuerpo de función, dándole el valor
actual. Y finalmente, llama a la función de actualización para crear un nuevo
valor y comienza desde el principio.

Cuando definas la función, puedes usar un ciclo regular para hacer los
ciclos reales.

{{if interactive

```{test: no}
// Tu código aquí.

loop(3, n => n > 0, n => n - 1, console.log);
// → 3
// → 2
// → 1
```

if}}

### Cada

{{index "predicate function", "everything (exercise)", "every method", "some method", [array, methods], "&& operator", "|| operator"}}

De forma análoga al método `some`, los arrays también tienen un método `every`
("cada"). Este retorna true cuando la función dada devuelve verdadero
para _cada_ elemento en el array. En cierto modo, `some` es una versión
del operador `||` que actúa en arrays, y `every` es como el operador `&&`.

Implementa `every` como una función que tome un array y una función predicado
como parámetros. Escribe dos versiones, una usando un ciclo y una
usando el método `some`.

{{if interactive

```{test: no}
function cada(array, test) {
  // Tu código aquí.
}

console.log(cada([1, 3, 5], n => n < 10));
// → true
console.log(cada([2, 4, 16], n => n < 10));
// → false
console.log(cada([], n => n < 10));
// → true
```

if}}

{{hint

{{index "everything (exercise)", "short-circuit evaluation", "return keyword"}}

Al igual que el operador `&&`, el método `every` puede dejar de evaluar más
elementos tan pronto como haya encontrado uno que no coincida. Entonces
la versión basada en un ciclo puede saltar fuera del ciclo—con `break` o
`return`—tan pronto como se encuentre con un elemento para el cual la función
predicado retorne falso. Si el ciclo corre hasta su final sin encontrar
tal elemento, sabemos que todos los elementos coinciden y debemos
retornar verdadero.

Para construir `cada` usando `some`, podemos aplicar las _((leyes De
Morgan))_, que establecen que `a && b` es igual a `!(!a ||! b)`. Esto puede ser
generalizado a arrays, donde todos los elementos del array coinciden si no hay
elemento en el array que no coincida.

hint}}

### Dirección de Escritura Dominante

{{index "SCRIPTS data set", "direction (writing)", "groupBy function", "dominant direction (exercise)"}}

Escriba una función que calcule la dirección de escritura dominante en un
string de texto. Recuerde que cada objeto de codigo tiene una propiedad
`direction` que puede ser `"ltr"` (de izquierda a derecha), `"rtl"`
(de derecha a izquierda), o `"ttb"` (arriba a abajo).

{{index "characterScript function", "countBy function"}}

La dirección dominante es la dirección de la mayoría de los
caracteres que tienen un código asociado a ellos. Las funciones
`codigoCaracter` y `contarPor` definidas anteriormente en el
capítulo probablemente seran útiles aquí.

{{if interactive

```{test: no}
function direccionDominante(texto) {
  // Tu código aquí.
}

console.log(direccionDominante("Hola!"));
// → ltr
console.log(direccionDominante("Hey, مساء الخير"));
// → rtl
```
if}}

{{hint

{{index "dominant direction (exercise)", "textScripts function", "filter method", "characterScript function"}}

Tu solución puede parecerse mucho a la primera mitad del
ejemplo `codigosTexto`. De nuevo debes contar los caracteres por el
criterio basado en `codigoCaracter`, y luego filtrar hacia afuera la parte del
resultado que se refiere a caracteres sin interés (que no tengan codigos).

{{index "reduce method"}}

Encontrar la dirección con la mayor cantidad de caracteres se puede hacer
con `reduce`. Si no está claro cómo, refiérate al ejemplo
anterior en el capítulo, donde se usa `reduce` para encontrar el código
con la mayoría de los caracteres.

hint}}
