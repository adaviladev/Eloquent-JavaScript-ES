{{meta {load_files: ["code/chapter/07_robot.js", "code/animatevillage.js"], zip: html}}}

# Proyecto: Un Robot

{{quote {author: "Edsger Dijkstra", title: "The Threats to Computing Science", chapter: true}

[...] la pregunta de si las Maquinas Pueden Pensar [...] es tan
relevante como la pregunta de si los Submarinos Pueden Nadar.

quote}}

{{index "artificial intelligence", "Dijkstra, Edsger"}}

{{figure {url: "img/chapter_picture_7.jpg", alt: "Picture of a package-delivery robot", chapter: framed}}}

{{index "project chapter", "reading code", "writing code"}}

En los capítulos de "proyectos", dejaré de golpearte con teoría nueva
por un breve momento y en su lugar vamos a trabajar juntos en un programa.
La teoría es necesaria para aprender a programar, pero leer y entender
programas reales es igual de importante.

Nuestro proyecto en este capítulo es construir un ((autómata)), un pequeño
programa que realiza una tarea en un ((mundo virtual)). Nuestro autómata
será un ((robot)) de entregas por correo que recoge y deja paquetes.

## VillaPradera

{{index "roads array"}}

El pueblo de ((VillaPradera)) no es muy grande. Este consiste de 11
lugares con 14 caminos entre ellos. Puede ser describido con este
array de caminos:


```{includeCode: true}
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

{{figure {url: "img/village2x.png", alt: "The village of Meadowfield"}}}

La red de caminos en el pueblo forma un _((grafo))_. Un grafo es una
colección de puntos (lugares en el pueblo) con líneas entre ellos
(caminos). Este grafo será el mundo por el que nuestro robot se movera.

{{index "roadGraph object"}}

El array de strings no es muy fácil de trabajar. En lo que estamos
interesados es en los destinos a los que podemos llegar desde un lugar
determinado. Vamos a convertir la lista de caminos en una estructura de datos
que, para cada lugar, nos diga a donde se pueda llegar desde allí.

```{includeCode: true}
function construirGrafo(bordes) {
  let grafo = Object.create(null);
  function añadirBorde(desde, hasta) {
    if (grafo[desde] == null) {
      grafo[desde] = [hasta];
    } else {
      grafo[desde].push(hasta);
    }
  }
  for (let [desde, hasta] of bordes.map(c => c.split("-"))) {
    añadirBorde(desde, hasta);
    añadirBorde(hasta, desde);
  }
  return grafo;
}

const grafoCamino = construirGrafo(roads);
```

Dado un conjunto de bordes, `construirGrafo` crea un objeto de mapa que, para
cada nodo, almacena un array de nodos conectados.

{{index "split method"}}

Utiliza el método `split` para ir de los strings de caminos, que tienen la
forma `"Comienzo-Final"`, a arrays de dos elementos que contienen el inicio y
el final como strings separados.

## La tarea

Nuestro ((robot)) se moverá por el pueblo. Hay paquetes
en varios lugares, cada uno dirigido a otro lugar. El robot tomara
paquetes cuando los encuentre y los entregara cuando llegue a
sus destinos.

El autómata debe decidir, en cada punto, a dónde ir después. Ha
finalizado su tarea cuando se han entregado todos los paquetes.

{{index simulation, "virtual world"}}

Para poder simular este proceso, debemos definir un mundo virtual que
pueda describirlo. Este modelo nos dice dónde está el robot y dónde estan
los paquetes. Cuando el robot ha decidido moverse a alguna parte, necesitamos
actualizar el modelo para reflejar la nueva situación.

Si estás pensando en términos de ((programación orientada a objetos)), tu
primer impulso podría ser comenzar a definir objetos para los diversos
elementos en el mundo. Una ((clase)) para el robot, una para un paquete,
tal vez una para los lugares. Estas podrían tener propiedades que describen
su ((estado)) actual, como la pila de paquetes en un lugar,
que podríamos cambiar al actualizar el mundo.

Esto está mal.

Al menos, usualmente lo esta. El hecho de que algo suena como un objeto
no significa automáticamente que debe ser un objeto en tu
programa. Escribir por reflejo las clases para cada concepto en tu
aplicación tiende a dejarte con una colección de objetos interconectados
donde cada uno tiene su propio estado interno y cambiante. Tales
programas a menudo son difíciles de entender y, por lo tanto, fáciles
de romper.

En lugar de eso, condensemos el ((estado)) del pueblo hasta el mínimo
conjunto de valores que lo definan. Está la ubicación actual del robot y
la colección de paquetes no entregados, cada uno de los cuales tiene una
ubicación actual y una dirección de destino. Eso es todo.

{{index "VillageState class", "persistent data structure"}}

Y mientras estamos en ello, hagámoslo de manera que no _cambiemos_ este
estado cuándo se mueva el robot, sino calcular un _nuevo_ estado para la
situación después del movimiento.

```{includeCode: true}
class EstadoPueblo {
  constructor(lugar, paquetes) {
    this.lugar = lugar;
    this.paquetes = paquetes;
  }

