## Tipos de datos genéricos

Usando los genéricos donde usualmente ponemos tipos, como en firmas de función
o estructuras, nos deja crear definiciones que podemos usar para tipos 
de datos concretos. Miremos cómo definir funciones, estructuras, enum y
métodos usando genéricos, y al final de esta sección discutiremos el
desempeño del código usando genéricos.

### Usando Tipos de Datos Genéricos en la Definición de Funciones

Podemos definir funciones que usan genéricos en la firma de la función
donde los tipos de datos de los parametros y los valores de respuesta van. De esta manera,
el código que escribimos puede ser más flexible y proveer más funcionalidad a los llamados de
nuestra función, aunque no introduzcamos una duplicación de código.

Continuando con nuestra función `largest`, el listado 10-4 muestra dos funciones
que proveen la misma funcionalidad para encontrar el valor más grande en un pedazo. La
primera función es la que extrajimos en el listado 10-3 que encuentra el más grande
`i32` en un pedazo. La segunda función encuentra el más grande `char` en un pedazo:

<span class="filename">Nombre del archivo: src/main.rs</span>

```rust
fn largest_i32(list: &[i32]) -> i32 {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn largest_char(list: &[char]) -> char {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest_i32(&number_list);
    println!("The largest number is {}", result);
#    assert_eq!(result, 100);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest_char(&char_list);
    println!("The largest char is {}", result);
#    assert_eq!(result, 'y');
}
```

<span class="caption">Listado 10-4: Dos funciones que difieren solo en sus
nombres y los tipos en sus firmas</span>

Aquí, las funciones `largest_i32` y `largest_char` tienen exactamente el mismo cuerpo,
así que sería bueno si pudieramos convertir estas dos funciones en una y eliminar
la duplicación. Afortunadamente, ¡Podemos hacer eso introduciendo un parámetro de un tipo
genérico!

Para parametrizar los tipos en las firmas de una función, vamos a
definir, necesitamos crear un nombre para el tipo de parámetro, justo como nosotros le damos
nombres para los valores del parametro de una función. Vamos a escoger el nombre
`T`. Cualquier identificador puede ser usado como el nombre para un tipo de parametro, pero escogeremos
`T` porque la convención de nombres de Rust es Camelcase. Los parámetros de nombre genéricos
también tienden a ser cortos por conveniencia, usualmente solo una letra. Diminutivo para
“type”, `T` es la opción por defecto para la mayoría de los programadores de Rust.

Cuando usamos un parametro en el cuerpo de la función, debemos declarar el
parametro en la firma para que el compilador sepa qué es lo que ese nombre en el
cuerpo significa. De manera similar, cuando usamos un nombre de tipo de parametro en una 
firma de función, debemos declarar el nombre de tipo de parametro antes de que lo usemos. Las declaraciones
de los nombres de los tipos van en paréntesis angulares entre el nombre de la función y
la lista de parametros.

La firma de la función de la función genérica `largest` que vamos a definir 
lucirá así:

```rust,ignore
fn largest<T>(list: &[T]) -> T {
```

Leeríamos esto como: la función `largest` es genérica sobre un tipo `T`. 
Tiene un parámetro llamado `list`, y el tipo de `list` es un pedazo de valores del
tipo `T`. La función `largest` dará un valor del mismo tipo `T`.

El listado 10-5 muestra la función unificada de `largest` usando tipos de datos genéricos
en su firma, y muestra cómo podríamos llamar a `largest` con
un pedazo de los valores de  `i32` o los valores de `char`. ¡Nota que este código no
compilará aun!

<span class="filename">Nombre del archivo: src/main.rs</span>

```rust,ignore
fn largest<T>(list: &[T]) -> T {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest(&number_list);
    println!("The largest number is {}", result);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest(&char_list);
    println!("The largest char is {}", result);
}
```

<span class="caption">Listado 10-5: Una definición de la función `largest` que 
usa tipos de parametros genéricos pero no compila aun</span>

