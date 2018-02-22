## `Box<T>` apunta a los datos en la zona libre y tiene un tamaño conocido

El puntero inteligente más sencillo es un *recuadro*, cuyo tipo está escrito
`Box<T>`. Los recuadros te permiten almacenar datos en la zona libre en vez de en la
pila. Lo que queda en la pila es el puntero en la zona libre de datos. Consulta el
capítulo 4 para ver la diferencia entre la pila y la zona libre.

Los cuadros no tienen sobrecarga, además de almacenar sus datos en la zona libre
en vez de en la pila. Pero no tienen muchas capacidades extras tampoco.
Los usarás más a menudo en estas situaciones:

* Cuando tienes un tipo cuyo tamaño no se puede saber en tiempo de compilación, y quieres
  usar un valor de ese tipo en un contexto que necesita saber un tamaño exacto
* Cuando tienes una gran cantidad de datos y quieres transferir la propiedad pero
  asegúrate que los datos no se copiarán cuando lo hagas
* Cuando quieres tener un valor y solo importa que sea un tipo que implemente una
  característica partícular en vez de conocer el tipo concreto

Vamos a demostrar la primera situación en esta sección. Pero antes de hacer eso,
elaboraremos un poco más las otras dos situaciones: en el segundo caso,
transferir la propiedad de una gran cantidad de datos puede tomar mucho tiempo porque
los datos se copian cerca de la pila. Para mejorar el rendimiento en esta 
situación, podemos almacenar la gran cantidad de datos en la zona libre en un recuadro. Entonces,
solo la cantidad pequeña de datos del puntero son copiadas en la pila, y los 
datos permanecen en un lugar en la zona libre. El tercer caso es conocido como un *trait
object*, y el capítulo 17 le dedica una sección completa a ese tema. Asique, lo que
aprendas aquí lo aplicarás de nuevo en el capítulo 17!


### Uso de `Box<T>` para almacenar datos en la zona libre

Antes de discutir este caso de uso para `Box<T>`, cubriremos la sintaxis y como
interactuar con valores almacenados dentro de un `Box<T>`.

El listado 15-1 muestra como usar un recuadro para almacenar un valor `i32` en la zona libre:

<span class="filename">Nombre del archivo: src/main.rs</span>

```rust
fn main() {
    let b = Box::new(5);
    println!("b = {}", b);
}
```
<span class="caption">Listado 15-1: Almacenando un valor `i32` en la zona libre usando
un recuadro</span>

Definimos la variable `b` para tener el valor de un `Box` que apunta al valor
 `5`, que es asignado en la zona libre. Este programa imprimirá `b = 5`; en
este caso, podemos ingresar los datos en un recuadro en una forma similar como
lo hariamos si los datos estuvieran en la pila. Al igual que cualquier valor propio,
cuando un recuadro sale del alcance como `b` se hace al final de `main`, será desasignado. La
designación ocurre para el recuadro (almacenada en la pila) y los datos que apuntan a
(almacenado en la zona libre).


Poner un solo valor en la zona libre no es muy útil, por lo que no usarás recuadros
por si mismos de esta manera muy seguido. Teniendo valores como un solo `i32` en
la pila, donde son almacenadas por defecto, es más apropiado en la mayoría de las
situaciones. Veamos un caso donde los recuadros nos permiten definir tipos que 
no estarían permitidos si no tuvieramos recuadros.

### Recuadros Habilitar Tipos Recursivos


A la hora de compilar, Rust necesita saber cuanto espacio ocupa un tipo. Un tipo
cuyo tamaño no se puede saber al momento d ela compilación es un *tipo recursivo*,
donde un valor puede tener como parte de si mismo otro valor del mismo tipo. Porque este
anidamiento de valores teóricamente podría continuar infinitamente, Rust no necesita saber
cuanto espacio necesita un valor de un tipo recursivo. Sin embargo, los recuadros tienen
un tamaño conocido, por lo que al insertar un recuadro en una definición de tipo recursivo,
podemos tener tipos recursivos.

