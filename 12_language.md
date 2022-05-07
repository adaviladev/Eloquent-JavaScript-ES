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

Luego cortamos la parte que coincidio del string del programa y
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

{{index "evaluar función", evaluación, interpretación, "árbol sintáctico", "lenguaje huevo"}}

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

{{index [function, application]}}

Applications are more involved. If they are a ((special form)), like
`if`, we do not evaluate anything and pass the argument expressions,
along with the scope, to the function that handles this form. If it is
a normal call, we evaluate the operator, verify that it is a function,
and call it with the evaluated arguments.

We use plain JavaScript function values to represent Egg's function
values. We will come back to this [later](language#egg_fun), when the
special form called `fun` is defined.

{{index readability, "evaluate function", recursion, parsing}}

The recursive structure of `evaluate` resembles the similar structure
of the parser, and both mirror the structure of the language itself.
It would also be possible to integrate the parser with the evaluator
and evaluate during parsing, but splitting them up this way makes the
program clearer.

{{index "Egg language", interpretation}}

This is really all that is needed to interpret Egg. It is that simple.
But without defining a few special forms and adding some useful values
to the ((environment)), you can't do much with this language yet.

## Special forms

{{index "special form", "specialForms object"}}

The `specialForms` object is used to define special syntax in Egg. It
associates words with functions that evaluate such forms. It is
currently empty. Let's add `if`.

```{includeCode: true}
specialForms.si = (args, scope) => {
  if (args.length != 3) {
    throw new SyntaxError("Wrong number of args to si");
  } else if (evaluate(args[0], scope) !== false) {
    return evaluate(args[1], scope);
  } else {
    return evaluate(args[2], scope);
  }
};
```

{{index "conditional execution", "ternary operator", "?: operator", "conditional operator"}}

Egg's `si` construct expects exactly three arguments. It will evaluate
the first, and if the result isn't the value `false`, it will evaluate
the second. Otherwise, the third gets evaluated. This `if` form is
more similar to JavaScript's ternary `?:` operator than to
JavaScript's `if`. It is an expression, not a statement, and it
produces a value, namely the result of the second or third argument.

{{index Boolean}}

Egg also differs from JavaScript in how it handles the condition value
to `if`. It will not treat things like zero or the empty string as
false, only the precise value `false`.

{{index "short-circuit evaluation"}}

The reason we need to represent `if` as a special form, rather than a
regular function, is that all arguments to functions are evaluated
before the function is called, whereas `if` should evaluate only
_either_ its second or its third argument, depending on the value of
the first.

The `while` form is similar.

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

Another basic building block is `hacer`, which executes all its arguments
from top to bottom. Its value is the value produced by the last
argument.

```{includeCode: true}
specialForms.hacer = (args, scope) => {
  let value = false;
  for (let arg of args) {
    value = evaluate(arg, scope);
  }
  return value;
};
```

{{index "= operator"}}

To be able to create ((binding))s and give them new values, we also
create a form called `definir`. It expects a word as its first argument
and an expression producing the value to assign to that word as its
second argument. Since `definir`, like everything, is an expression, it
must return a value. We'll make it return the value that was assigned
(just like JavaScript's `=` operator).

```{includeCode: true}
specialForms.definir = (args, scope) => {
  if (args.length != 2 || args[0].type != "word") {
    throw new SyntaxError("Incorrect use of definir");
  }
  let value = evaluate(args[1], scope);
  scope[args[0].name] = value;
  return value;
};
```

## The environment

{{index "Egg language", "evaluate function"}}

The ((scope)) accepted by `evaluate` is an object with properties
whose names correspond to binding names and whose values correspond to
the values those ((binding))s are bound to. Let's define an object to
represent the ((global scope)).

To be able to use the `if` construct we just defined, we must have
access to ((Boolean)) values. Since there are only two Boolean values,
we do not need special syntax for them. We simply bind two names to
the values `true` and `false` and use those.

```{includeCode: true}
const topScope = Object.create(null);

topScope.true = true;
topScope.false = false;
```

We can now evaluate a simple expression that negates a Boolean value.

```
let prog = parse(`si(true, false, true)`);
console.log(evaluate(prog, topScope));
// → false
```

{{index arithmetic, "Function constructor"}}

To supply basic ((arithmetic)) and ((comparison)) ((operator))s, we
will also add some function values to the ((scope)). In the interest
of keeping the code short, we'll use `Function` to synthesize a bunch
of operator functions in a loop, instead of defining them
individually.

```{includeCode: true}
for (let op of ["+", "-", "*", "/", "==", "<", ">"]) {
  topScope[op] = Function("a, b", `return a ${op} b;`);
}
```

A way to ((output)) values is also very useful, so we'll wrap
`console.log` in a function and call it `imprimir`.

```{includeCode: true}
topScope.imprimir = value => {
  console.log(value);
  return value;
};
```

{{index parsing, "run function"}}

That gives us enough elementary tools to write simple programs. The
following function provides a convenient way to parse a program and
run it in a fresh scope.

```{includeCode: true}
function run(programa) {
  return evaluate(parse(programa), Object.create(topScope));
}
```

{{index "Object.create function", prototype}}

We'll use object prototype chains to represent nested scopes, so that
the program can add bindings to its local scope without changing the
top-level scope.

```
run(`
hacer(definir(total, 0),
   definir(count, 1),
   while(<(count, 11),
         hacer(definir(total, +(total, count)),
            definir(count, +(count, 1)))),
   imprimir(total))
`);
// → 55
```

{{index "summing example", "Egg language"}}

This is the program we've seen several times before, which computes
the sum of the numbers 1 to 10, expressed in Egg. It is clearly uglier
than the equivalent JavaScript program—but not bad for a language
implemented in less than 150 ((lines of code)).

{{id egg_fun}}

## Functions

{{index function, "Egg language"}}

A programming language without functions is a poor programming
language indeed.

Fortunately, it isn't hard to add a `fun` construct, which treats its
last argument as the function's body and uses all arguments before
that as the names of the function's parameters.

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

{{index "local scope"}}

Functions in Egg get their own local scope. The function produced by
the `fun` form creates this local scope and adds the argument bindings
to it. It then evaluates the function body in this scope and returns
the result.

```{startCode: true}
run(`
hacer(definir(plusOne, fun(a, +(a, 1))),
   imprimir(plusOne(10)))
`);
// → 11

run(`
hacer(definir(pow, fun(base, exp,
     si(==(exp, 0),
        1,
        *(base, pow(base, -(exp, 1)))))),
   imprimir(pow(2, 10)))
`);
// → 1024
```

## Compilation

{{index interpretation, compilation}}

What we have built is an interpreter. During evaluation, it acts
directly on the representation of the program produced by the parser.

{{index efficiency, performance}}

_Compilation_ is the process of adding another step between the
parsing and the running of a program, which transforms the program
into something that can be evaluated more efficiently by doing as much
work as possible in advance. For example, in well-designed languages
it is obvious, for each use of a ((binding)), which binding is being
referred to, without actually running the program. This can be used to
avoid looking up the binding by name every time it is accessed,
instead directly fetching it from some predetermined ((memory))
location.

Traditionally, ((compilation)) involves converting the program to
((machine code)), the raw format that a computer's processor can
execute. But any process that converts a program to a different
representation can be thought of as compilation.

{{index simplicity, "Function constructor", transpilation}}

It would be possible to write an alternative ((evaluation)) strategy
for Egg, one that first converts the program to a JavaScript program,
uses `Function` to invoke the JavaScript compiler on it, and then runs
the result. When done right, this would make Egg run very fast while
still being quite simple to implement.

If you are interested in this topic and willing to spend some time on
it, I encourage you to try to implement such a compiler as an
exercise.

## Cheating

{{index "Egg language"}}

When we defined `if` and `while`, you probably noticed that they were
more or less trivial wrappers around JavaScript's own `if` and
`while`. Similarly, the values in Egg are just regular old JavaScript
values.

If you compare the implementation of Egg, built on top of JavaScript,
with the amount of work and complexity required to build a programming
language directly on the raw functionality provided by a machine, the
difference is huge. Regardless, this example hopefully gave you an
impression of the way ((programming language))s work.

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
hacer(definir(sum, fun(array,
     hacer(definir(i, 0),
        definir(sum, 0),
        while(<(i, length(array)),
          hacer(definir(sum, +(sum, element(array, i))),
             definir(i, +(i, 1)))),
        sum))),
   imprimir(sum(array(1, 2, 3))))
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
hacer(definir(f, fun(a, fun(b, +(a, b)))),
   imprimir(f(4)(5)))
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