  mover(destino) {
    if (!grafoCamino[this.lugar].includes(destino)) {
      return this;
    } else {
      let paquetes = this.paquetes.map(p => {
        if (p.lugar != this.lugar) return p;
        return {lugar: destino, direccion: p.direccion};
      }).filter(p => p.lugar != p.direccion);
      return new EstadoPueblo(destino, paquetes);
    }
  }
}
```

En el método `mover` es donde ocurre la acción. Este primero verifica si
hay un camino que va del lugar actual al destino, y
si no, retorna el estado anterior, ya que este no es un movimiento válido.

{{index "map method", "filter method"}}

Luego crea un nuevo estado con el destino como el nuevo lugar del robot.
Pero también necesita crear un nuevo conjunto de paquetes—los paquetes que
el robot esta llevando (que están en el lugar actual del robot) necesitan de
moverse también al nuevo lugar. Y paquetes que están dirigidos al nuevo
lugar donde deben de ser entregados—es decir, deben de eliminarse del
conjunto de paquetes no entregados. La llamada a `map` se encarga de mover
los paquetes, y la llamada a `filter` hace la entrega.

Los objetos de paquete no se modifican cuando se mueven, sino que se vuelven
a crear. El método `movee` nos da un nuevo estado de aldea, pero deja el
viejo completamente intacto

```
let primero = new EstadoPueblo(
  "Oficina de Correos",
  [{lugar: "Oficina de Correos", direccion: "Casa de Alicia"}]
);
let siguiente = primero.mover("Casa de Alicia");