Si tratásemos de compilar este código ahora mismo, nos daría este error:

```text
error[E0369]: binary operation `>` cannot be applied to type `T`
  |
5 |         if item > largest {
  |            ^^^^
  |
note: an implementation of `std::cmp::PartialOrd` might be missing for `T`
```

La nota menciona `std::cmp::PartialOrd`, el cual es un *rasgo*. Vamos a 
hablar sobre los rasgos en la siguiente sección, pero brevemente, lo que el error dice
es que el cuerpo de `largest` no trabajará por todos los tipos posibles que `T` podría 
ser; ya que nosotros queremos comparar los valores del tipo `T` en el cuerpo, solo podemos usar
tipos que sepan como ser ordenados. La libreria estandar ha definido el rasgo
`std::cmp::PartialOrd` que los tipos pueden implementar para habilitar las comparaciones. 
Volveremos a hablar de los rasgos y como especificar que un tipo genérico tiene un 
rasgo particular en la siguiente sección, pero veamos este ejemplo por un momento y
explorar otros lugares en donde podemos usar los tipos genéricos de parametros primero.

<!-- Liz: this is the reason we had the topics in the order we did in the first
draft of this chapter; it's hard to do anything interesting with generic types
in functions unless you also know about traits and trait bounds. I think this
ordering could work out okay, though, and keep a stronger thread with the
`longest` function going through the whole chapter, but we do pause with a
not-yet-compiling example here, which I know isn't ideal either. Let us know
what you think. /Carol -->

### Usando tipos de datos genéricos en definiciones de estructuras

Podemos definir estructuras para usar tipos de parametros genericos en uno o más de los
campos de estructura con la sintaxis `<>` también. El listado 10-6 muestra la definición y
el uso de una estructura `Point` que pueda albergar los valores de cordenadas `x` y `y` de cada tipo:

<span class="filename">Nombre del archivo: src/main.rs</span>

```rust
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let integer = Point { x: 5, y: 10 };
    let float = Point { x: 1.0, y: 4.0 };
}
```

<span class="caption">Listado 10-6: A `Point` estructura que `x` y `y`
albergan los valores del tipo `T`</span>

La sintaxis es similar a usar genéricos en definiciones de funciones. Primero, tenemos
que declarar el nombre del tipo de parametro dentro de parentesis angulares justo luego del
nombre de la estructura. Entonces podemos usar el tipo genérico en la definición de la estructura
donde especificariamos tipos de datos concretos.

Nota que ya que solo hemos usado un tipo genérico en la definición de 
`Point`, lo que estamos diciendo es que la estructura `Point` es genérica sobre un tipo
`T`, y los campos `x` y `y` son *ambos* ese mismo tipo, como sea que termine
siendo. Si tratamos de crear una instancia de un `Point` que tenga valores de
diferentes tipos, como en el listado 10-7, nuestro codigo no compilara:

<span class="filename">Nombre de archivo: src/main.rs</span>

```rust,ignore
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let wont_work = Point { x: 5, y: 4.0 };
}
```

<span class="caption">Listado 10-7: Los campos `x` y `y` deberan ser los mismos
tipos porque ambos tienen el mismo tipo de dato genérico `T`</span>

Si tratasemos de compilar esto, nos daría el siguiente error:

```text
error[E0308]: mismatched types
 -->
  |
7 |     let wont_work = Point { x: 5, y: 4.0 };
  |                                      ^^^ expected integral variable, found
  floating-point variable
  |
  = note: expected type `{integer}`
  = note:    found type `{float}`
```

Cuando le asignamos un valor entero de 5 a `x`, el compilador entonces sabe que para esta 
instancia de `Point` que el tipo genérico `T` será un número entero. Entonces nosotros
especificamos 4.0 para `y`, lo que es definido para que tenga el mismo tipo `x`, entonces tenemos un
error de desajuste de tipos.