Exploremos la *lista de constructores*, que es un tipo de dato común en lenguajes
de programación de tipo funcional, como un ejemplo de tipo recursivo. La lista de tipos
constantes la definiremos sencillamente excepto por la recursión; por lo tanto, el
concepto en el ejemplo con el que trabajaremos será util cada vez que situaciones más
complejas que involucran tipos recursivos.

#### Más información sobre la lista de constantes

Una *lista de constructores* es una estructura de datos que proviene del lenguaje de
programación Lisp y sus dialectos. En Lisp, la función `cons`(abreviatura de “función de
construcción”) construye un nuevo par desde sus dos argumentos, que generalmente son un
valor único y otro par. Estos pares que contienen pares forman una lista.

El concepto de función de construcción ha llegado más a la jerga de programación
funcional general: “para construir x en y” informalmente significa construir una nueva
instancia de contenedor poniendo el elemento x al comienzo de este nuevo contenedor,
seguido por el contenedor y.

Cada elemento en una lista de constructores contiene dos elementos: el valor del elemento
actual y del elemento siguiente. El último elementi en la lista contiene solo un valor
llamado `Nil` sin un elemento siguiente. Una lista de constructores es producida  recursivamente
llamando a la función `cons`. El nombre canónico para denotar el caso base de la recursión es `Nil`.
Tenga en cuenta que esto no es igual al concepto “null” o “nil” del capítulo Capítulo 6,
que es un valor inválido o ausente.

Aunque los lenguajes de programación funcionales usan lista de constructores frecuentemente,
no es una estructura de datos usada comunmente en Rust. La mayoría de las veces cuando tienes
una lista de elementos en Rist, `Vec<T>` es la mejor opción para usar. Otro, tipos de datos
recursivos más complejos *son* útiles en varias situaciones, pero empezando por la lista de
constructores, podemos explorar como los recuadros nos permiten definir un tipo de datos
recursivo sin mucha distracción.

El listado 15-2 contiene una definición de enumeraciones para una lista de constructores.
Tenga en cuenta que este código no será compilado todavía porque el tipo `List` no tiene un 
tamaño conocido, que demostraremos:


<span class="filename">Nombre del archivo: src/main.rs</span>

```rust,ignore
enum List {
    Cons(i32, List),
    Nil,
}
```

<span class="caption">Listado 15-2: El primer intento de definir una enumeración
para representar una lista de constructores de estructuras de datos de los valores `i32`</span>

> Nota: Estamos implementando una lista de constructores que solo contiene valores `i32`
> para los propósitos de este ejemplo. Podríamos haberlas implementado usando genéricos,
> como discutimos en el Capítulo 10, para definir un tipo de lista de constructores que
> podría almacenar valores de cualquier tipo.


Using the `List` type to store the list `1, 2, 3` would look like the code in
Listing 15-3:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
use List::{Cons, Nil};

fn main() {
    let list = Cons(1, Cons(2, Cons(3, Nil)));
}
```

<span class="caption">Listing 15-3: Using the `List` enum to store the list `1,
2, 3`</span>

The first `Cons` value holds `1` and another `List` value. This `List` value is
another `Cons` value that holds `2` and another `List` value. This `List` value
is one more `Cons` value that holds `3` and a `List` value, which is finally
`Nil`, the non-recursive variant that signals the end of the list.

If we try to compile the code in Listing 15-3, we get the error shown in
Listing 15-4:

```text
error[E0072]: recursive type `List` has infinite size
 --> src/main.rs:1:1
  |