console.log(siguiente.lugar);
// → Casa de Alicia
console.log(siguiente.parcels);
// → []
console.log(primero.lugar);
// → Oficina de Correos
```

Mover hace que se entregue el paquete, y esto se refleja en
el próximo estado. Pero el estado inicial todavía describe la situación
donde el robot está en la oficina de correos y el paquete aun no ha
sido entregado.

## Datos persistentes

{{index "persistent data structure", mutability, "data structure"}}

Las estructuras de datos que no cambian se llaman _((inmutables))_ o
_persistentes_. Se comportan de manera muy similar a los strings y números
en que son quienes son, y se quedan así, en lugar de contener diferentes
cosas en diferentes momentos.

En JavaScript, casi todo _puede_ ser cambiado, así que trabajar con
valores que se supone que sean persistentes requieren cierta restricción.
Hay una función llamada `Object.freeze` ("Objeto.congelar") que cambia un objeto
de manera que escribir en sus propiedades sea ignorado. Podrías usar eso para
asegurarte de que tus objetos no cambien, si quieres ser cuidadoso. La congelación
requiere que la computadora haga un trabajo extra e ignorar actualizaciones
es probable que confunda a alguien tanto como para que hagan
lo incorrecto. Por lo general, prefiero simplemente decirle a la gente que un
determinado objeto no debe ser molestado, y espero que lo recuerden.

```
let objeto = Object.freeze({valor: 5});
objeto.valor = 10;
console.log(objeto.valor);
// → 5
```

Por qué me salgo de mi camino para no cambiar objetos cuando el lenguaje
obviamente está esperando que lo haga?

Porque me ayuda a entender mis programas. Esto es acerca de manejar la
complejidad nuevamente. Cuando los objetos en mi sistema son cosas fijas y
estables, puedo considerar las operaciones en ellos de forma aislada—moverse
a la casa de Alicia desde un estado de inicio siempre produce el mismo nuevo
estado. Cuando los objetos cambian con el tiempo, eso agrega una dimensión
completamente nueva de complejidad a este tipo de razonamiento.

Para un sistema pequeño como el que estamos construyendo en este capítulo,
podriamos manejar ese poco de complejidad adicional. Pero el límite
más importante sobre qué tipo de sistemas podemos construir es cuánto podemos
entender. Cualquier cosa que haga que tu código sea más fácil de entender hace
que sea posible construir un sistema más ambicioso.

Lamentablemente, aunque entender un sistema basado en estructuras de datos
persistentes es más fácil, _diseñar_ uno, especialmente cuando tu
lenguaje de programación no ayuda, puede ser un poco más difícil.
Buscaremos oportunidades para usar estructuras de datos persistentes en este
libro, pero también utilizaremos las modificables.

## Simulación

{{index simulation, "virtual world"}}

Un ((robot)) de entregas mira al mundo y decide en qué
dirección que quiere moverse. Como tal, podríamos decir que un robot es una
función que toma un objeto `EstadoPueblo` y retorna el nombre de un
lugar cercano.

{{index "runRobot function"}}

Ya que queremos que los robots sean capaces de recordar cosas, para que puedan
hacer y ejecutar planes, también les pasamos su memoria y les permitimos
retornar una nueva memoria. Por lo tanto, lo que retorna un robot es un objeto
que contiene tanto la dirección en la que quiere moverse como un
valor de memoria que se le sera regresado la próxima vez que se llame.

```{includeCode: true}
function correrRobot(estado, robot, memoria) {
  for (let turno = 0;; turno++) {
    if (estado.paquetes.length == 0) {
      console.log(`Listo en ${turno} turnos`);
      break;
    }
    let accion = robot(estado, memoria);
    estado = estado.mover(accion.direccion);
    memoria = accion.memoria;
    console.log(`Moverse a ${accion.direccion}`);
  }
}
```

Considera lo que un robot tiene que hacer para "resolver" un estado dado. Debe
recoger todos los paquetes visitando cada ubicación que tenga un paquete, y
entregarlos visitando cada lugar al que se dirige un paquete,
pero solo después de recoger el paquete.

Cuál es la estrategia más estúpida que podría funcionar? El robot podría
simplemente caminar hacia una dirección aleatoria en cada vuelta. Eso significa, con
gran probabilidad, que eventualmente se encontrara con todos los paquetes, y luego
también en algún momento llegara a todos los lugares donde estos deben
ser entregados.

{{index "randomPick function", "randomRobot function"}}

Aqui esta como se podria ver eso:

```{includeCode: true}
function eleccionAleatoria(array) {
  let eleccion = Math.floor(Math.random() * array.length);
  return array[eleccion];
}

