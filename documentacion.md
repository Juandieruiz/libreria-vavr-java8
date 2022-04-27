# Librería VAVR - Documentación

###### Versión en Español de la Documentación de D.Dietrich, R. Winkler

---

# 1. Introducción

Vavr (anteriormente llamado Javaslang) es una biblioteca funcional para Java 8+ que proporciona tipos de datos persistentes y estructuras de control funcional.

## 1.1. Estructuras de datos funcionales en Java 8 con Vavr

[Las lambdas](https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html) de Java 8 nos permiten crear maravillosas API. Aumentan increíblemente la expresividad del lenguaje.



[Vavr](https://www.vavr.io/) aprovechó las lambdas para crear varias características nuevas basadas en patrones funcionales. Uno de ellos es una biblioteca de colección funcional que pretende ser un reemplazo para las colecciones estándar de Java.

<img title="" src="file:///C:/Users/jdgomez/AppData/Roaming/marktext/images/2022-04-27-09-44-36-image.png" alt="" width="665" data-align="center">

*(Esto es solo una vista de pájaro, encontrará una versión legible por humanos a continuación).*

## 1.2 Programación funcional

Antes de profundizar en los detalles sobre las estructuras de datos, quiero hablar sobre algunos conceptos básicos. Esto dejará en claro por qué creé Vavr y específicamente nuevas colecciones de Java.

## 1.2.1 Efectos secundarios

Las aplicaciones Java suelen tener muchos [efectos secundarios](https://en.wikipedia.org/wiki/Side_effect_(computer_science)) . Mutan algún tipo de estado, tal vez el mundo exterior. Los efectos secundarios comunes son cambiar objetos o variables *en su lugar* , imprimir en la consola, escribir en un archivo de registro o en una base de datos. Los efectos secundarios se consideran *dañinos* si afectan la semántica de nuestro programa de una manera no deseada.

Por ejemplo, si una función arroja una excepción y esta excepción se *interpreta* , se considera un efecto secundario que *afecta a nuestro programa* . Además [, las excepciones son como sentencias goto no locales](http://c2.com/cgi/wiki?DontUseExceptionsForFlowControl) . Rompen el flujo de control normal. Sin embargo, las aplicaciones del mundo real tienen efectos secundarios.

```java
int dividir(int dividendo, int divisor){
    // lanza si el divisor es cero
    return dividendo/ divisor;
}
```

En un entorno funcional, estamos en una situación favorable para encapsular el efecto secundario en un Try:

```java
// = Si va Bien(resultado) si Falla (excepción)
Try<Integer> dividir(Integer dividend, Integer divisor) {
    return Try.of(() -> dividend / divisor);
}
```

Esta versión de divide ya no arroja ninguna excepción. Hicimos explícito el posible fallo utilizando el tipo Try.

## 1.2.2. Transparecia Referencial

Una función, o más generalmente una expresión, se llama [referencialmente transparente](https://en.wikipedia.org/wiki/Referential_transparency) si una llamada puede ser reemplazada por su valor sin afectar el comportamiento del programa. En pocas palabras, dada la misma entrada, la salida es siempre la misma.

```java
// no referencialmente transparente
Math.random();

// referencialmente transparente
Math.max(1,2);
```

Una función se llama [pura](https://en.wikipedia.org/wiki/Pure_function) si todas las expresiones involucradas son referencialmente transparentes. Una aplicación compuesta de funciones puras probablemente *solo funcionará* si se compila. Somos capaces de razonar al respecto. Las pruebas unitarias son fáciles de escribir y la depuración se convierte en una reliquia del pasado.

## 1.2.3. Pensando en Valores

Rich Hickey, el creador de Clojure, dio una gran charla sobre [El valor de los valores](https://www.youtube.com/watch?v=-6BsiVyC1kM). 

Los valores más interesantes son los valores [inmutables](https://en.wikipedia.org/wiki/Immutable_object) . La razón principal es que los valores inmutables

- son intrínsecamente seguros para subprocesos y, por lo tanto, no es necesario sincronizarlos

- son estables con respecto a *equals* y *hashCode* y, por lo tanto, son claves hash confiables

- no necesita ser clonado

- comportarse con seguridad de tipos cuando se usa en conversiones covariantes no verificadas (específico de Java)
  La clave para un mejor Java es usar *valores inmutables* emparejados con *funciones referencialmente transparentes* .

Vavr proporciona los [controles](http://static.javadoc.io/io.vavr/vavr/0.10.4/io/vavr/control/package-summary.html) y [colecciones](https://static.javadoc.io/io.vavr/vavr/0.10.4/io/vavr/collection/package-summary.html) necesarios para lograr este objetivo en la programación Java diaria.

### 1.3. Estructuras de datos en pocas palabras

La biblioteca de colecciones de Vavr se compone de un rico conjunto de estructuras de datos funcionales construidas sobre lambdas. La única interfaz que comparten con las colecciones originales de Java es iterable. La razón principal es que los métodos mutadores de las interfaces de colección de Java no devuelven un objeto del tipo de colección subyacente.

Veremos por qué esto es tan esencial echando un vistazo a los diferentes tipos de estructuras de datos.

#### 1.3.1. Estructuras de datos mutables

Java es un lenguaje de programación orientado a objetos. Encapsulamos el estado en objetos para lograr la ocultación de datos y proporcionamos métodos mutadores para controlar el estado. El [marco de colecciones de Java (JCF)](https://en.wikipedia.org/wiki/Java_collections_framework) se basa en esta idea.

```
interface Collection<E> {
    // removes all elements from this collection
    void clear();
}
```

Hoy entiendo un tipo de retorno *vacío* como un olor. Es evidencia [de que se producen efectos secundarios](https://en.wikipedia.org/wiki/Side_effect_(computer_science)) , el estado está mutado. *El estado mutable compartido* es una fuente importante de fallas, no solo en un entorno concurrente.

#### 1.3.2. Estructuras de datos inmutables

[Las estructuras de datos inmutables](https://en.wikipedia.org/wiki/Immutable_object) no se pueden modificar después de su creación. En el contexto de Java, se utilizan ampliamente en forma de contenedores de colección.

```
List<String> list = Collections.unmodifiableList(otherList);

// Boom!
list.add("why not?");
```

Hay varias bibliotecas que nos proporcionan métodos de utilidad similares. El resultado es siempre una vista no modificable de la colección específica. Por lo general, se lanzará en tiempo de ejecución cuando llamamos a un método mutador.

#### 1.3.3. Estructuras de datos persistentes

Una [estructura de datos persistente](https://en.wikipedia.org/wiki/Persistent_data_structure) conserva la versión anterior de sí misma cuando se modifica y, por lo tanto, es *efectivamente* inmutable. Las estructuras de datos totalmente persistentes permiten actualizaciones y consultas en cualquier versión.

Muchas operaciones solo realizan pequeños cambios. Simplemente copiar la versión anterior no sería eficiente. Para ahorrar tiempo y memoria, es fundamental identificar similitudes entre dos versiones y compartir la mayor cantidad de datos posible.

Este modelo no impone ningún detalle de implementación. Aquí entran en juego las estructuras de datos funcionales.

### 1.4. Estructuras de datos funcionales

También conocidas como estructuras de datos [*puramente* funcionales](https://en.wikipedia.org/wiki/Purely_functional) , son *inmutables* y *persistentes* . Los métodos de las estructuras de datos funcionales son *referencialmente transparentes* .

Vavr presenta una amplia gama de las estructuras de datos funcionales más utilizadas. Los siguientes ejemplos se explican en profundidad.

#### 1.4.1. Lista enlazada

Una de las estructuras de datos funcionales más populares y también más simples es la [Lista enlazada (individualmente)](https://en.wikipedia.org/wiki/Linked_list) . Tiene un elemento de *cabeza* y una lista de *cola* . Una Lista enlazada se comporta como una Pila que sigue el método [LIFO (último en entrar, primero en salir) .](https://en.wikipedia.org/wiki/Stack_(abstract_data_type))

En [Vavr](http://vavr.io/) instanciamos una Lista como esta:

```
// = List(1, 2, 3)
List<Integer> list1 = List.of(1, 2, 3);
```

Cada uno de los elementos de Lista forma un nodo de Lista separado. La cola del último elemento es Nil, la Lista vacía.

![Lista 1](https://docs.vavr.io/images/list1.png?w=660)

Esto nos permite compartir elementos entre diferentes versiones de la Lista.

```
// = List(0, 2, 3)
List<Integer> list2 = list1.tail().prepend(0);
```

El nuevo elemento de cabeza 0 está *vinculado* a la cola de la Lista original. La Lista original permanece sin modificaciones.

![Lista 2](https://docs.vavr.io/images/list2.png?w=660)

Estas operaciones se realizan en tiempo constante, es decir, son independientes del tamaño de la Lista. La mayoría de las otras operaciones toman tiempo lineal. En Vavr esto se expresa mediante la interfaz LinearSeq, que quizás ya conozcamos de Scala.

Si necesitamos estructuras de datos que se puedan consultar en tiempo constante, Vavr ofrece Array y Vector. Ambos tienen capacidades [de acceso aleatorio .](https://en.wikipedia.org/wiki/Random_access)

El tipo Array está respaldado por una matriz de objetos Java. Las operaciones de inserción y eliminación toman un tiempo lineal. Vector está entre Array y List. Funciona bien en ambas áreas, acceso aleatorio y modificación.

De hecho, la Lista vinculada también se puede utilizar para implementar una estructura de datos de cola.

#### 1.4.2. Cola

Se puede implementar una Cola funcional muy eficiente basada en dos Listas enlazadas. La Lista *frontal contiene los elementos que están fuera de la* *cola* , la Lista *posterior contiene los elementos que están en la* *cola* . Ambas operaciones enqueue y dequeue se realizan en O(1).

```
Queue<Integer> queue = Queue.of(1, 2, 3)
                            .enqueue(4)
                            .enqueue(5);
```

La cola inicial se crea a partir de tres elementos. Dos elementos se ponen en cola en la lista posterior.

![cola 1](https://docs.vavr.io/images/queue1.png?w=660)

Si la Lista frontal se queda sin elementos al quitar la cola, la Lista posterior se invierte y se convierte en la nueva Lista frontal.

![cola 2](https://docs.vavr.io/images/queue2.png?w=660)

Al quitar la cola de un elemento, obtenemos un par del primer elemento y la Cola restante. Es necesario devolver la nueva versión de la cola porque las estructuras de datos funcionales son inmutables y persistentes. La cola original no se ve afectada.

```
Queue<Integer> queue = Queue.of(1, 2, 3);

// = (1, Queue(2, 3))
Tuple2<Integer, Queue<Integer>> dequeued =
        queue.dequeue();
```

¿Qué sucede cuando la cola está vacía? Entonces dequeue() lanzará una NoSuchElementException. Para hacerlo de *manera funcional* , preferiríamos esperar un resultado opcional.

```
// = Some((1, Queue()))
Queue.of(1).dequeueOption();

// = None
Queue.empty().dequeueOption();
```

Un resultado opcional puede seguir procesándose, independientemente de si está vacío o no.

```
// = Queue(1)
Queue<Integer> queue = Queue.of(1);

// = Some((1, Queue()))
Option<Tuple2<Integer, Queue<Integer>>> dequeued =
        queue.dequeueOption();

// = Some(1)
Option<Integer> element = dequeued.map(Tuple2::_1);

// = Some(Queue())
Option<Queue<Integer>> remaining =
        dequeued.map(Tuple2::_2);
```

#### 1.4.3. Conjunto ordenado

Los conjuntos ordenados son estructuras de datos que se utilizan con más frecuencia que las colas. Usamos árboles de búsqueda binarios para modelarlos de manera funcional. Estos árboles constan de nodos con hasta dos hijos y valores en cada nodo.

Construimos árboles de búsqueda binarios en presencia de un ordenamiento, representado por un elemento Comparador. Todos los valores del subárbol izquierdo de cualquier nodo dado son estrictamente menores que el valor del nodo dado. Todos los valores del subárbol derecho son estrictamente mayores.

```
// = TreeSet(1, 2, 3, 4, 6, 7, 8)
SortedSet<Integer> xs = TreeSet.of(6, 1, 3, 2, 4, 7, 8);
```

![Árbol binario 1](https://docs.vavr.io/images/binarytree1.png?w=660)

Las búsquedas en dichos árboles se ejecutan en tiempo O (log n). Comenzamos la búsqueda en la raíz y decidimos si encontramos el elemento. Debido a la ordenación total de los valores, sabemos dónde buscar a continuación, en la rama izquierda o derecha del árbol actual.

```
// = TreeSet(1, 2, 3);
SortedSet<Integer> set = TreeSet.of(2, 3, 1, 2);

// = TreeSet(3, 2, 1);
Comparator<Integer> c = (a, b) -> b - a;
SortedSet<Integer> reversed = TreeSet.of(c, 2, 3, 1, 2);
```

La mayoría de las operaciones de árbol son inherentemente [recursivas](https://en.wikipedia.org/wiki/Recursion) . La función de inserción se comporta de manera similar a la función de búsqueda. Cuando se llega al final de una ruta de búsqueda, se crea un nuevo nodo y se reconstruye toda la ruta hasta la raíz. Se hace referencia a los nodos secundarios existentes siempre que sea posible. Por lo tanto, la operación de inserción requiere tiempo y espacio O(log n).

```
// = TreeSet(1, 2, 3, 4, 5, 6, 7, 8)
SortedSet<Integer> ys = xs.add(5);
```

![Árbol binario 2](https://docs.vavr.io/images/binarytree2.png?w=660)

Para mantener las características de rendimiento de un árbol de búsqueda binario, es necesario mantenerlo equilibrado. Todos los caminos desde la raíz hasta una hoja deben tener aproximadamente la misma longitud.

En Vavr implementamos un árbol de búsqueda binario basado en un árbol [rojo/negro](https://en.wikipedia.org/wiki/Red%E2%80%93black_tree) . Utiliza una estrategia de coloración específica para mantener el árbol equilibrado en inserciones y eliminaciones. Para leer más sobre este tema, consulte el libro [Estructuras de datos puramente funcionales](http://www.amazon.com/Purely-Functional-Structures-Chris-Okasaki/dp/0521663504) de Chris Okasaki.

### 1.5. Estado de las colecciones

Generalmente estamos observando una convergencia de lenguajes de programación. Las buenas características lo hacen, otras desaparecen. Pero Java es diferente, está obligado a ser compatible con versiones anteriores para siempre. Eso es una fortaleza pero también frena la evolución.

Lambda acercó más a Java y Scala, pero siguen siendo muy diferentes. Martin Odersky, el creador de Scala, mencionó recientemente en su discurso de apertura de [BDSBTB 2015](https://www.youtube.com/watch?v=NW5h8d_ZyOs) el estado de las colecciones de Java 8.

Describió el flujo de Java como una forma elegante de un iterador. La API de flujo de Java 8 es un ejemplo de una colección *levantada* . Lo que hace es *definir* un cómputo y *vincularlo* a una colección específica en otro paso explícito.

```
// i + 1
i.prepareForAddition()
 .add(1)
 .mapBackToInteger(Mappers.toInteger())
```

Así funciona la nueva API de Java 8 Stream. Es una capa computacional por encima de las conocidas colecciones de Java.

```
// = ["1", "2", "3"] in Java 8
Arrays.asList(1, 2, 3)
      .stream()
      .map(Object::toString)
      .collect(Collectors.toList())
```

Vavr está muy inspirado en Scala. Así es como debería haber sido el ejemplo anterior en Java 8.

```
// = Stream("1", "2", "3") in Vavr
Stream.of(1, 2, 3).map(Object::toString)
```

Durante el último año pusimos mucho esfuerzo en implementar la biblioteca de la colección Vavr. Comprende los tipos de colección más utilizados.

#### 1.5.1. secuencia

Comenzamos nuestro viaje implementando tipos secuenciales. Ya describimos la Lista enlazada arriba. Stream, una lista enlazada perezosa, siguió. Nos permite procesar posiblemente infinitas secuencias largas de elementos.

![secuencia](https://docs.vavr.io/images/collections-seq.png?w=660)

Todas las colecciones son iterables y, por lo tanto, podrían usarse en declaraciones for mejoradas.

```
for (String s : List.of("Java", "Advent")) {
    // side effects and mutation
}
```

Podríamos lograr lo mismo internalizando el ciclo e inyectando el comportamiento usando una lambda.

```
List.of("Java", "Advent").forEach(s -> {
    // side effects and mutation
});
```

De todos modos, como vimos anteriormente, preferimos las expresiones que devuelven un valor a las declaraciones que no devuelven nada. Al observar un ejemplo simple, pronto reconoceremos que las declaraciones agregan ruido y dividen lo que pertenece.

```
String join(String... words) {
    StringBuilder builder = new StringBuilder();
    for(String s : words) {
        if (builder.length() > 0) {
            builder.append(", ");
        }
        builder.append(s);
    }
    return builder.toString();
}
```

Las colecciones de Vavr nos brindan muchas funciones para operar en los elementos subyacentes. Esto nos permite expresar las cosas de una manera muy concisa.

```
String join(String... words) {
    return List.of(words)
               .intersperse(", ")
               .foldLeft(new StringBuilder(), StringBuilder::append)
               .toString();
}
```

La mayoría de los objetivos se pueden lograr de varias maneras usando Vavr. Aquí redujimos todo el cuerpo del método a llamadas de función fluidas en una instancia de List. Incluso podríamos eliminar todo el método y usar directamente nuestra Lista para obtener el resultado del cálculo.

```
List.of(words).mkString(", ");
```

En una aplicación del mundo real, ahora podemos reducir drásticamente la cantidad de líneas de código y, por lo tanto, reducir el riesgo de errores.

#### 1.5.2. Conjunto y mapa

Las secuencias son geniales. Pero para estar completa, una biblioteca de colección también necesita diferentes tipos de Conjuntos y Mapas.

![Conjunto y mapa](https://docs.vavr.io/images/collections-set-map.png?w=660)

Describimos cómo modelar conjuntos ordenados con estructuras de árbol binario. Un mapa ordenado no es más que un conjunto ordenado que contiene pares clave-valor y tiene un orden para las claves.

La implementación de HashMap está respaldada por [Hash Array Mapped Trie (HAMT)](http://lampwww.epfl.ch/papers/idealhashtrees.pdf) . En consecuencia, el HashSet está respaldado por un HAMT que contiene pares clave-clave.

Nuestro mapa *no* tiene un tipo de entrada especial para representar pares clave-valor. En su lugar, usamos Tuple2, que ya forma parte de Vavr. Se enumeran los campos de una Tupla.

```
// = (1, "A")
Tuple2<Integer, String> entry = Tuple.of(1, "A");

Integer key = entry._1;
String value = entry._2;
```

Los mapas y las tuplas se utilizan en Vavr. Las tuplas son inevitables para manejar tipos de devolución de valores múltiples de manera general.

```
// = HashMap((0, List(2, 4)), (1, List(1, 3)))
List.of(1, 2, 3, 4).groupBy(i -> i % 2);

// = List((a, 0), (b, 1), (c, 2))
List.of('a', 'b', 'c').zipWithIndex();
```

En Vavr, exploramos y probamos nuestra biblioteca implementando los [99 problemas de Euler](https://projecteuler.net/archives) . Es una gran prueba de concepto. Por favor, no dude en enviar solicitudes de extracción.

## 2. Primeros pasos

Los proyectos que incluyen Vavr deben apuntar a Java 1.8 como mínimo.

El .jar está disponible en [Maven Central](http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22io.vavr%22%20a%3A%22vavr%22) .

### 2.1. gradle

```
dependencies {
    compile "io.vavr:vavr:0.10.4"
}
```

Gradle 7+

```
dependencies {
    implementation "io.vavr:vavr:0.10.4"
}
```

### 2.2. Experto

```
<dependencies>
    <dependency>
        <groupId>io.vavr</groupId>
        <artifactId>vavr</artifactId>
        <version>0.10.4</version>
    </dependency>
</dependencies>
```

### 2.3. Ser único

Debido a que Vavr *no* depende de ninguna biblioteca (aparte de la JVM), puede agregarlo fácilmente como .jar independiente a su classpath.

### 2.4. Instantáneas

Las versiones de desarrollador se pueden encontrar [aquí](https://oss.sonatype.org/content/repositories/snapshots/io/vavr/vavr) .

#### 2.4.1. gradle

Agregue el repositorio de instantáneas adicional a su `build.gradle`:

```
repositories {
    (...)
    maven { url "https://oss.sonatype.org/content/repositories/snapshots" }
}
```

#### 2.4.2. Experto

Asegúrese de que su `~/.m2/settings.xml`cuenta contenga lo siguiente:

```
<profiles>
    <profile>
        <id>allow-snapshots</id>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
        <repositories>
            <repository>
                <id>snapshots-repo</id>
                <url>https://oss.sonatype.org/content/repositories/snapshots</url>
                <releases>
                    <enabled>false</enabled>
                </releases>
                <snapshots>
                    <enabled>true</enabled>
                </snapshots>
            </repository>
        </repositories>
    </profile>
</profiles>
```

## 3. Guía de uso

Vavr viene con representaciones bien diseñadas de algunos de los tipos más básicos que aparentemente faltan o son rudimentarios en Java: `Tuple`, `Value`y `λ`.  
En Vavr, todo se basa en estos tres bloques de construcción básicos:

![Descripción general de Vavr](https://docs.vavr.io/images/vavr-overview.png)

### 3.1. tuplas

A Java le falta una noción general de tuplas. Una tupla combina un número fijo de elementos para que puedan pasarse como un todo. A diferencia de una matriz o lista, una tupla puede contener objetos con diferentes tipos, pero también son inmutables.  
Las tuplas son del tipo Tuple1, Tuple2, Tuple3, etc. Actualmente hay un límite superior de 8 elementos. Para acceder a los elementos de una tupla `t`, puede usar el método `t._1`para acceder al primer elemento, `t._2`al segundo, y así sucesivamente.

#### 3.1.1. Crear una tupla

Aquí hay un ejemplo de cómo crear una tupla que contenga una Cadena y un Entero:

```
// (Java, 8)
Tuple2<String, Integer> java8 = Tuple.of("Java", 8); 

// "Java"
String s = java8._1; 

// 8
Integer i = java8._2; 
```

|     | Se crea una tupla a través del método de fábrica estático`Tuple.of()` |
| --- | --------------------------------------------------------------------- |
|     | Obtenga el primer elemento de esta tupla.                             |
|     | Obtenga el segundo elemento de esta tupla.                            |

#### 3.1.2. Asignar una tupla por componentes

El mapa de componentes evalúa una función por elemento en la tupla, devolviendo otra tupla.

```
// (vavr, 1)
Tuple2<String, Integer> that = java8.map(
        s -> s.substring(2) + "vr",
        i -> i / 8
);
```

#### 3.1.3. Mapear una tupla usando un mapeador

También es posible mapear una tupla usando una función de mapeo.

```
// (vavr, 1)
Tuple2<String, Integer> that = java8.map(
        (s, i) -> Tuple.of(s.substring(2) + "vr", i / 8)
);
```

#### 3.1.4. Transformar una tupla

Transform crea un nuevo tipo basado en el contenido de la tupla.

```
// "vavr 1"
String that = java8.apply(
        (s, i) -> s.substring(2) + "vr " + i / 8
);
```

### 3.2. Funciones

La programación funcional tiene que ver con los valores y la transformación de valores usando funciones. Java 8 solo proporciona un `Function`que acepta un parámetro y otro `BiFunction`que acepta dos parámetros. Vavr proporciona funciones hasta un límite de 8 parámetros. Las interfaces funcionales son de llamadas `Function0, Function1, Function2, Function3`y así sucesivamente. Si necesita una función que arroje una excepción verificada, puede usar `CheckedFunction1, CheckedFunction2`y así sucesivamente.  
La siguiente expresión lambda crea una función para sumar dos enteros:

```
// sum.apply(1, 2) = 3
Function2<Integer, Integer, Integer> sum = (a, b) -> a + b;
```

Esta es una abreviatura de la siguiente definición de clase anónima:

```
Function2<Integer, Integer, Integer> sum = new Function2<Integer, Integer, Integer>() {
    @Override
    public Integer apply(Integer a, Integer b) {
        return a + b;
    }
};
```

También puede usar el método de fábrica estático `Function3.of(…​)`para crear una función a partir de cualquier referencia de método.

```
Function3<String, String, String, String> function3 =
        Function3.of(this::methodWhichAccepts3Parameters);
```

De hecho, las interfaces funcionales de Vavr son interfaces funcionales de Java 8 con esteroides. También proporcionan características como:

- Composición

- Levantamiento

- Zurra

- Memoización

#### 3.2.1. Composición

Puede componer funciones. En matemáticas, la composición de funciones es la aplicación de una función al resultado de otra para producir una tercera función. Por ejemplo, las funciones f : X → Y y g : Y → Z se pueden componer para producir una función `h: g(f(x))`que mapea X → Z.  
Puede usar `andThen`:

```
Function1<Integer, Integer> plusOne = a -> a + 1;
Function1<Integer, Integer> multiplyByTwo = a -> a * 2;

Function1<Integer, Integer> add1AndMultiplyBy2 = plusOne.andThen(multiplyByTwo);

then(add1AndMultiplyBy2.apply(2)).isEqualTo(6);
```

o `compose`:

```
Function1<Integer, Integer> add1AndMultiplyBy2 = multiplyByTwo.compose(plusOne);

then(add1AndMultiplyBy2.apply(2)).isEqualTo(6);
```

#### 3.2.2. Levantamiento

Puede elevar una función parcial a una función total que devuelve un `Option`resultado. El término *función parcial* proviene de las matemáticas. Una función parcial de X a Y es una función f: X′ → Y, para algún subconjunto X′ de X. Generaliza el concepto de una función f: X → Y al no forzar a f a asignar cada elemento de X a un elemento de Y. Eso significa que una función parcial funciona correctamente solo para algunos valores de entrada. Si se llama a la función con un valor de entrada no permitido, normalmente generará una excepción.

El siguiente método `divide`es una función parcial que solo acepta divisores distintos de cero.

```
Function2<Integer, Integer, Integer> divide = (a, b) -> a / b;
```

Usamos `lift`para convertir `divide`en una función total que se define para todas las entradas.

```
Function2<Integer, Integer, Option<Integer>> safeDivide = Function2.lift(divide);

// = None
Option<Integer> i1 = safeDivide.apply(1, 0); 

// = Some(2)
Option<Integer> i2 = safeDivide.apply(4, 2); 
```

|     | Una función elevada regresa `None`en lugar de lanzar una excepción, si la función se invoca con valores de entrada no permitidos. |
| --- | --------------------------------------------------------------------------------------------------------------------------------- |
|     | Una función elevada devuelve `Some`, si la función se invoca con valores de entrada permitidos.                                   |

El siguiente método `sum`es una función parcial que solo acepta valores de entrada positivos.

```
int sum(int first, int second) {
    if (first < 0 || second < 0) {
        throw new IllegalArgumentException("Only positive integers are allowed"); 
    }
    return first + second;
}
```

|     | La función `sum`lanza un `IllegalArgumentException`para valores de entrada negativos. |
| --- | ------------------------------------------------------------------------------------- |

Podemos levantar el `sum`método proporcionando la referencia de métodos.

```
Function2<Integer, Integer, Option<Integer>> sum = Function2.lift(this::sum);

// = None
Option<Integer> optionalResult = sum.apply(-1, 2); 
```

|     | La función elevada captura el `IllegalArgumentException`y lo asigna a `None`. |
| --- | ----------------------------------------------------------------------------- |

#### 3.2.3. Aplicación parcial

La aplicación parcial le permite derivar una nueva función de una existente fijando algunos valores. Puede corregir uno o más parámetros, y el número de parámetros fijos define la aridad de la nueva función de modo que `new arity = (original arity - fixed parameters)`. Los parámetros están enlazados de izquierda a derecha.

```
Function2<Integer, Integer, Integer> sum = (a, b) -> a + b;
Function1<Integer, Integer> add2 = sum.apply(2); 

then(add2.apply(4)).isEqualTo(6);
```

|     | El primer parámetro `a`se fija en el valor 2. |
| --- | --------------------------------------------- |

Esto se puede demostrar fijando los tres primeros parámetros de a `Function5`, lo que da como resultado a `Function2`.

```
Function5<Integer, Integer, Integer, Integer, Integer, Integer> sum = (a, b, c, d, e) -> a + b + c + d + e;
Function2<Integer, Integer, Integer> add6 = sum.apply(2, 3, 1); 

then(add6.apply(4, 3)).isEqualTo(13);
```

|     | Los parámetros `a`, `b`y `c`se fijan en los valores 2, 3 y 1 respectivamente. |
| --- | ----------------------------------------------------------------------------- |

La aplicación parcial difiere de [Currying](https://docs.vavr.io/#_currying) , como se explorará en la sección correspondiente.

#### 3.2.4. Zurra

Currying es una técnica para aplicar parcialmente una función fijando un valor para uno de los parámetros, lo que da como resultado una `Function1`función que devuelve un `Function1`.

Cuando `Function2`se *curry* a , el resultado es indistinguible de la *aplicación parcial* de a `Function2`porque ambos dan como resultado una función de 1-aridad.

```
Function2<Integer, Integer, Integer> sum = (a, b) -> a + b;
Function1<Integer, Integer> add2 = sum.curried().apply(2); 

then(add2.apply(4)).isEqualTo(6);
```

|     | El primer parámetro `a`se fija en el valor 2. |
| --- | --------------------------------------------- |

Es posible que observe que, aparte del uso de `.curried()`, este código es idéntico al ejemplo dado de 2 aridades en [Aplicación parcial](https://docs.vavr.io/#_partial_application) . Con funciones de mayor aridad, la diferencia se vuelve clara.

```
Function3<Integer, Integer, Integer, Integer> sum = (a, b, c) -> a + b + c;
final Function1<Integer, Function1<Integer, Integer>> add2 = sum.curried().apply(2);

then(add2.apply(4).apply(3)).isEqualTo(9); 
```

|     | Tenga en cuenta la presencia de funciones adicionales en los parámetros.       |
| --- | ------------------------------------------------------------------------------ |
|     | Otras llamadas a `apply`devuelve otro `Function1`, además de la llamada final. |

#### 3.2.5. Memoización

La memorización es una forma de almacenamiento en caché. Una función memorizada se ejecuta solo una vez y luego devuelve el resultado de un caché.  
El siguiente ejemplo calcula un número aleatorio en la primera invocación y devuelve el número en caché en la segunda invocación.

```
Function0<Double> hashCache =
        Function0.of(Math::random).memoized();

double randomValue1 = hashCache.apply();
double randomValue2 = hashCache.apply();

then(randomValue1).isEqualTo(randomValue2);
```

### 3.3. Valores

En un entorno funcional, vemos un valor como una especie de [forma normal](https://en.wikipedia.org/wiki/Normal_form_(abstract_rewriting)) , una expresión que no puede evaluarse más. En Java expresamos esto haciendo que el estado de un objeto sea final y lo llamemos [inmutable](https://en.wikipedia.org/wiki/Immutable_object) .  
El valor funcional de Vavr se abstrae de los objetos inmutables. Se agregan operaciones de escritura eficientes al compartir memoria inmutable entre instancias. ¡Lo que obtenemos es seguridad de subprocesos gratis!

#### 3.3.1. Opción

Option es un tipo de contenedor monádico que representa un valor opcional. Las instancias de Option son una instancia de `Some`o el `None`.

```
// optional *value*, no more nulls
Option<T> option = Option.of(...);
```

Si viene a Vavr después de usar la `Optional`clase de Java, hay una diferencia crucial. En `Optional`, una llamada a `.map`que da como resultado un valor nulo dará como resultado un valor vacío `Optional`. En Vavr, daría como resultado un `Some(null)`que luego puede conducir a un `NullPointerException`.

Usando `Optional`, este escenario es válido.

```
Optional<String> maybeFoo = Optional.of("foo"); 
then(maybeFoo.get()).isEqualTo("foo");
Optional<String> maybeFooBar = maybeFoo.map(s -> (String)null)  
                                       .map(s -> s.toUpperCase() + "bar");
then(maybeFooBar.isPresent()).isFalse();
```

|     | la opción es`Some("foo")`              |
| --- | -------------------------------------- |
|     | La opción resultante queda vacía aquí. |

Usando Vavr `Option`, el mismo escenario dará como resultado un `NullPointerException`.

```
Option<String> maybeFoo = Option.of("foo"); 
then(maybeFoo.get()).isEqualTo("foo");
try {
    maybeFoo.map(s -> (String)null) 
            .map(s -> s.toUpperCase() + "bar"); 
    Assert.fail();
} catch (NullPointerException e) {
    // this is clearly not the correct approach
}
```

|     | la opción es`Some("foo")`                           |
| --- | --------------------------------------------------- |
|     | La opción resultante es`Some(null)`                 |
|     | La llamada a `s.toUpperCase()`se invoca en un`null` |

Parece que la implementación de Vavr está rota, pero de hecho no lo está; más bien, se adhiere al requisito de una mónada para mantener el contexto computacional al llamar a `.map`. En términos de an `Option`, esto significa que invocar `.map`a `Some`dará como resultado a `Some`, e invocar `.map`a `None`dará como resultado a `None`. En el ejemplo de Java `Optional`anterior, ese contexto cambió de `Some`a a `None`.

Esto puede parecer `Option`inútil, pero en realidad lo obliga a prestar atención a posibles ocurrencias `null`y a tratarlas en consecuencia en lugar de aceptarlas sin saberlo. La forma correcta de lidiar con las ocurrencias de `null`es usar `flatMap`.

```
Option<String> maybeFoo = Option.of("foo"); 
then(maybeFoo.get()).isEqualTo("foo");
Option<String> maybeFooBar = maybeFoo.map(s -> (String)null) 
                                     .flatMap(s -> Option.of(s) 
                                                         .map(t -> t.toUpperCase() + "bar"));
then(maybeFooBar.isEmpty()).isTrue();
```

|     | la opción es`Some("foo")`              |
| --- | -------------------------------------- |
|     | La opción resultante es`Some(null)`    |
|     | `s`, que es `null`, se convierte`None` |

Como alternativa, mueva el `.flatMap`para que se coloque junto con el posible `null`valor.

```
Option<String> maybeFoo = Option.of("foo"); 
then(maybeFoo.get()).isEqualTo("foo");
Option<String> maybeFooBar = maybeFoo.flatMap(s -> Option.of((String)null)) 
                                     .map(s -> s.toUpperCase() + "bar");
then(maybeFooBar.isEmpty()).isTrue();
```

|     | la opción es`Some("foo")`     |
| --- | ----------------------------- |
|     | La opción resultante es`None` |

Esto se explora con más detalle en el [blog de Vavr](http://blog.vavr.io/the-agonizing-death-of-an-astronaut/) .

#### 3.3.2. Tratar

Try es un tipo de contenedor monádico que representa un cálculo que puede resultar en una excepción o devolver un valor calculado con éxito. Es similar a, pero semánticamente diferente de `Either`. Las instancias de Try son una instancia de `Success`o `Failure`.

```
// no need to handle exceptions
Try.of(() -> bunchOfWork()).getOrElse(other);
```

```
import static io.vavr.API.*;        // $, Case, Match
import static io.vavr.Predicates.*; // instanceOf

A result = Try.of(this::bunchOfWork)
    .recover(x -> Match(x).of(
        Case($(instanceOf(Exception_1.class)), t -> somethingWithException(t)),
        Case($(instanceOf(Exception_2.class)), t -> somethingWithException(t)),
        Case($(instanceOf(Exception_n.class)), t -> somethingWithException(t))
    ))
    .getOrElse(other);
```

#### 3.3.3. Vago

Lazy es un tipo de contenedor monádico que representa un valor evaluado perezoso. Comparado con un Proveedor, Lazy está memorizando, es decir, evalúa una sola vez y por lo tanto es referencialmente transparente.

```
Lazy<Double> lazy = Lazy.of(Math::random);
lazy.isEvaluated(); // = false
lazy.get();         // = 0.123 (random generated)
lazy.isEvaluated(); // = true
lazy.get();         // = 0.123 (memoized)
```

También puede crear un valor perezoso real (funciona solo con interfaces):

```
CharSequence chars = Lazy.val(() -> "Yay!", CharSequence.class);
```

#### 3.3.4. O

Cualquiera representa un valor de dos tipos posibles. Un Cualquiera es un Izquierdo o un Derecho. Si el O bien dado es Derecha y se proyecta a Izquierda, las operaciones de Izquierda no tienen efecto en el valor de Derecha. Si el O bien dado es un Izquierdo y se proyecta a un Derecho, las operaciones Derecha no tienen efecto en el valor Izquierdo. Si se proyecta un Left a un Left o un Right a un Right, las operaciones surten efecto.

Ejemplo: una función de cálculo (), que da como resultado un valor entero (en caso de éxito) o un mensaje de error de tipo cadena (en caso de falla). Por convención, el caso de éxito es Derecha y el de falla es Izquierda.

```
Either<String,Integer> value = compute().right().map(i -> i * 2).toEither();
```

Si el resultado de compute() es Right(1), el valor es Right(2).  
Si el resultado de compute() es Left("error"), el valor es Left("error").

#### 3.3.5. Futuro

Un futuro es un resultado de cálculo que está disponible en algún momento. Todas las operaciones proporcionadas son sin bloqueo. El ExecutorService subyacente se utiliza para ejecutar controladores asíncronos, por ejemplo, a través de onComplete(...).

Un futuro tiene dos estados: pendiente y completado.

**Pendiente:** El cálculo está en curso. Solo se puede completar o cancelar un futuro pendiente.

**Completado:** el cálculo finalizó correctamente con un resultado, falló con una excepción o se canceló.

Las devoluciones de llamada se pueden registrar en un Futuro en cada momento. Estas acciones se realizan tan pronto como se completa el futuro. Una acción que se registra en un Futuro completado se realiza inmediatamente. La acción puede ejecutarse en un subproceso independiente, según el ExecutorService subyacente. Las acciones que se registran sobre un Futuro cancelado se realizan con resultado fallido.

```
// future *value*, result of an async calculation
Future<T> future = Future.of(...);
```

#### 3.3.6. Validación

El control de Validación es un *funtor aplicativo* y facilita la acumulación de errores. Al intentar componer Monads, el proceso de combinación se cortocircuitará en el primer error encontrado. Pero 'Validación' continuará procesando las funciones de combinación, acumulando todos los errores. Esto es especialmente útil cuando se realiza la validación de varios campos, por ejemplo, un formulario web, y desea conocer todos los errores encontrados, en lugar de uno a la vez.

Ejemplo: Obtenemos los campos 'nombre' y 'edad' de un formulario web y queremos crear una instancia de Persona válida o devolver la lista de errores de validación.

```
PersonValidator personValidator = new PersonValidator();

// Valid(Person(John Doe, 30))
Validation<Seq<String>, Person> valid = personValidator.validatePerson("John Doe", 30);

// Invalid(List(Name contains invalid characters: '!4?', Age must be greater than 0))
Validation<Seq<String>, Person> invalid = personValidator.validatePerson("John? Doe!4", -1);
```

Un valor válido está contenido en una `Validation.Valid`instancia, una lista de errores de validación está contenida en una `Validation.Invalid`instancia.

El siguiente validador se utiliza para combinar diferentes resultados de validación en una sola `Validation`instancia.

clase PersonValidator {
    Cadena final estática privada VALID_NAME_CHARS = "[a-zA-Z]";
    int final estático privado MIN_AGE = 0;
    public Validation<Seq<String>, Person> validatePerson(String name, int age) {
        return Validation.combine(validateName(nombre), validarEdad(edad)).ap(Persona::nueva);
    }
    Validación privada <Cadena, Cadena> validar Nombre (Cadena nombre) {
        return CharSeq.of(nombre).replaceAll(VALID_NAME_CHARS, "").transform(seq -> seq.isEmpty()
                ? Validación.válido(nombre)
                : Validation.invalid("El nombre contiene caracteres no válidos: '"
                + seq.distinct().sorted() + "'"));
    }
    Validación privada<Cadena, Entero> validarEdad(edad int) {
        edad de devolución < MIN_AGE
                ? Validation.invalid("La edad debe ser al menos " + MIN_AGE)
                : Validación.valid(edad);
    }
}

Si la validación tiene éxito, es decir, los datos de entrada son válidos, entonces se crea una instancia de `Person`los campos dados `name`y `age`.

Persona de clase {
    Nombre de cadena final público;
    edad pública final int;
    public Person(String nombre, int edad) {
        este.nombre = nombre;
        esta.edad = edad;
    }
    @Anular
    Cadena pública a Cadena () {
        return "Persona(" + nombre + ", " + edad + ")";
    }
}

### 3.4. Colecciones

Se ha puesto mucho esfuerzo en diseñar una biblioteca de colección completamente nueva para Java que cumpla con los requisitos de la programación funcional, es decir, la inmutabilidad.

Stream de Java eleva un cálculo a una capa diferente y se vincula a una colección específica en otro paso explícito. Con Vavr no necesitamos todo este repetitivo adicional.

Las nuevas colecciones se basan en [java.lang.Iterable](http://docs.oracle.com/javase/8/docs/api/java/lang/Iterable.html) , por lo que aprovechan el estilo de iteración azucarado.

```
// 1000 random numbers
for (double random : Stream.continually(Math::random).take(1000)) {
    ...
}
```

`TraversableOnce`tiene una gran cantidad de funciones útiles para operar en la colección. Su API es similar a [java.util.stream.Stream](http://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html) pero más madura.

#### 3.4.1. Lista

Vavr's `List`es una lista enlazada inmutable. Las mutaciones crean nuevas instancias. La mayoría de las operaciones se realizan en tiempo lineal. Las operaciones consiguientes se ejecutan una por una.

##### Java 8

```
Arrays.asList(1, 2, 3).stream().reduce((i, j) -> i + j);
```

```
IntStream.of(1, 2, 3).sum();
```

##### Vavr

```
// io.vavr.collection.List
List.of(1, 2, 3).sum();
```

#### 3.4.2. Arroyo

La `io.vavr.collection.Stream`implementación es una lista enlazada perezosa. Los valores se calculan solo cuando es necesario. Debido a su pereza, la mayoría de las operaciones se realizan en tiempo constante. Las operaciones son intermedias en general y se ejecutan en un solo paso.

Lo asombroso de los streams es que podemos usarlos para representar secuencias que son (teóricamente) infinitamente largas.

```
// 2, 4, 6, ...
Stream.from(1).filter(i -> i % 2 == 0);
```

#### 3.4.3. Características de presentación

*Tabla 1. Complejidad temporal de las operaciones secuenciales*

|               | head()     | tail()     | get(int)   | update(int,T) | prepend(T)  | append(T)   |
| ------------- | ---------- | ---------- | ---------- | ------------- | ----------- | ----------- |
| Array         | const      | linear     | const      | const         | linear      | linear      |
| CharSeq       | const      | linear     | const      | linear        | linear      | linear      |
| Iterator      | const      | const      | ______     | ______        | ______      | ______      |
| List          | const      | const      | linear     | linear        |             | linear      |
| Queue         | const      | const`a`   | linear     | linear        | const       | const       |
| PriorityQueue | log        | log        | ______     | ______        | log         | log         |
| Stream        | const      | const      | linear     | linear        | const*lazy* | const*lazy* |
| Vector        | const*eff* | const*eff* | const*eff* | const*eff*    | const*eff*  | const*eff*  |

*Table 2. Time Complexity of Map/Set Operations*

|               | contains/Key | add/put    | remove     | min    |
| ------------- |:------------:| ---------- | ---------- | ------ |
| HashMap       | const*eff*   | const*eff* | const*eff* | linear |
| HashSet       | const*eff*   | const*eff* | const*eff* | linear |
| LinkedHashMap | const*eff*   | linear     | linear     | linear |
| LinkedHashSet | const*eff*   | linear     | linear     | linear |
| Tree          | log          | log        | log        | log    |
| TreeMap       | log          | log        | log        | log    |
| TreeSet       | log          | log        | log        | log    |

Leyenda:

- const — tiempo constante

- const a — tiempo constante amortizado, pocas operaciones pueden tomar más tiempo

- const eff — tiempo efectivamente constante, dependiendo de supuestos como la distribución de claves hash

- const lazy — tiempo constante perezoso, la operación se aplaza

- log — tiempo logarítmico

- lineal — tiempo lineal

### 3.5. Comprobación de propiedades

La verificación de propiedades (también conocida como [prueba de propiedades](http://en.wikipedia.org/wiki/Property_testing) ) es una forma verdaderamente poderosa de probar las propiedades de nuestro código de una manera funcional. Se basa en datos aleatorios generados, que se pasan a una función de verificación definida por el usuario.

Vavr tiene soporte para pruebas de propiedades en su `io.vavr:vavr-test`módulo, así que asegúrese de incluirlo para usarlo en sus pruebas.

```java
Arbitrary<Integer> ints = Arbitrary.integer();

// square(int) >= 0: OK, passed 1000 tests.
Property.def("square(int) >= 0")
        .forAll(ints)
        .suchThat(i -> i * i >= 0)
        .check()
        .assertIsSatisfied();
```

Los generadores de estructuras de datos complejas se componen de generadores simples.

### 3.6. La coincidencia de patrones

Scala tiene coincidencia de patrones nativos, una de las ventajas sobre Java *simple .* La sintaxis básica está cerca del interruptor de Java:

```java
val s = i match {
  case 1 => "one"
  case 2 => "two"
  case _ => "?"
}
```

En particular *, el partido* es una expresión, produce un resultado. Además ofrece

- parámetros con nombre`case i: Int ⇒ "Int " + i`

- deconstrucción de objetos`case Some(i) ⇒ i`

- guardias`case Some(i) if i > 0 ⇒ "positive " + i`

- múltiples condiciones`case "-h" | "--help" ⇒ displayHelp`

- comprobaciones en tiempo de compilación para la exhaustividad

La coincidencia de patrones es una gran característica que nos evita escribir montones de ramas if-then-else. Reduce la cantidad de código mientras se enfoca en las partes relevantes.

#### 3.6.1. Los fundamentos de Match para Java

Vavr proporciona una API de coincidencia que está cerca de la coincidencia de Scala. Se habilita agregando la siguiente importación a nuestra aplicación:

```java
import static io.vavr.API.*;
```

Tener los métodos estáticos *Match* , *Case* y los *patrones atómicos*

- `$()`- patrón comodín

- `$(value)`- patrón igual

- `$(predicate)`- patrón condicional

en alcance, el ejemplo inicial de Scala se puede expresar así:

```java
String s = Match(i).of(
    Case($(1), "one"),
    Case($(2), "two"),
    Case($(), "?")
);
```

⚡ Usamos nombres de métodos uniformes en mayúsculas porque 'case' es una palabra clave en Java. Esto hace que la API sea especial.

##### Agotamiento

El último patrón de comodines `$()`nos salva de un MatchError que se lanza si ningún caso coincide.

Debido a que no podemos realizar comprobaciones exhaustivas como el compilador de Scala, brindamos la posibilidad de devolver un resultado opcional:

```java
Option<String> s = Match(i).option(
    Case($(0), "zero")
);
```

##### Azúcar sintáctica

Como ya se mostró, `Case`permite hacer coincidir patrones condicionales.

```java
Case($(predicate), ...)
```

Vavr ofrece un conjunto de predicados predeterminados.

```java
import static io.vavr.Predicates.*;
```

Estos se pueden usar para expresar el ejemplo inicial de Scala de la siguiente manera:

```java
String s = Match(i).of(
    Case($(is(1)), "one"),
    Case($(is(2)), "two"),
    Case($(), "?")
);
```

**Múltiples Condiciones**

Usamos el `isIn`predicado para verificar múltiples condiciones:

```java
Case($(isIn("-h", "--help")), ...)
```

**Realización de efectos secundarios**

Match actúa como una expresión, da como resultado un valor. Para realizar efectos secundarios, necesitamos usar la función auxiliar `run`que devuelve `Void`:

```java
Match(arg).of(
    Case($(isIn("-h", "--help")), o -> run(this::displayHelp)),
    Case($(isIn("-v", "--version")), o -> run(this::displayVersion)),
    Case($(), o -> run(() -> {
        throw new IllegalArgumentException(arg);
    }))
);
```

⚡ `run`se usa para evitar ambigüedades y porque `void`no es un valor de retorno válido en Java.

**Precaución:** `run` no debe usarse como valor de retorno directo, es decir, fuera de un cuerpo lambda:

```java
// Wrong!
Case($(isIn("-h", "--help")), run(this::displayHelp))
```

De lo contrario, los casos se evaluarán ansiosamente *antes* de que coincidan los patrones, lo que rompe toda la expresión Match. En su lugar, lo usamos dentro de un cuerpo lambda:

```java
// Ok
Case($(isIn("-h", "--help")), o -> run(this::displayHelp))
```

Como podemos ver, `run`es propenso a errores si no se usa correctamente. Ten cuidado. Consideramos desaprobarlo en una versión futura y tal vez también proporcionemos una mejor API para realizar efectos secundarios.

##### Parámetros con nombre

Vavr aprovecha lambdas para proporcionar parámetros con nombre para valores coincidentes.

```java
Number plusOne = Match(obj).of(
    Case($(instanceOf(Integer.class)), i -> i + 1),
    Case($(instanceOf(Double.class)), d -> d + 1),
    Case($(), o -> { throw new NumberFormatException(); })
);
```

Hasta ahora hemos emparejado valores directamente usando patrones atómicos. Si un patrón atómico coincide, el tipo correcto del objeto coincidente se deduce del contexto del patrón.

A continuación, echaremos un vistazo a los patrones recursivos que pueden hacer coincidir gráficos de objetos de (teóricamente) profundidad arbitraria.

##### Descomposición de objetos

En Java usamos constructores para crear instancias de clases. Entendemos la *descomposición* de objetos como la destrucción de objetos en sus partes.

Mientras que un constructor es una *función* que se *aplica* a los argumentos y devuelve una nueva instancia, un deconstructor es una función que toma una instancia y devuelve las partes. Decimos que un objeto no se *aplica* .

La destrucción de objetos no es necesariamente una operación única. Por ejemplo, una LocalDate se puede descomponer en

- los componentes de año, mes y día

- el valor largo que representa la época en milisegundos del Instante correspondiente

- etc.

#### 3.6.2. Patrones

En Vavr usamos patrones para definir cómo se deconstruye una instancia de un tipo específico. Estos patrones se pueden usar junto con Match API.

##### Patrones predefinidos

Para muchos tipos de Vavr ya existen patrones de coincidencia. Se importan a través de

```java
import static io.vavr.Patterns.*;
```

Por ejemplo, ahora podemos hacer coincidir el resultado de un intento:

```java
Match(_try).of(
    Case($Success($()), value -> ...),
    Case($Failure($()), x -> ...)
);
```

⚡ Un primer prototipo de Match API de Vavr permitió extraer una selección de objetos definida por el usuario de un patrón de coincidencia. Sin el soporte adecuado del compilador, esto no es factible porque la cantidad de métodos generados se disparó exponencialmente. La API actual se compromete a que todos los patrones coincidan, pero solo se *descomponen* los patrones raíz .

```java
Match(_try).of(
    Case($Success($Tuple2($("a"), $())), tuple2 -> ...),
    Case($Failure($(instanceOf(Error.class))), error -> ...)
);
```

Aquí los patrones fundamentales son Éxito y Fracaso. Se descomponen en Tuple2 y Error, teniendo los tipos genéricos correctos.

⚡ Los tipos profundamente anidados se infieren según el argumento Match y *no* según los patrones coincidentes.

##### Patrones definidos por el usuario

Es esencial poder dejar de aplicar objetos arbitrarios, incluidas instancias de clases finales. Vavr hace esto en un estilo declarativo proporcionando las anotaciones de tiempo de compilación `@Patterns`y `@Unapply`.

Para habilitar el procesador de anotaciones, se debe agregar el artefacto [vavr-match](http://search.maven.org/#search%7Cga%7C1%7Cvavr-match) como dependencia del proyecto.

⚡ Nota: Por supuesto, los patrones se pueden implementar directamente sin usar el generador de código. Para obtener más información, eche un vistazo a la fuente generada.

```java
import io.vavr.match.annotation.*;

@Patterns
class My {

    @Unapply
    static <T> Tuple1<T> Optional(java.util.Optional<T> optional) {
        return Tuple.of(optional.orElse(null));
    }
}
```

El procesador de anotaciones coloca un archivo MyPatterns en el mismo paquete (por defecto en target/generated-sources). Las clases internas también son compatibles. Caso especial: si el nombre de la clase es $, el nombre de la clase generada es solo Patrones, sin prefijo.

##### guardias

Ahora podemos emparejar opcionales usando *guardias* .

```java
Match(optional).of(
    Case($Optional($(v -> v != null)), "defined"),
    Case($Optional($(v -> v == null)), "empty")
);
```

Los predicados podrían simplificarse implementando `isNull`y `isNotNull`.

⚡ Y sí, extraer nulo es extraño. ¡En lugar de usar Opcional de Java, pruebe la Opción de Vavr!

```java
Match(option).of(
    Case($Some($()), "defined"),
    Case($None(), "empty")
);
```












