## `Box<T>` Points to Data on the Heap and Has a Known Size

The most straightforward smart pointer is a *box*, whose type is written
`Box<T>`. Boxes allow you to store data on the heap rather than the stack. What
remains on the stack is the pointer to the heap data. Refer to Chapter 4 to
review the difference between the stack and the heap.

Boxes don’t have performance overhead, other than storing their data on the
heap instead of on the stack. But they don’t have many extra capabilities
either. You’ll use them most often in these situations:

* When you have a type whose size can’t be known at compile time, and you want
  to use a value of that type in a context that needs to know an exact size
* When you have a large amount of data and you want to transfer ownership but
  ensure the data won’t be copied when you do so
* When you want to own a value and only care that it’s a type that implements a
  particular trait rather than knowing the concrete type

We’ll demonstrate the first situation in this section. But before we do so,
we’ll elaborate on the other two situations a bit more: in the second case,
transferring ownership of a large amount of data can take a long time because
the data is copied around on the stack. To improve performance in this
situation, we can store the large amount of data on the heap in a box. Then,
only the small amount of pointer data is copied around on the stack, and the
data stays in one place on the heap. The third case is known as a *trait
object*, and Chapter 17 devotes an entire section just to that topic. So what
you learn here you’ll apply again in Chapter 17!

### Using a `Box<T>` to Store Data on the Heap

Before we discuss this use case for `Box<T>`, we’ll cover the syntax and how to
interact with values stored within a `Box<T>`.

Listing 15-1 shows how to use a box to store an `i32` value on the heap:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let b = Box::new(5);
    println!("b = {}", b);
}
```

<span class="caption">Listing 15-1: Storing an `i32` value on the heap using a
box</span>

We define the variable `b` to have the value of a `Box` that points to the
value `5`, which is allocated on the heap. This program will print `b = 5`; in
this case, we can access the data in the box in a similar way as we would if
this data was on the stack. Just like any owned value, when a box goes out of
scope like `b` does at the end of `main`, it will be deallocated. The
deallocation happens for the box (stored on the stack) and the data it points
to (stored on the heap).

Putting a single value on the heap isn’t very useful, so you won’t use boxes by
themselves in this way very often. Having values like a single `i32` on the
stack, where they’re stored by default, is more appropriate in the majority of
situations. Let’s look at a case where boxes allow us to define types that we
wouldn’t be allowed to if we didn’t have boxes.

### Boxes Enable Recursive Types

At compile time, Rust needs to know how much space a type takes up. One type
whose size can’t be known at compile time is a *recursive type*, where a value
can have as part of itself another value of the same type. Because this nesting
of values could theoretically continue infinitely, Rust doesn’t know how much
space a value of a recursive type needs. However, boxes have a known size, so
by inserting a box in a recursive type definition, we can have recursive types.

Let’s explore the *cons list*, which is a data type common in functional
programming languages, as an example of a recursive type. The cons list type
we’ll define is straightforward except for the recursion; therefore, the
concepts in the example we’ll work with will be useful any time you get into
more complex situations involving recursive types.

#### More Information About the Cons List

A *cons list* is a data structure that comes from the Lisp programming language
and its dialects. In Lisp, the `cons` function (short for “construct function”)
constructs a new pair from its two arguments, which usually are a single value
and another pair. These pairs containing pairs form a list.

The cons function concept has made its way into more general functional
programming jargon: “to cons x onto y” informally means to construct a new
container instance by putting the element x at the start of this new container,
followed by the container y.

Each item in a cons list contains two elements: the value of the current item
and the next item. The last item in the list contains only a value called `Nil`
without a next item. A cons list is produced by recursively calling the `cons`
function. The canonical name to denote the base case of the recursion is `Nil`.
Note that this is not the same as the “null” or “nil” concept in Chapter 6,
which is an invalid or absent value.

Although functional programming languages use cons lists frequently, it isn’t a
commonly used data structure in Rust. Most of the time when you have a list of
items in Rust, `Vec<T>` is a better choice to use. Other, more complex
recursive data types *are* useful in various situations, but by starting with
the cons list, we can explore how boxes let us define a recursive data type
without much distraction.

Listing 15-2 contains an enum definition for a cons list. Note that this code
won’t compile yet because the `List` type doesn’t have a known size, which
we’ll demonstrate:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
enum List {
    Cons(i32, List),
    Nil,
}
```