1 | enum List {
  | ^^^^^^^^^ recursive type has infinite size
2 |     Cons(i32, List),
  |               ----- recursive without indirection
  |
  = help: insert indirection (e.g., a `Box`, `Rc`, or `&`) at some point to
  make `List` representable
```

<span class="caption">Listing 15-4: The error we get when attempting to define
a recursive enum</span>

The error shows this type “has infinite size.” The reason is that we’ve defined
`List` with a variant that is recursive: it holds another value of itself
directly. As a result, Rust can’t figure out how much space it needs to store a
`List` value. Let’s break down why we get this error a bit: first, let’s look
at how Rust decides how much space it needs to store a value of a non-recursive
type.

#### Computing the Size of a Non-Recursive Type

Recall the `Message` enum we defined in Listing 6-2 when we discussed enum
definitions in Chapter 6:

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
```

To determine how much space to allocate for a `Message` value, Rust goes
through each of the variants to see which variant needs the most space. Rust
sees that `Message::Quit` doesn’t need any space, `Message::Move` needs enough
space to store two `i32` values, and so forth. Because only one variant will be
used, the most space a `Message` value will need is the space it would take to
store the largest of its variants.

Contrast this to what happens when Rust tries to determine how much space a
recursive type like the `List` enum in Listing 15-2 needs. The compiler starts
by looking at the `Cons` variant, which holds a value of type `i32` and a value
of type `List`. Therefore, `Cons` needs an amount of space equal to the size of
an `i32` plus the size of a `List`. To figure out how much memory the `List`
type needs, the compiler looks at the variants, starting with the `Cons`
variant. The `Cons` variant holds a value of type `i32` and a value of type
`List`, and this process continues infinitely, as shown in Figure 15-1:

<img alt="An infinite Cons list" src="img/trpl15-01.svg" class="center" style="width: 50%;" />

<span class="caption">Figure 15-1: An infinite `List` consisting of infinite
`Cons` variants</span>

#### Using `Box<T>` to Get a Recursive Type with a Known Size

Rust can’t figure out how much space to allocate for recursively defined types,
so the compiler gives the error in Listing 15-4. But the error does include
this helpful suggestion:

```text
  = help: insert indirection (e.g., a `Box`, `Rc`, or `&`) at some point to
  make `List` representable
```

In this suggestion, “indirection” means that instead of storing a value
directly, we’ll change the data structure to store the value indirectly by
storing a pointer to the value instead.

Because a `Box<T>` is a pointer, Rust always knows how much space a `Box<T>`
needs: a pointer’s size doesn’t change based on the amount of data it’s
pointing to. This means we can put a `Box<T>` inside the `Cons` variant instead
of another `List` value directly. The `Box<T>` will point to the next `List`
value that will be on the heap rather than inside the `Cons` variant.
Conceptually, we still have a list, created with lists “holding” other lists,
but this implementation is now more like the items being next to one another
rather than inside one another.

We can change the definition of the `List` enum in Listing 15-2 and the usage
of the `List` in Listing 15-3 to the code in Listing 15-5, which will compile:

<span class="filename">Filename: src/main.rs</span>

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

<span class="caption">Listing 15-5: Definition of `List` that uses `Box<T>` in
order to have a known size</span>

The `Cons` variant will need the size of an `i32` plus the space to store the
box’s pointer data. The `Nil` variant stores no values, so it needs less space
than the `Cons` variant. We now know that any `List` value will take up the
size of an `i32` plus the size of a box’s pointer data. By using a box, we’ve
broken the infinite, recursive chain, so the compiler can figure out the size
it needs to store a `List` value. Figure 15-2 shows what the `Cons` variant
looks like now:

<img alt="A finite Cons list" src="img/trpl15-02.svg" class="center" />

<span class="caption">Figure 15-2: A `List` that is not infinitely sized
because `Cons` holds a `Box`</span>

Boxes only provide the indirection and heap allocation; they don’t have any
other special capabilities, like those we’ll see with the other smart pointer
types. They also don’t have any performance overhead that these special
capabilities incur, so they can be useful in cases like the cons list where the
indirection is the only feature we need. We’ll look at more use cases for boxes
in Chapter 17, too.

The `Box<T>` type is a smart pointer because it implements the `Deref` trait,
which allows `Box<T>` values to be treated like references. When a `Box<T>`
value goes out of scope, the heap data that the box is pointing to is cleaned
up as well because of the `Drop` trait implementation. Let’s explore these two
traits in more detail. These two traits will be even more important to the
functionality provided by the other smart pointer types we’ll discuss in the
rest of this chapter.