function robotAleatorio(estado) {
  return {direccion: eleccionAleatoria(grafoCamino[estado.lugar])};
}
```

{{index "Math.random function", "Math.floor function", array}}

Recuerda que `Math.random ()` retorna un número entre cero y uno,
pero siempre debajo de uno. Multiplicar dicho número por la longitud de un
array y luego aplicarle `Math.floor` nos da un índice aleatorio para
el array.

Como este robot no necesita recordar nada, ignora su
segundo argumento (recuerda que puedes llamar a las funciones en JavaScript con
argumentos adicionales sin efectos negativos) y omite la propiedad `memoria`
en su objeto retornado.

Para poner en funcionamiento este sofisticado robot, primero necesitaremos una
forma de crear un nuevo estado con algunos paquetes. Un método estático
(escrito aquí al agregar directamente una propiedad al constructor) es un
buen lugar para poner esa funcionalidad.

```{includeCode: true}
EstadoPueblo.aleatorio = function(numeroDePaquetes = 5) {
  let paquetes = [];
  for (let i = 0; i < numeroDePaquetes; i++) {
    let direccion = eleccionAleatoria(Object.keys(grafoCamino));
    let lugar;
    do {
      lugar = eleccionAleatoria(Object.keys(grafoCamino));
    } while (lugar == direccion);
    paquetes.push({lugar, direccion});
  }
  return new EstadoPueblo("Oficina de Correos", paquetes);
};
```

{{index "do loop"}}

No queremos paquetes que sean enviados desde el mismo lugar al que están
dirigidos. Por esta razón, el bucle `do` sigue seleccionando nuevos
lugares cuando obtenga uno que sea igual a la dirección.

Comencemos un mundo virtual.

```{test: no}
correrRobot(EstadoPueblo.aleatorio(), robotAleatorio);
// → Moverse a Mercado
// → Moverse a Ayuntamiento
// → …
// → Listo en 63 turnos
```

Le toma muchas vueltas al robot para entregar los paquetes, porque este
no está planeando muy bien. Nos ocuparemos de eso pronto.

{{if interactive

Para una perspectiva más agradable de la simulación, puedes usar la
función `runRobotAnimation` que está disponible en el entorno de programación
de este capítulo. Esta ejecuta la simulación, pero en lugar de
mostrar texto, muestra al robot moviéndose por el mapa del pueblo.

```{test: no}
runRobotAnimation(EstadoPueblo.aleatorio(), robotAleatorio);
```

La forma en la que `runRobotAnimation` esta implementada seguirá siendo un
misterio por ahora, pero después de que hayas leído [capítulos mas avanzados](dom)
de este libro, que discuten la integración de JavaScript en los navegadores web,
podrás adivinar cómo esta funciona.

if}}

## La ruta del camión de correos

{{index "mailRoute array"}}

Deberíamos poder hacer algo mucho mejor que el ((robot)) aleatorio. Una
mejora fácil sería tomar una pista de la forma en que como funciona la entrega
de correos en el mundo real. Si encontramos una ruta que pasa por todos los
lugares en el pueblo, el robot podría ejecutar esa ruta dos veces,
y en ese punto esta garantizado que ha entregado todos los paquetes.
Aquí hay una de esas rutas (comenzando desde la Oficina de Correos).

```{includeCode: true}
const rutaCorreo = [
  "Casa de Alicia", "Cabaña", "Casa de Alicia", "Casa de Bob",
  "Ayuntamiento", "Casa de Daria", "Casa de Ernie",
  "GCasa de Grete", "Tienda", "Casa de Grete", "Granja",
  "Mercado", "Oficina de Correos"
];
```

{{index "routeRobot function"}}

Para implementar el robot que siga la ruta, necesitaremos hacer uso de la
memoria del robot. El robot mantiene el resto de su ruta en su memoria y
deja caer el primer elemento en cada vuelta.

```{includeCode: true}
function robotRuta(estado, memoria) {
  if (memoria.length == 0) {
    memoria = rutaCorreo;
  }
  return {direction: memoria[0], memoria: memoria.slice(1)};
}
```

Este robot ya es mucho más rápido. Tomará un máximo de 26 turnos
(dos veces la ruta de 13 pasos), pero generalmente seran menos.

{{if interactive

```{test: no}
runRobotAnimation(EstadoPueblo.aleatorio(), robotRuta, []);
```

if}}

## Búsqueda de rutas

Aún así, realmente no llamaría seguir ciegamente una ruta fija
comportamiento inteligente. El ((robot)) podría funcionar más eficientemente si
ajustara su comportamiento al trabajo real que necesita hacerse.

{{index pathfinding}}

Para hacer eso, tiene que ser capaz de avanzar deliberadamente hacia un
determinado paquete, o hacia la ubicación donde se debe entregar un paquete.
Al hacer eso, incluso cuando el objetivo esté a más de un movimiento de
distancia, requiere algún tipo de función de búsqueda de ruta.

El problema de encontrar una ruta a través de un ((grafo)) es un típico
_((problema de búsqueda))_. Podemos decir si una solución dada (una ruta)
es una solución válida, pero no podemos calcular directamente la solución de
la misma manera que podríamos para 2 + 2. En cambio, tenemos que seguir
creando soluciones potenciales hasta que encontremos una que funcione.

El número de rutas posibles a través de un grafo es infinito. Pero
cuando buscamos una ruta de _A_ a _B_, solo estamos interesados ​​en aquellas
que comienzan en _A_. Tampoco nos importan las rutas que visitan
el mismo lugar dos veces, definitivamente esa no es la ruta más eficiente
en cualquier sitio. Entonces eso reduce la cantidad de rutas que el
buscador de rutas tiene que considerar.

De hecho, estamos más interesados ​​en la ruta _mas corta_. Entonces queremos
asegurarnos de mirar las rutas cortas antes de mirar las más largas. Un
buen enfoque sería "crecer" las rutas desde el punto de partida,
explorando cada lugar accesible que aún no ha sido visitado, hasta que
una ruta alcanze la meta. De esa forma, solo exploraremos las rutas que son
potencialmente interesantes, y encontremos la ruta más corta (o una de las
rutas más cortas, si hay más de una) a la meta.

{{index "findRoute function"}}

{{id findRoute}}

Aquí hay una función que hace esto:

```{includeCode: true}
function encontrarRuta(grafo, desde, hasta) {
  let trabajo = [{donde: desde, ruta: []}];
  for (let i = 0; i < trabajo.length; i++) {
    let {donde, ruta} = trabajo[i];
    for (let lugar of grafo[donde]) {
      if (lugar == hasta) return ruta.concat(lugar);
      if (!trabajo.some(w => w.donde == lugar)) {
        trabajo.push({donde: lugar, ruta: ruta.concat(lugar)});
      }
    }
  }
}
```

La exploración tiene que hacerse en el orden correcto—los lugares que fueron
alcanzados primero deben ser explorados primero. No podemos explorar de inmediato
un lugar apenas lo alcanzamos, porque eso significaría que los lugares alcanzados
_desde allí_ también se explorarían de inmediato, y así sucesivamente, incluso
aunque puedan haber otros caminos más cortos que aún no han sido
explorados.

Por lo tanto, la función mantiene una _((lista de trabajo))_. Esta es un array de
lugares que deberían explorarse a continuación, junto con la ruta que nos llevó
ahí. Esta comienza solo con la posición de inicio y una ruta vacía.

La búsqueda luego opera tomando el siguiente elemento en la lista y
explorando eso, lo que significa que todos los caminos que van desde ese lugar
son mirados. Si uno de ellos es el objetivo, una ruta final puede ser
retornada. De lo contrario, si no hemos visto este lugar antes, un nuevo
elemento se agrega a la lista. Si lo hemos visto antes, ya que
estamos buscando primero rutas cortas, hemos encontrado una ruta más larga
a ese lugar o una precisamente tan larga como la existente, y
no necesitamos explorarla.

Puedes imaginar esto visualmente como una red de rutas conocidas que se arrastran
desde el lugar de inicio, creciendo uniformemente hacia todos los lados (pero nunca
enredándose de vuelta a si misma). Tan pronto como el primer hilo llegue a
la ubicación objetivo, ese hilo se remonta al comienzo, dándonos asi nuestra
ruta.

{{index "connected graph"}}

Nuestro código no maneja la situación donde no hay más elementos de trabajo
en la lista de trabajo, porque sabemos que nuestro gráfico está
_conectado_, lo que significa que se puede llegar a todos los lugares
desde todos los otros lugares. Siempre podremos encontrar una ruta entre
dos puntos, y la búsqueda no puede fallar.

```{includeCode: true}
function robotOrientadoAMetas({lugar, paquetes}, ruta) {
  if (ruta.length == 0) {
    let paquete = paquetes[0];
    if (paquete.lugar != lugar) {
      ruta = encontrarRuta(grafoCamino, lugar, paquete.lugar);
    } else {
      ruta = encontrarRuta(grafoCamino, lugar, paquete.direccion);
    }
  }
  return {direccion: ruta[0], memoria: ruta.slice(1)};
}
```

{{index "goalOrientedRobot function"}}

Este robot usa su valor de memoria como una lista de instrucciones para moverse,
como el robot que sigue la ruta. Siempre que esa lista esté vacía, este
tiene que descubrir qué hacer a continuación. Toma el primer paquete no entregado
en el conjunto y, si ese paquete no se ha recogido aún, traza una
ruta hacia el. Si el paquete _ha_ sido recogido, todavía debe ser
entregado, por lo que el robot crea una ruta hacia la dirección de entrega
en su lugar.

{{if interactive

Veamos cómo le va.

```{test: no, startCode: true}
runRobotAnimation(EstadoPueblo.aleatorio(),
                  robotOrientadoAMetas, []);