Si quisieramos definir una estructura `Point` donde `x` y `y` pudieran tener diferentes
tipos pero aun haría que esos tipos fueran genéricos, podemos usar multiples parametros de tipos
genéricos. En el listado 10-8, hemos cambiado la definición de `Point` para que fuese
generica sobre tipos `T` y `U`. El campo `x` es de tipo `T`, y el campo `y`
es de tipo `U`:

<span class="filename">Nombre del archivo: src/main.rs</span>

```rust
struct Point<T, U> {
    x: T,
    y: U,
}

fn main() {
    let both_integer = Point { x: 5, y: 10 };
    let both_float = Point { x: 1.0, y: 4.0 };
    let integer_and_float = Point { x: 5, y: 4.0 };
}
```

<span class="caption">Listado 10-8: Un `Point` genérico sobre dos tipos para que 
`x` y `y` puedan ser valores de dos tipos diferentes</span>

¡Ahora todas estas instancias de `Point` están permitidas! Podrás usar cuantos tipos de parámetros
de tipo genérico quieras en una definición, pero el usar más de unos cuantos se hace
dificil de leer y entender. Si llegas al punto de necesitar muchos tipos
genéricos, es probablemente un signo de que tu código podría necesitar un poco de reestructuración para
ser separado en piezas más pequeñas.

### Using Generic Data Types in Enum Definitions

Similarly to structs, enums can be defined to hold generic data types in their
variants. We used the `Option<T>` enum provided by the standard library in
Chapter 6, and now its definition should make more sense. Let’s take another
look:

```rust
enum Option<T> {
    Some(T),
    None,
}
```

In other words, `Option<T>` is an enum generic in type `T`. It has two
variants: `Some`, which holds one value of type `T`, and a `None` variant that
doesn’t hold any value. The standard library only has to have this one
definition to support the creation of values of this enum that have any
concrete type. The idea of “an optional value” is a more abstract concept than
one specific type, and Rust lets us express this abstract concept without lots
of duplication.

Enums can use multiple generic types as well. The definition of the `Result`
enum that we used in Chapter 9 is one example:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

The `Result` enum is generic over two types, `T` and `E`. `Result` has two
variants: `Ok`, which holds a value of type `T`, and `Err`, which holds a value
of type `E`. This definition makes it convenient to use the `Result` enum
anywhere we have an operation that might succeed (and return a value of some
type `T`) or fail (and return an error of some type `E`). Recall Listing 9-2
when we opened a file: in that case, `T` was filled in with the type
`std::fs::File` when the file was opened successfully and `E` was filled in
with the type `std::io::Error` when there were problems opening the file.

When you recognize situations in your code with multiple struct or enum
definitions that differ only in the types of the values they hold, you can
remove the duplication by using the same process we used with the function
definitions to introduce generic types instead.

### Using Generic Data Types in Method Definitions

Like we did in Chapter 5, we can implement methods on structs and enums that
have generic types in their definitions. Listing 10-9 shows the `Point<T>`
struct we defined in Listing 10-6. We’ve then defined a method named `x` on
`Point<T>` that returns a reference to the data in the field `x`:

<span class="filename">Filename: src/main.rs</span>

```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}

fn main() {
    let p = Point { x: 5, y: 10 };

    println!("p.x = {}", p.x());
}
```

<span class="caption">Listing 10-9: Implementing a method named `x` on the
`Point<T>` struct that will return a reference to the `x` field, which is of
type `T`.</span>

Note that we have to declare `T` just after `impl` in order to use `T` in the
type `Point<T>`. Declaring `T` as a generic type after the `impl` is how Rust
knows the type in the angle brackets in `Point` is a generic type rather than a
concrete type. For example, we could choose to implement methods on
`Point<f32>` instances rather than `Point` instances with any generic type.
Listing 10-10 shows that we don’t declare anything after the `impl` in this
case, since we’re using a concrete type, `f32`:

```rust
# struct Point<T> {
#     x: T,
#     y: T,
# }
#
impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}
```

<span class="caption">Listing 10-10: Building an `impl` block which only
applies to a struct with a specific type is used for the generic type parameter
`T`</span>