<span class="caption">Listing 15-2: The first attempt at defining an enum to
represent a cons list data structure of `i32` values</span>

> Note: We’re implementing a cons list that only holds `i32` values for the
> purposes of this example. We could have implemented it using generics, as we
> discussed in Chapter 10, to define a cons list type that could store values of
> any type.


Usar el tipo `List` para almacenar la lista `1, 2, 3` se vería como el código
en el Listado 15-3:

<span class="filename">Nombre del archivo: src/main.rs</span>

```rust,ignore
use List::{Cons, Nil};

fn main() {
    let list = Cons(1, Cons(2, Cons(3, Nil)));
}
```

<span class="caption">Listado 15-3: Usando la enumeración `List` para almacenar la
lista `1, 2, 3`</span>

El primer valor `Cons` tiene `1` y otro valor de `List`. Este valor `List` 
es otro valor `Cons` que tiene `2` y otro valor `List`. Este valor `List` 
es un valor más `Cons` que tiene `3` y un valor `List`, que es finalmente
`Nil`, la variante no recursiva que señala el final de la lista.

Si intentamos compilar el código en el Listado 15-3, obtenemos el error mostardo
en el Listado 15-4:

```texto
error[E0072]: el tipo recursivo `List` tiene tamaño infinito
 --> src/main.rs:1:1
  |
1 | enum List {
  | ^^^^^^^^^ el tipo recursivo tiene tamaño infinito
2 |     Cons(i32, List),
  |               ----- recursivo sin indirección
  |
  = ayuda: insertar indireccionamiento (e.g., a `Box`, `Rc`, or `&`) en algún punto
  para hacer `List` representable
```
<span class="caption">Listado 15-4: El error que obtenemos cuando intentamos definir
una enumeración recursiva</span>

El error que muestra este tipo “tiene tamaño infinito.” La razón es que hemos
definido `List` con una variante que es recursiva: tiene otro valor de sí mismo
directamente. Como resultado, Rust no puede calcular cuanto tamaño necesita para
almacenar un valor `List`. Analicemos un poco porque obtenemos este error: primero, 
veamos como Rust decide cuanto espacio necesita para almacenar un valor de un tipo
no recursivo.



#### Calcular el tamaño de un tipo No Recursivo

Recuerda la enumeración `Message` que definimos en el Listado 6-2 cuando discutimos
definiciones de enumeraciones en el Capítulo 6:

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
```

Para determinar cuanto espacio asignar para un valor `Message`, Rust va
a través de cada variante para ver cual necesita más espacio. Rust
ve que `Message::Quit` no necesita ningún espacio, `Message::Move` necesita suficiente
espacio para almacenar dos valores `i32`, etcétera. Porque solo una variante será usada,
el mayor espacio que un valor `Message` necesitará es el espacio que le tomaría almacenar
la más grande de sus variantes.

Contrasta esto con lo que pasa cuando Rust intenta determinar cuanto espacio
un tipo recursivo como la enumeración `List` en el Listado 15-2 necesita. El comilador comienza
mirando a al variante `Cons`, que mantiene un valor de tipo `i32` y un valor de
tipo `List`. Por lo tanto, `Cons` necesita una cantidad de espacio igual al tamaño de
un `i32` sumado al tamaño de una `List`. Para averiguar cuanta memoria el tipo `List`
necesita, el compilador mira las variantes, comenzando con la variante `Cons`.
La variante `Cons` tiene un valor de tipo `i32` y un valor de tipo
`List`, y este proceso continúa infinitamente, como es mostrado en la figura 15-1:


<img alt="An infinite Cons list" src="img/trpl15-01.svg" class="center" style="width: 50%;" />

<span class="caption">Figura 15-1: Una `Lista` infinita consiste en infinitas
variantes `Cons` </span>

#### Usando `Box<T>` para obtener un Tipo Recursivo con un Tamaño Conocido

Rust no puede averiguar cuanto espacio asignar para los tipos definidos
recursivamente, por lo que el compilador da el error en el Listado 15-4.
Pero el error incluye esta sugerencia de ayuda:

```texto
  = ayuda: inserte indirección (e.g., a `Box`, `Rc`, or `&`) en algún punto
  para hacer `List` representable