```

if}}

Este robot generalmente termina la tarea de entregar 5 paquetes en
16 turnos aproximadamente. Un poco mejor que `robotRuta`,
pero definitivamente no es óptimo.

## Ejercicios

### Midiendo un robot

{{index "measuring a robot (exercise)", testing, automation, "compareRobots function"}}

Es difícil comparar objetivamente ((robot))s simplemente dejándolos resolver
algunos escenarios. Tal vez un robot acaba de conseguir tareas más fáciles, o
el tipo de tareas en las que es bueno, mientras que el otro no.

Escribe una función `compararRobots` que toma dos robots (y su
memoria de inicio). Debe generar 100 tareas y dejar que cada uno de
los robots resuelvan cada una de estas tareas. Cuando terminen,
debería generar el promedio de pasos que cada robot tomó por tarea.

En favor de lo que es justo, asegúrate de la misma tarea a ambos
robots, en lugar de generar diferentes tareas por robot.

{{if interactive

```{test: no}
function compararRobots(robot1, memoria1, robot2, memoria2) {
  // Tu código aqui
}

compararRobots(robotRuta, [], robotOrientadoAMetas, []);
```
if}}

{{hint

{{index "measuring a robot (exercise)", "runRobot function"}}

Tendrás que escribir una variante de la función `correrRobot` que,
en lugar de registrar los eventos en la consola, retorne el número de
pasos que le tomó al robot completar la tarea.

Tu función de medición puede, en un ciclo, generar nuevos estados y
contar los pasos que lleva cada uno de los robots. Cuando has generado
suficientes mediciones, puedes usar `console.log` para mostrar el
promedio de cada robot, que es la cantidad total de pasos tomados dividido por
el número de mediciones

hint}}

### Eficiencia del robot

{{index "robot efficiency (exercise)"}}

Puedes escribir un robot que termine la tarea de entrega más rápido que
`robotOrientadoAMetas`? Si observas el comportamiento de ese robot, qué
obviamente cosas estúpidas este hace? Cómo podrían mejorarse?

Si resolviste el ejercicio anterior, es posible que desees utilizar tu
función `compararRobots` para verificar si has mejorado al robot.

{{if interactive

```{test: no}
// Tu código aqui