This code means the type `Point<f32>` will have a method named
`distance_from_origin`, and other instances of `Point<T>` where `T` is not of
type `f32` will not have this method defined. This method measures how far our
point is from the point of coordinates (0.0, 0.0) and uses mathematical
operations which are only available for floating-point types.

Generic type parameters in a struct definition aren’t always the same generic
type parameters you want to use in that struct’s method signatures. Listing
10-11 defines a method `mixup` on the `Point<T, U>` struct from Listing 10-8.
The method takes another `Point` as a parameter, which might have different
types than the `self` `Point` that we’re calling `mixup` on. The method creates
a new `Point` instance that has the `x` value from the `self` `Point` (which is
of type `T`) and the `y` value from the passed-in `Point` (which is of type
`W`):

<span class="filename">Filename: src/main.rs</span>

```rust
struct Point<T, U> {
    x: T,
    y: U,
}

impl<T, U> Point<T, U> {
    fn mixup<V, W>(self, other: Point<V, W>) -> Point<T, W> {
        Point {
            x: self.x,
            y: other.y,
        }
    }
}

fn main() {
    let p1 = Point { x: 5, y: 10.4 };
    let p2 = Point { x: "Hello", y: 'c'};

    let p3 = p1.mixup(p2);

    println!("p3.x = {}, p3.y = {}", p3.x, p3.y);
}
```

<span class="caption">Listing 10-11: Methods that use different generic types
than their struct’s definition</span>

In `main`, we’ve defined a `Point` that has an `i32` for `x` (with value `5`)
and an `f64` for `y` (with value `10.4`). `p2` is a `Point` that has a string
slice for `x` (with value `"Hello"`) and a `char` for `y` (with value `c`).
Calling `mixup` on `p1` with the argument `p2` gives us `p3`, which will have
an `i32` for `x`, since `x` came from `p1`. `p3` will have a `char` for `y`,
since `y` came from `p2`. The `println!` will print `p3.x = 5, p3.y = c`.

Note that the generic parameters `T` and `U` are declared after `impl`, since
they go with the struct definition. The generic parameters `V` and `W` are
declared after `fn mixup`, since they are only relevant to the method.

### Performance of Code Using Generics

You may have been reading this section and wondering if there’s a run-time cost
to using generic type parameters. Good news: the way that Rust has implemented
generics means that your code will not run any slower than if you had specified
concrete types instead of generic type parameters!

Rust accomplishes this by performing *monomorphization* of code using generics
at compile time. Monomorphization is the process of turning generic code into
specific code with the concrete types that are actually used filled in.

What the compiler does is the opposite of the steps that we performed to create
the generic function in Listing 10-5. The compiler looks at all the places that
generic code is called and generates code for the concrete types that the
generic code is called with.

Let’s work through an example that uses the standard library’s `Option` enum:

```rust
let integer = Some(5);
let float = Some(5.0);
```

When Rust compiles this code, it will perform monomorphization. The compiler
will read the values that have been passed to `Option` and see that we have two
kinds of `Option<T>`: one is `i32`, and one is `f64`. As such, it will expand
the generic definition of `Option<T>` into `Option_i32` and `Option_f64`,
thereby replacing the generic definition with the specific ones.

The monomorphized version of our code that the compiler generates looks like
this, with the uses of the generic `Option` replaced with the specific
definitions created by the compiler:

<span class="filename">Filename: src/main.rs</span>

```rust
enum Option_i32 {
    Some(i32),
    None,
}

enum Option_f64 {
    Some(f64),
    None,
}

fn main() {
    let integer = Option_i32::Some(5);
    let float = Option_f64::Some(5.0);
}
```

We can write the non-duplicated code using generics, and Rust will compile that
into code that specifies the type in each instance. That means we pay no
runtime cost for using generics; when the code runs, it performs just like it
would if we had duplicated each particular definition by hand. The process of
monomorphization is what makes Rust’s generics extremely efficient at runtime.
