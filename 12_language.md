{{meta {load_files: ["code/chapter/12_language.js"], zip: "node/html"}}}

# Proyecto: Un Lenguaje de Programación

{{quote {author: "Hal Abelson and Gerald Sussman", title: "Structure and Interpretation of Computer Programs", chapter: true}

El evaluador, que determina el significado de las expresiones en un
lenguaje de programación, es solo otro programa.

quote}}

{{index "Abelson, Hal", "Sussman, Gerald", SICP, "project chapter"}}

Construir tu propio ((lenguaje de programación)) es sorprendentemente fácil
(siempre y cuando no apuntes demasiado alto) y muy esclarecedor.

Lo principal que quiero mostrar en este capítulo es que no hay
((magia)) involucrada en la construcción de tu propio lenguaje. A menudo
he sentido que algunos inventos humanos eran tan inmensamente inteligentes
y complicados que nunca podría llegar a entenderlos. Pero con un poco de lectura
y experimentación, a menudo resultan ser bastante mundanos.

{{index "Egg language"}}

Construiremos un lenguaje de programación llamado Egg. Será un lenguaje pequeño
y simple—pero lo suficientemente poderoso como para expresar cualquier
computación que puedes pensar. Permitirá una ((abstracción)) simple
basada en ((funcion))es.

{{id parsing}}

## Análisis

{{index parsing, validation}}

La parte más visible de un lenguaje de programación es su
_((sintaxis))_, o notación. Un _analizador_ es un programa que lee una pieza
de texto y produce una estructura de datos que refleja la estructura del
programa contenido en ese texto. Si el texto no forma un programa válido,
el analizador debe señalar el error.

{{index "special form", [function, application]}}

Nuestro lenguaje tendrá una sintaxis simple y uniforme. Todo en Egg
es una ((expresión)). Una expresión puede ser el nombre de una vinculación (binding), un
número, una cadena de texto o una _aplicación_. Las aplicaciones son usadas para
llamadas de función pero también para constructos como `if` o `while`.

{{index "double-quote character", parsing, [escaping, "in strings"]}}

Para mantener el analizador simple, las cadenas en Egg no soportarán nada parecido
a escapes de barra invertida. Una cadena es simplemente una secuencia de
caracteres que no sean comillas dobles, envueltas en comillas dobles. Un número
es un secuencia de dígitos. Los nombres de vinculaciones pueden consistir en
cualquier carácter no que sea un ((espacio en blanco)) y eso no tiene un
significado especial en la sintaxis.

{{index "comma character"}}

Las aplicaciones se escriben tal y como están en JavaScript, poniendo
((paréntesis)) después de una expresión y teniendo cualquier cantidad de
((argumento))s entre esos paréntesis, separados por comas.

```{lang: null}
hacer(definir(x, 10),
   si(>(x, 5),
      imprimir("grande"),
      imprimir("pequeño")))
```

{{index block}}

La ((uniformidad)) del ((lenguaje Egg)) significa que las cosas que son
((operador))es en JavaScript (como `>`) son vinculaciones normales en este
lenguaje, aplicadas como cualquier ((función)). Y dado que la
((sintaxis)) no tiene un concepto de bloque, necesitamos una construcción
`hacer` para representar el hecho de realizar múltiples cosas en secuencia.

{{index "type property", parsing}}

La ((estructura de datos)) que el analizador usará para describir un programa
consta de objetos de ((expresión)), cada uno de los cuales tiene una
propiedad `tipo` que indica el tipo de expresión que este es y otras propiedades
que describen su contenido.

{{index identifier}}

Las expresiones de tipo `"valor"` representan strings o números literales.
Su propiedad `valor` contiene el string o valor numérico que estos
representan. Las expresiones de tipo `"palabra"` se usan para identificadores
(nombres). Dichos objetos tienen una propiedad `nombre` que contienen el
nombre del identificador como un string. Finalmente, las expresiones `"aplicar"`
representan aplicaciones. Tienen una propiedad `operador` que se refiere
a la expresión que está siendo aplicada, y una propiedad `argumentos` que
contiene un array de expresiones de argumentos.

La parte `>(x, 5)` del programa anterior se representaría de esta manera:

```{lang: "application/json"}
{
  tipo: "aplicar",
  operador: {tipo: "palabra", nombre: ">"},
  argumentos: [
    {tipo: "palabra", nombre: "x"},
    {tipo: "valor", valor: 5}
  ]
}
```