runRobotAnimation(EstadoPueblo.aleatorio(), tuRobot, memoria);
```

if}}

{{hint

{{index "robot efficiency (exercise)"}}

La principal limitación de `robotOrientadoAMetas` es que solo considera
un paquete a la vez. A menudo caminará de ida y vuelta por el
pueblo porque el paquete que resulta estar mirando sucede que esta
en el otro lado del mapa, incluso si hay otros mucho más cerca.

Una posible solución sería calcular rutas para todos los paquetes, y
luego tomar la más corta. Se pueden obtener incluso mejores resultados, si
hay múltiples rutas más cortas, al ir prefiriendo las que van a
recoger un paquete en lugar de entregar un paquete.

hint}}

### Conjunto persistente

{{index "persistent group (exercise)", "persistent data structure", "Set class", "set (data structure)", "Group class", "PGroup class"}}

La mayoría de las estructuras de datos proporcionadas en un entorno de
JavaScript estándar no son muy adecuadas para usos persistentes. Los arrays
tienen los métodos `slice` y `concat`, que nos permiten fácilmente crear nuevos
arrays sin dañar al anterior. Pero `Set`, por ejemplo, no tiene métodos para
crear un nuevo conjunto con un elemento agregado o eliminado.

Escribe una nueva clase `ConjuntoP`, similar a la clase `Conjunto` del [Capitulo 6](objeto#grupos), que almacena un conjunto de valores. Como `Grupo`, tiene
métodos `añadir`, `eliminar`, y `tiene`.

Su método `añadir`, sin embargo, debería retornar una _nueva_ instancia de
`ConjuntoP` con el miembro dado agregado, y dejar la instancia anterior sin cambios.
Del mismo modo, `eliminar` crea una nueva instancia sin un miembro dado.

La clase debería funcionar para valores de cualquier tipo, no solo strings. Esta
_no_ tiene que ser eficiente cuando se usa con grandes cantidades de
valores.

El ((constructor)) no deberia ser parte de la interfaz de la clase
(aunque definitivamente querrás usarlo internamente). En cambio, allí
hay una instancia vacía, `ConjuntoP.vacio`, que se puede usar como un valor
de inicio.

{{index singleton}}

Por qué solo necesitas un valor `ConjuntoP.vacio`, en lugar de tener una
función que crea un nuevo mapa vacío cada vez?

{{if interactive

```{test: no}
class ConjuntoP {
  // Tu código aqui
}