```

En esta sugerencia, “indirección” significa que en lugar de almacenar un valor
directamente, cambiaremos la estructura de datos para almacenar el valor indirectamente
almacenando un puntero al valor en su lugar.


Ya que `Box<T>` es un puntero, Rust cuanto espacio un `Box<T>` 
necesita: el tamaño de un puntero no cambia basado en la cantidad de datos
a los que se apuntan. Esto significa que podemos poner un `Box<T>` adentro de
la variante `Cons` en lugar de otro valor `List` directamente. The `Box<T>` 
apuntará al siguiente valor de `List` que estará en el montón en lugar de adentro
la variante `Cons`. 
Conceptualmente, aún tenemos una lista, creada con listas “manteniendo” otras listas,
pero esta implementacíón ahora es más como los elementos que estan uno al ladro del otro
en vez de uno adentro de otro.

Podemos cambiar la definición de la enumeración `List` en el Listado 15-2 y el uso
de la `List` en el Listado 15-3 al código en el Listado 15-5, que compilará:

<span class="filename">Nombre de archivo: src/main.rs</span>

```rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}

use List::{Cons, Nil};

fn main() {
    let list = Cons(1,
        Box::new(Cons(2,
            Box::new(Cons(3,
                Box::new(Nil))))));
}
```

<span class="caption">Listado 15-5: Definición de `List` que usa `Box<T>` para
tener un tamaño conocido</span>

La variante `Cons` necesitará el tamaño de un `i32` más el espacio para almacenar
los datos del puntero del recuadro. La variante `Nil` no almacena valores, por lo
que necesita menos espacio que la variante `Cons`. Ahora sabemos que ningún valor `List`
tomará el tamaño de un `i32` más el tamaño de datos de un puntero de recuadro.
Usando un recuadro, rompemos la cadena recursiva e infinita, por lo que el compilador
puede averiguar el tamaño que necesita para almacenar un valor `List`. La figura 15-2
muestra que la variante `Cons` ahora se ve como:

<img alt="A finite Cons list" src="img/trpl15-02.svg" class="center" />

<span class="caption">Figura 15-2: Una `List` que no tiene un tamaño infinito
porque `Cons` contiene un `Box`</span>

Los recuadros solo proveen la asignación indirecta y del montón; no tienen
otras capacidades especiales, como las que veremos con los otros tipos de punteros
inteligentes. Tampoco tienen ninguna sobrecarga de rendimiento que estas capacidades
especiales incurren, por lo que pueden ser útiles en casos como la lista de constructores
donde la indirección es la única característica que necesitamos. Miraremos más casos de uso
para recuadros en el Capítulo 17, también.

El tipo `Box<T>` es un puntero inteligente porque implementa la característica `Deref`,
que permite que los valores `Box<T>` sean tratados como referencias. Cuando un valor `Box<T>`
se va del alcance, los datos del montón a los que apunta el recuadro son limpiados
debido a la implementación de la característica `Drop`. Exploremos estas dos 
características en más detalles. Estas dos características serán inclusi más importantes
para la funcionalidad prevista por los otros tipos de punteros inteligentes que discutiremos
en el resto de este capítulo.