{{indexsee "abstract syntax tree", "syntax tree"}}

Tal ((estructura de datos)) se llama _((árbol de sintaxis))_. Si tu
imaginas los objetos como puntos y los enlaces entre ellos como líneas
entre esos puntos, tiene una forma similar a un ((árbol)). El hecho de que
las expresiones contienen otras expresiones, que a su vez pueden contener
más expresiones, es similar a la forma en que las ramas de los árboles se
dividen y dividen una y otra vez.

{{figure {url: "img/syntax_tree.svg", alt: "The structure of a syntax tree",width: "5cm"}}}

{{index parsing}}

Compara esto con el analizador que escribimos para el formato de archivo de
configuración en el [Capítulo 9](regexp#ini), que tenía una estructura simple:
dividía la entrada en líneas y manejaba esas líneas una a la vez. Habían
solo unas pocas formas simples que se le permitía tener a una línea.

{{index recursion, [nesting, "of expressions"]}}

Aquí debemos encontrar un enfoque diferente. Las expresiones no están
separadas en líneas, y tienen una estructura recursiva. Las
expresiones de aplicaciones _contienen_ otras expresiones.

{{index elegance}}

Afortunadamente, este problema se puede resolver muy bien escribiendo una
función analizadora que es recursiva en una manera que refleje
la naturaleza recursiva del lenguaje.

{{index "parseExpression function", "syntax tree"}}

Definimos una función `analizarExpresion`, que toma un string como entrada
y retorna un objeto que contiene la estructura de datos para la expresión
al comienzo del string, junto con la parte del string que queda
después de analizar esta expresión Al analizar subexpresiones (el
argumento para una aplicación, por ejemplo), esta función puede ser llamada
de nuevo, produciendo la expresión del argumento, así como al texto que
permanece. A su vez, este texto puede contener más argumentos o puede ser el
paréntesis de cierre que finaliza la lista de argumentos.

Esta es la primera parte del analizador:

```{includeCode: true}
function analizarExpresion(programa) {
  programa = saltarEspacio(programa);
  let emparejamiento, expresion;
  if (emparejamiento = /^"([^"]*)"/.exec(programa)) {
    expresion = {tipo: "valor", valor: emparejamiento[1]};
  } else if (emparejamiento = /^\d+\b/.exec(programa)) {
    expresion = {tipo: "valor", valor: Number(emparejamiento[0])};
  } else if (emparejamiento = /^[^\s(),"]+/.exec(programa)) {
    expresion = {tipo: "palabra", nombre: emparejamiento[0]};
  } else {
    throw new SyntaxError("Sintaxis inesperada: " + programa);
  }

  return aplicarAnalisis(expresion, programa.slice(emparejamiento[0].length));
}

function saltarEspacio(string) {
  let primero = string.search(/\S/);
  if (primero == -1) return "";
  return string.slice(primero);
}
```

{{index "skipSpace function"}}

Dado que Egg, al igual que JavaScript, permite cualquier cantidad de ((espacios
en blanco)) entre sus elementos, tenemos que remover repetidamente los espacios
en blanco del inicio del string del programa. Para esto nos ayuda la función
`saltarEspacio`.

{{index "literal expression", "SyntaxError type"}}

Después de saltar cualquier espacio en blanco, `analizarExpresion` usa tres
((expresiones regular))es para detectar los tres elementos atómicos que Egg
soporta: strings, números y palabras. El analizador construye un
tipo diferente de estructura de datos dependiendo de cuál coincida. Si
la entrada no coincide con ninguna de estas tres formas, la expresión no
es válida, y el analizador arroja un error. Usamos `SyntaxError`
en lugar de `Error` como constructor de excepción, el cual es otro
tipo de error estándar, dado que es un poco más específico—también es el
tipo de error lanzado cuando se intenta ejecutar un programa de
JavaScript no válido.

{{index "parseApply function"}}

Luego cortamos la parte que coincidió del string del programa y
pasamos eso, junto con el objeto para la expresión, a `aplicarAnalisis`,
el cual verifica si la expresión es una aplicación. Si es así,
analiza una lista de los argumentos entre paréntesis.

```{includeCode: true}
function aplicarAnalisis(expresion, programa) {
  programa = saltarEspacio(programa);
  if (programa[0] != "(") {
    return {expresion: expresion, resto: programa};
  }

  programa = saltarEspacio(programa.slice(1));
  expresion = {tipo: "aplicar", operador: expresion, argumentos: []};
  while (programa[0] != ")") {
    let argumento = analizarExpresion(programa);
    expresion.argumentos.push(argumento.expresion);
    programa = saltarEspacio(argumento.resto);
    if (programa[0] == ",") {
      programa = saltarEspacio(programa.slice(1));
    } else if (programa[0] != ")") {
      throw new SyntaxError("Experaba ',' o ')'");
    }
  }
  return aplicarAnalisis(expresion, programa.slice(1));
}
```

{{index parsing}}

Si el siguiente carácter en el programa no es un paréntesis de apertura,
esto no es una aplicación, y `aplicarAnalisis` retorna la expresión que
se le dio.

{{index recursion}}

De lo contrario, salta el paréntesis de apertura y crea el objeto de
((árbol)) de sintaxis para esta expresión de aplicación. Entonces, recursivamente
llama a `analizarExpresion` para analizar cada argumento hasta que se encuentre
el paréntesis de cierre. La recursión es indirecta, a través de `aplicarAnalisis`
y `analizarExpresion` llamando una a la otra.

Dado que una expresión de aplicación puede ser aplicada a sí misma (como en
`multiplicador(2)(1)`), `aplicarAnalisis` debe, después de haber analizado una
aplicación, llamarse asi misma de nuevo para verificar si otro par de
paréntesis sigue a continuación.

{{index "syntax tree", "Egg language", "parse function"}}

Esto es todo lo que necesitamos para analizar Egg. Envolvemos esto en una
conveniente función `analizar` que verifica que ha llegado al final del string
de entrada después de analizar la expresión (un programa Egg es una sola
expresión), y eso nos da la estructura de datos del programa.

```{includeCode: strip_log, test: join}
function analizar(programa) {
  let {expresion, resto} = analizarExpresion(programa);
  if (saltarEspacio(resto).length > 0) {
    throw new SyntaxError("Texto inesperado despues de programa");
  }
  return expresion;
}

console.log(analizar("+(a, 10)"));
// → {tipo: "aplicar",
//    operador: {tipo: "palabra", nombre: "+"},
//    argumentos: [{tipo: "palabra", nombre: "a"},
//           {tipo: "valor", valor: 10}]}
```

{{index "error message"}}

¡Funciona! No nos da información muy útil cuando falla
y no almacena la línea y la columna en que comienza cada expresión,
lo que podría ser útil al informar errores más tarde, pero es lo
suficientemente bueno para nuestros propósitos.

## El evaluador

{{index "evaluar función", evaluación, interpretación, "árbol sintáctico", "lenguaje de Egg"}}

¿Qué poddemos hacer son el árrbol sintáctico de un programa? ¡Correrlo, por supuesto!
Y eso es lo que el evaluador hace. Le das un árbol sintáctico y un objeto de ámbito
que asocie nombres con valores, y evaluará la expresión que el árbol representa y regresará
el valor que el árbol produce.

```{includeCode: true}
const specialForms = Object.create(null);

function evaluate(expresion, scope) {
  if (expresion.type == "value") {
    return expresion.value;
  } else if (expresion.type == "word") {
    if (expresion.name in scope) {
      return scope[expresion.name];
    } else {
      throw new ReferenceError(
        `Undefined binding: ${expresion.name}`);
    }
  } else if (expresion.type == "apply") {
    let {operator, args} = expresion;
    if (operator.type == "word" &&
        operator.name in specialForms) {
      return specialForms[operator.name](expresion.args, scope);
    } else {
      let op = evaluate(operator, scope);
      if (typeof op == "function") {
        return op(...args.map(arg => evaluate(arg, scope)));
      } else {
        throw new TypeError("Applying a non-function.");
      }
    }
  }
}
```

{{index "expresión literal", ámbito}}

El evaluador tiene código para cada uno de los tipos de ((exprresión)). Un
expresión literal de un valor produce este valor. (Por ejemplo, la expresión `100` evalúa
al número 100.) Para un vínculo de valor, debemos verificar si está definido en el ámbito,
y si lo está, traer el valor vinculado.

{{index [función, aplicación]}}

La apliación implica más cosas. Si son una ((forma especial)), como
`if`, no evaluamos nada y pasamos las expresiones argumento, junto con el
ámbito, a la función que maneja esta forma. Si es una llamada normal, evaluamos el operador,
verificamos que sea una función, y la llamamos con los argumentos evaluados.

Usamos valores función planos de JavaScript para representar los valores función de Egg.
Regresaremos a esto [más tardde](language#egg_fun), cuando forma especial llamada
`fun` está definida.


{{index legibilidad, "evaluar función", recursión, parseo}}

La estructura recursiva de `evaluate` se parece a la estructura del parser, y
ambos reflejan la estructura del lenguaje mismo. También sería posible
integrar el parser con el evaluador y evaluar durante el parseo,
pero separarlos de esta manera hace el programa más claro.


{{index "lenguaje Egg", interpretación}}

Esto es todo lo que se necesita para interpretar Egg. Es así de simple.
Pero si no definimos unas cuantas formas especiales y agregamos algunos
valores útiles para el ((entorno)), todavía no podemos hacer mucho con
el lenguaje.

## Formas especiales

{{index "formas especiales", "objeto specialForms"}}

El objeto `specialForms` es usado para definir una sintaxis especial en
Egg. Asocia palabras con funciones que evalúan estas formas. Ahora mismo está
vacío. Agreguemos `if`.


```{includeCode: true}
specialForms.if = (args, scope) => {
  if (args.length != 3) {
    throw new SyntaxError("Wrong number of args to if");
  } else if (evaluate(args[0], scope) !== false) {
    return evaluate(args[1], scope);
  } else {
    return evaluate(args[2], scope);
  }
};
```

{{index "ejecución condicional", "operador ternario", "operador ?:", "operador condicional"}}

El constructo `if` de Egg espera exactamente tres arrgumentos. Evaluará el primero,
y si el resultado no es `false`, evaluará el segundo. En otro caso, el
tercero será evaluado. Esta forma `if` es más parecidda al operaddor
ternario `?:` que al `if` if de JavaScript. Es una expresión, no una
sentencia, y produce un valor: el resultado del segundo o tercer argumento.

{{index Booleano}}

Egg también se diferencia de JavaScript en como maneja el valor de la
condición para `if`. No tratará cosas como cero o la cadena vacía como 
falso, solamente el valor exacto `false`.

{{index "evaluación en corto circuito"}}

La razón por la que necesitamos representar `if` como una forma especial,
en vez de como una función regular, es que todos los argumentos de las
funciones son evaluados antes de que la función sea llamada, mientras
que el `if` debe evaluar solo el segundo _o_ el tercer argumento,
dependiendo del valor del primero.

La forma `while` es similar.


```{includeCode: true}
specialForms.while = (args, scope) => {
  if (args.length != 2) {
    throw new SyntaxError("Wrong number of args to while");
  }
  while (evaluate(args[0], scope) !== false) {
    evaluate(args[1], scope);
  }

  // Since undefined does not exist in Egg, we return false,
  // for lack of a meaningful result.
  return false;
};
```

Otro bloque de construcción principal es `do`, que ejecuta todos sus argumentos
de arriba hacia abajo. Su valos es el valorr producido por el último
argumento.


```{includeCode: true}
specialForms.do = (args, scope) => {
  let value = false;
  for (let arg of args) {
    value = evaluate(arg, scope);
  }
  return value;
};
```

{{index "operador ="}}

Para ser capaces de crear ((asignaciones)) y darle nuevos valores,
además necesitamos crear una forma llamada `define`. Espera una palabra
como prime argumento y una expresión producciendo un valor para asignar
a esa palabra como segundo argumento. Ya que `define`, como todo,
es una expresión, debe regresar un valor. Haremos que regrese el valor que
fue asignado (justo como el operador `=` de JavaScript).


```{includeCode: true}
specialForms.define = (args, scope) => {
  if (args.length != 2 || args[0].type != "word") {
    throw new SyntaxError("Incorrect use of define");
  }
  let value = evaluate(args[1], scope);
  scope[args[0].name] = value;
  return value;
};
```

## El entorno

{{index "lenguaje Egg", "evaluar función"}}

El ((ámbito)) aceptado por `evaluate` es un objeto con propiedades
de las que sus nombres corresponden a los nombres de las vinculaciones en el ámbito
y sus valores corresnponden a los valores de esos vínculos. Ahora definamos un objeto
para representar el ((ámbito global)).

Para poder usar el constructo `if` que acabamos de definir, necesitamos acceder
a valores ((Booleano))s. Como solo hay dos valores Booleanos, no necesitamos una sintaxis
especial. Simplemente asociaremos dos nombres a los valores `true` y `false` y usaremos estos nombres.

```{includeCode: true}
const topScope = Object.create(null);

topScope.true = true;
topScope.false = false;
```

Podemos evaluar una expresión simple que niegue un Booleano.

```
let prog = parse(`if(true, false, true)`);
console.log(evaluate(prog, topScope));
// → false
```

{{index aritmética, "constructor de Función"}}

Para proveer la ((aritmética)) y ((operador))es de ((comparación)),
necesitamos además añadir algunas funciones al ((ámbito)). Para mantener
el código peequeño, usaremos `Function` para sintetizar varias
funciones de operador en un ciclo, en vez de definirlas individualmente.


```{includeCode: true}
for (let op of ["+", "-", "*", "/", "==", "<", ">"]) {
  topScope[op] = Function("a, b", `return a ${op} b;`);
}
```

Tener una manera de ((devolver)) valores tambbién es muy útil,
así que envolveremos `console.log` en una función y la llamaremos
`print`.

```{includeCode: true}
topScope.print = value => {
  console.log(value);
  return value;
};
```

{{index parseo, "función run"}}

Esto nos da suficientes herramientas elementales para escribir
programas simples. La siguiente función provee una forma conveniente
de leer un programa y correrlo en un ámbito nuevo.


```{includeCode: true}
function run(program) {
  return evaluate(parse(program), Object.create(topScope));
}
```

{{index "función Object.create", prototipo}}

Usareomos las cadenas de prototipos de los objetos para representar
ámbitos anidados, para que el programa pueda añadir asignaciones a su
ámbito local sin cambiar el ámbito superior.

```
run(`
do(define(total, 0),
   define(count, 1),
   while(<(count, 11),
         do(define(total, +(total, count)),
            define(count, +(count, 1)))),
   print(total))
`);
// → 55
```

{{index "ejemplo de suma", "lenguaje Egg"}}

Este es el programa que hemos visto varias veces antes, que calcula la suma
de los números del 1 a 10, expresado en Egg. Es claramente más
feo que su equivalente en JavaScript, peror no está tan mal para
un lenguaje implementado en menos de 150 ((líneas de código)).


{{id egg_fun}}

## Funciones

{{index función, "lenguaje Egg"}}

Un lenguaje de programación sin funciones es un lenguaje de
programación pobre, en efecto.

Afortunadamente, no es difícil agregar un constructo `fun`,
que trate su último argumento como el cuerpo de la función y use
todos los argumentos antes de ese como los parámetros de la función.


```{includeCode: true}
specialForms.fun = (args, scope) => {
  if (!args.length) {
    throw new SyntaxError("Functions need a body");
  }
  let body = args[args.length - 1];
  let params = args.slice(0, args.length - 1).map(expr => {
    if (expr.type != "word") {
      throw new SyntaxError("Parameter names must be words");
    }
    return expr.name;
  });

  return function() {
    if (arguments.length != params.length) {
      throw new TypeError("Wrong number of arguments");
    }
    let localScope = Object.create(scope);
    for (let i = 0; i < arguments.length; i++) {
      localScope[params[i]] = arguments[i];
    }
    return evaluate(body, localScope);
  };
};
```

{{index "ámbito local"}}

Las funciones en Egg tienen su propio ámbito local. La función producida
por la forma `fun` crea este entorno local y agrega las asociaciones de
argumentos a este. Entonces evalúa el cuerpo de la función en este ámbito
y regresa el resultado.

```{startCode: true}
run(`
do(define(plusOne, fun(a, +(a, 1))),
   imprimir(plusOne(10)))
`);
// → 11

run(`
do(define(pow, fun(base, exp,
     si(==(exp, 0),
        1,
        *(base, pow(base, -(exp, 1)))))),
   imprimir(pow(2, 10)))
`);
// → 1024
```

## Compilación

{{index interpretación, compilación}}

Lo que hemos construido es un intérprete. Durante la evaluación, actúa
directamente en la representación del programa producido por el
parseador.


{{index eficiencia, rendimiento}}

La _Compilación_ es el proceso de añadir otro paso entre la lectura
del programa y su ejecución, que transforma el programa en algo que
pueda ser evaluado más eficientemente, haciendo la mayor cantidad
de trabajo posible por adelantado. Por ejemplo, en lenguajes bien
diseñados es obvio, para cada uso de una ((asignación)), a qué asignación
te estás refiriendo, sin correr el programa. Esto puede ser usado para
evitar buscar la asociación por nombbre cada vez que se usa, y, en vez de eso,
traer directamente el valor de una locación predeterminada de
((memoria)).

Tradicionalmente, la ((compilación)) implica convertir el programa
en ((código máquina)), el formato crudo que el procesador de la computadora
puede ejecutar. Pero puedes pensar en compilación como cualquier proceso
que convierta un programa en una representación diferente.

{{index simplicidad, "función constructor", transpilación}}

Sería posible escribir una estrategia de ((evaluación)) alternativa
para Egg, una que primero convierrta el programa a uno de JavaScript,
usa `Function` para invocar el compilador de JavaSctipt, y después correr
el resultado. Cuando se hace correctamente, esto haría que Egg
corrra muy rápiddo mientras todavía sigue siendo simple implementarlo.

Si estás interesado en este tema y dispuesto a gastar algo de tiempo,
te animo a que trates de implementar el compilador como ejercicio.

## Haciendo trampa

{{index "lenguaje Egg"}}

Cuando definimos `id` y `while`, probablemente te diste cuenta de que son
más o menos envoltorios triviales alrededor de los `if` y `while` de
JavaScript. Similarmente, los valores en Egg son solo valores de
JavaScript comunes.

Si comparas la implementación de Egg, construida soobre JavaScript,
con la cantidad de trabajo y complejidad requerida para
construir un lenguaje de programación directamente en la funcionalidad
pura y cruda provista por una máquina, la diferencia es grande. Sin embargo,
con suerte este ejemplo te dio un vistazo de la forma en
que un ((lenguaje de programación)) funciona.


And when it comes to getting something done, cheating is more
effective than doing everything yourself. Though the toy language in
this chapter doesn't do anything that couldn't be done better in
JavaScript, there _are_ situations where writing small languages helps
get real work done.

Such a language does not have to resemble a typical programming
language. If JavaScript didn't come equipped with regular expressions,
for example, you could write your own parser and evaluator for regular
expressions.

{{index "artificial intelligence"}}

Or imagine you are building a giant robotic ((dinosaur)) and need to
program its ((behavior)). JavaScript might not be the most effective
way to do this. You might instead opt for a language that looks like
this:

```{lang: null}
behavior walk
  perform when
    destination ahead
  actions
    move left-foot
    move right-foot

behavior attack
  perform when
    Godzilla in-view
  actions
    fire laser-eyes
    launch arm-rockets
```

{{index expressivity}}

This is what is usually called a _((domain-specific language))_, a
language tailored to express a narrow domain of knowledge. Such a
language can be more expressive than a general-purpose language
because it is designed to describe exactly the things that need to be
described in its domain, and nothing else.

## Exercises

### Arrays

{{index "Egg language", "arrays in egg (exercise)"}}

Add support for ((array))s to Egg by adding the following three
functions to the top scope: `array(...values)` to construct an array
containing the argument values, `length(array)` to get an array's
length, and `element(array, n)` to fetch the n^th^ element from an
array.

{{if interactive

```{test: no}
// Modify these definitions...

topScope.array = "...";

topScope.length = "...";

topScope.element = "...";

run(`
do(define(sum, fun(array,
     do(define(i, 0),
        define(sum, 0),
        while(<(i, length(array)),
          do(define(sum, +(sum, element(array, i))),
             define(i, +(i, 1)))),
        sum))),
   print(sum(array(1, 2, 3))))
`);
// → 6
```

if}}

{{hint

{{index "arrays in egg (exercise)"}}

The easiest way to do this is to represent Egg arrays with JavaScript
arrays.

{{index "slice method"}}

The values added to the top scope must be functions. By using a rest
argument (with triple-dot notation), the definition of `array` can be
_very_ simple.

hint}}

### Closure

{{index closure, [function, scope], "closure in egg (exercise)"}}

The way we have defined `fun` allows functions in Egg to reference
the surrounding scope, allowing the function's body to use local
values that were visible at the time the function was defined, just
like JavaScript functions do.

The following program illustrates this: function `f` returns a
function that adds its argument to `f`'s argument, meaning that it
needs access to the local ((scope)) inside `f` to be able to use
binding `a`.

```
run(`
do(define(f, fun(a, fun(b, +(a, b)))),
   print(f(4)(5)))
`);
// → 9
```

Go back to the definition of the `fun` form and explain which
mechanism causes this to work.

{{hint

{{index closure, "closure in egg (exercise)"}}

Again, we are riding along on a JavaScript mechanism to get the
equivalent feature in Egg. Special forms are passed the local scope in
which they are evaluated so that they can evaluate their subforms in
that scope. The function returned by `fun` has access to the `scope`
argument given to its enclosing function and uses that to create the
function's local ((scope)) when it is called.

{{index compilation}}

This means that the ((prototype)) of the local scope will be the scope
in which the function was created, which makes it possible to access
bindings in that scope from the function. This is all there is to
implementing closure (though to compile it in a way that is actually
efficient, you'd need to do some more work).

hint}}

### Comments

{{index "hash character", "Egg language", "comments in egg (exercise)"}}

It would be nice if we could write ((comment))s in Egg. For example,
whenever we find a hash sign (`#`), we could treat the rest of the
line as a comment and ignore it, similar to `//` in JavaScript.

{{index "skipSpace function"}}

We do not have to make any big changes to the parser to support this.
We can simply change `skipSpace` to skip comments as if they are
((whitespace)) so that all the points where `skipSpace` is called will
now also skip comments. Make this change.

{{if interactive

```{test: no}
// This is the old skipSpace. Modify it...
function skipSpace(string) {
  let first = string.search(/\S/);
  if (first == -1) return "";
  return string.slice(first);
}

console.log(parse("# hello\nx"));
// → {type: "word", name: "x"}

console.log(parse("a # one\n   # two\n()"));
// → {type: "apply",
//    operator: {type: "word", name: "a"},
//    args: []}
```

if}}

{{hint

{{index comment, "comments in egg (exercise)"}}

Make sure your solution handles multiple comments in a row, with
potentially ((whitespace)) between or after them.

A ((regular expression)) is probably the easiest way to solve this.
Write something that matches "whitespace or a comment, zero or more
times". Use the `exec` or `match` method and look at the length of the
first element in the returned array (the whole match) to find out how
many characters to slice off.

hint}}

### Fixing scope

{{index [binding, definition], assignment, "fixing scope (exercise)"}}

Currently, the only way to assign a ((binding)) a value is `definir`.
This construct acts as a way both to define new bindings and to give
existing ones a new value.

{{index "local binding"}}

This ((ambiguity)) causes a problem. When you try to give a nonlocal
binding a new value, you will end up defining a local one with the
same name instead. Some languages work like this by design, but I've
always found it an awkward way to handle ((scope)).

{{index "ReferenceError type"}}

Add a special form `set`, similar to `definir`, which gives a binding a
new value, updating the binding in an outer scope if it doesn't
already exist in the inner scope. If the binding is not defined at
all, throw a `ReferenceError` (another standard error type).

{{index "hasOwnProperty method", prototype, "getPrototypeOf function"}}

The technique of representing scopes as simple objects, which has made
things convenient so far, will get in your way a little at this point.
You might want to use the `Object.getPrototypeOf` function, which
returns the prototype of an object. Also remember that scopes do not
derive from `Object.prototype`, so if you want to call
`hasOwnProperty` on them, you have to use this clumsy expression:

```{test: no}
Object.prototype.hasOwnProperty.call(scope, name);
```

{{if interactive

```{test: no}
specialForms.set = (args, scope) => {
  // Your code here.
};

run(`
hacer(definir(x, 4),
   definir(setx, fun(val, set(x, val))),
   setx(50),
   imprimir(x))
`);
// → 50
run(`set(quux, true)`);
// → Some kind of ReferenceError
```

if}}

{{hint

{{index [binding, definition], assignment, "getPrototypeOf function", "hasOwnProperty method", "fixing scope (exercise)"}}

You will have to loop through one ((scope)) at a time, using
`Object.getPrototypeOf` to go the next outer scope. For each scope,
use `hasOwnProperty` to find out whether the binding, indicated by the
`name` property of the first argument to `set`, exists in that scope.
If it does, set it to the result of evaluating the second argument to
`set` and then return that value.

{{index "global scope", "run-time error"}}

If the outermost scope is reached (`Object.getPrototypeOf` returns
null) and we haven't found the binding yet, it doesn't exist, and an
error should be thrown.

hint}}