let a = ConjuntoP.vacio.añadir("a");
let ab = a.añadir("b");
let b = ab.eliminar("a");

console.log(b.tiene("b"));
// → true
console.log(a.tiene("b"));
// → false
console.log(b.tiene("a"));
// → false
```

if}}

{{hint

{{index "persistent map (exercise)", "Set class", array, "PGroup class"}}

La forma más conveniente de representar el conjunto de valores de miembro
sigue siendo un array, ya que son fáciles de copiar.

{{index "concat method", "filter method"}}

Cuando se agrega un valor al grupo, puedes crear un nuevo grupo con una
copia del array original que tiene el valor agregado (por ejemplo, usando
`concat`). Cuando se borra un valor, lo filtra afuera del array.

El ((constructor)) de la clase puede tomar un array como argumento, y
almacenarlo como la (única) propiedad de la instancia. Este array nunca es
actualizado.

{{index "static method"}}

Para agregar una propiedad (`vacio`) a un constructor que no sea un método,
tienes que agregarlo al constructor después de la definición de la clase, como
una propiedad regular.

Solo necesita una instancia `vacio` porque todos los conjuntos vacíos son
iguales y las instancias de la clase no cambian. Puedes crear muchos
conjuntos diferentes de ese único conjunto vacío sin afectarlo.

hint}}
