## Generic Data Types

Using generics where we usually place types, like in function signatures or
structs, lets us create definitions that we can use for many different concrete
data types. Let’s take a look at how to define functions, structs, enums, and
methods using generics, and at the end of this section we’ll discuss the
performance of code using generics.

### Using Generic Data Types in Function Definitions

We can define functions that use generics in the signature of the function
where the data types of the parameters and return value go. In this way, the
code we write can be more flexible and provide more functionality to callers of
our function, while not introducing code duplication.

Continuing with our `largest` function, Listing 10-4 shows two functions
providing the same functionality to find the largest value in a slice. The
first function is the one we extracted in Listing 10-3 that finds the largest
`i32` in a slice. The second function finds the largest `char` in a slice:

<span class="filename">Filename: src/main.rs</span>

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

<span class="caption">Listing 10-4: Two functions that differ only in their
names and the types in their signatures</span>

Here, the functions `largest_i32` and `largest_char` have the exact same body,
so it would be nice if we could turn these two functions into one and get rid
of the duplication. Luckily, we can do that by introducing a generic type
parameter!

To parameterize the types in the signature of the one function we’re going to
define, we need to create a name for the type parameter, just like how we give
names for the value parameters to a function. We’re going to choose the name
`T`. Any identifier can be used as a type parameter name, but we’re choosing
`T` because Rust’s type naming convention is CamelCase. Generic type parameter
names also tend to be short by convention, often just one letter. Short for
“type”, `T` is the default choice of most Rust programmers.

When we use a parameter in the body of the function, we have to declare the
parameter in the signature so that the compiler knows what that name in the
body means. Similarly, when we use a type parameter name in a function
signature, we have to declare the type parameter name before we use it. Type
name declarations go in angle brackets between the name of the function and the
parameter list.

The function signature of the generic `largest` function we’re going to define
will look like this:

```rust,ignore
fn largest<T>(list: &[T]) -> T {
```

We would read this as: the function `largest` is generic over some type `T`. It
has one parameter named `list`, and the type of `list` is a slice of values of
type `T`. The `largest` function will return a value of the same type `T`.

Listing 10-5 shows the unified `largest` function definition using the generic
data type in its signature, and shows how we’ll be able to call `largest` with
either a slice of `i32` values or `char` values. Note that this code won’t
compile yet!

<span class="filename">Filename: src/main.rs</span>

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

<span class="caption">Listing 10-5: A definition of the `largest` function that
uses generic type parameters but doesn’t compile yet</span>

If we try to compile this code right now, we’ll get this error:

```text
error[E0369]: binary operation `>` cannot be applied to type `T`
  |
5 |         if item > largest {
  |            ^^^^
  |
note: an implementation of `std::cmp::PartialOrd` might be missing for `T`
```

The note mentions `std::cmp::PartialOrd`, which is a *trait*. We’re going to
talk about traits in the next section, but briefly, what this error is saying
is that the body of `largest` won’t work for all possible types that `T` could
be; since we want to compare values of type `T` in the body, we can only use
types that know how to be ordered. The standard library has defined the trait
`std::cmp::PartialOrd` that types can implement to enable comparisons. We’ll
come back to traits and how to specify that a generic type has a particular
trait in the next section, but let’s set this example aside for a moment and
explore other places we can use generic type parameters first.

<!-- Liz: this is the reason we had the topics in the order we did in the first
draft of this chapter; it's hard to do anything interesting with generic types
in functions unless you also know about traits and trait bounds. I think this
ordering could work out okay, though, and keep a stronger thread with the
`longest` function going through the whole chapter, but we do pause with a
not-yet-compiling example here, which I know isn't ideal either. Let us know
what you think. /Carol -->

### Using Generic Data Types in Struct Definitions

We can define structs to use a generic type parameter in one or more of the
struct’s fields with the `<>` syntax too. Listing 10-6 shows the definition and
use of a `Point` struct that can hold `x` and `y` coordinate values of any type:

<span class="filename">Filename: src/main.rs</span>

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

<span class="caption">Listing 10-6: A `Point` struct that holds `x` and `y`
values of type `T`</span>

The syntax is similar to using generics in function definitions. First, we have
to declare the name of the type parameter within angle brackets just after the
name of the struct. Then we can use the generic type in the struct definition
where we would specify concrete data types.

Note that because we’ve only used one generic type in the definition of
`Point`, what we’re saying is that the `Point` struct is generic over some type
`T`, and the fields `x` and `y` are *both* that same type, whatever it ends up
being. If we try to create an instance of a `Point` that has values of
different types, as in Listing 10-7, our code won’t compile:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let wont_work = Point { x: 5, y: 4.0 };
}
```

<span class="caption">Listing 10-7: The fields `x` and `y` must be the same
type because both have the same generic data type `T`</span>

If we try to compile this, we’ll get the following error:

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

When we assigned the integer value 5 to `x`, the compiler then knows for this
instance of `Point` that the generic type `T` will be an integer. Then when we
specified 4.0 for `y`, which is defined to have the same type as `x`, we get a
type mismatch error.

If we wanted to define a `Point` struct where `x` and `y` could have different
types but still have those types be generic, we can use multiple generic type
parameters. In listing 10-8, we’ve changed the definition of `Point` to be
generic over types `T` and `U`. The field `x` is of type `T`, and the field `y`
is of type `U`:

<span class="filename">Filename: src/main.rs</span>

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

<span class="caption">Listing 10-8: A `Point` generic over two types so that
`x` and `y` may be values of different types</span>

Now all of these instances of `Point` are allowed! You can use as many generic
type parameters in a definition as you want, but using more than a few gets
hard to read and understand. If you get to a point of needing lots of generic
types, it’s probably a sign that your code could use some restructuring to be
separated into smaller pieces.

### Usando Tipos de Datos Genéricos en Definiciones Enum

Similares a las estructuras, Los Enums pueden ser definidos para albergar tipos de datos genéricos en sus
variantes. Usamos el enum `Option<T>` proporcionado por la biblioteca estándar en el
Capítulo 6, y ahora su definición debería de tener más sentido. Echemos
otro vistazo:

```rust
enum Option<T> {
    Some(T),
    None,
}
```

En otras palabras, `Option<T>` es un Enum genérico del tipo `T`. El cual tiene dos
variantes: `Some`, que alberga un valor de tipo `T`, y una variante `None` que 
no alberga ningún valor. La biblioteca estándar solo debe de tener esta única
definición para respaldar la creación de valores en este enum que tenga cualquier
tipo concreto. La idea de “un valor opcional” es más un concepto abstracto que
un tipo específico, y Rust nos deja expresar este concepto abstracto sin muchas
duplicaciones.

Enums también pueden usar múltiples tipos genéricos. La definición del enum `Result`
que usamos en el capítulo 9 es un ejemplo:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

El Enum `Result` es genérico sobre dos tipos, `T` y `E`. `Result` tiene dos
variantes: `Ok`, la cual alberga un valor de tipo `T`, y `Err`, el cual alberga un valor
de tipo `E`. Esta definición hace conveniente el usar el enum `Result` 
en donde queramos tener una operación que pueda tener éxito (y dar un valor de algún
tipo `T`) o fallar (y dar un error de algún tipo `E`). Recuerda el listado 9-2
donde abrimos un archivo: en ese caso, `T` estaba rellenado con el tipo
`std::fs::File` cuando el archivo se abría con éxito y `E` estaba rellenado
con el tipo `std::io::Error` cuando habían problemas abriendo el archivo.

Cuando reconoces situaciones en tu código con múltiple estructura o definiciones
Enum que eran diferentes solo en los tipos de los valores que albergaban, puedes
retirar la duplicación al usar el mismo proceso que usamos con las definiciones de
función para introducir tipos genéricos en su lugar.

### Usando Tipos de Datos Genéricos en Definiciones de Métodos

Como hicimos en el capítulo 5, podemos implementar métodos en estructuras y Enums que
tengan tipos genéricos en sus definiciones. El listado 10-9 muestra la estructura `Point<T>`
que definimos en el listado 10-6. Hemos entonces identificado un método llamado `x` en
`Point<T>` que da una referencia a los datos en el campo `x`:

<span class="filename">Number of archive: src/main.rs</span>

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

<span class="caption">Listado 10-9: Implementando un método llamado `x` en la estructura
`Point<T>` que dará una referencia al campo `x`, la cual es del 
tipo `T`.</span>

Nota que tenemos que declarar `T` justo luego de `impl` para usar `T` en el
tipo `Point<T>`. Declarando `T` como un tipo genérico antes del `impl` es como Rust
sabe que el tipo en los corchetes angulares en `Point` es un tipo genérico en lugar de un
tipo concreto. por ejemplo, podríamos escoger el implementar métodos en instancias
`Point<f32>` en lugar de instancias `Point` con cualquier tipo genérico.
El listado 10-10 muestra que no declaramos nada luego del `impl` en este 
caso, ya que estamos usando el tipo concreto, `f32`:

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

<span class="caption">Listado 10-10: Construyendo un bloqueo `impl`que solo 
aplica a una estructura con un tipo específico es usada para el parámetro de tipo genérico
 `T`</span>

Este código significa que el tipo `Point<f32>` tendrá un método llamado
`distance_from_origin`, y otras instancias de `Point<T>` donde `T` no es de 
tipo `32` no tendrá este método definido. Este método mide que tan lejos es nuestro
punto del punto de las coordinadas (0.0, 0.0) y usa operaciones
matemáticas que solo son disponibles para tipos de punto flotante-

parámetros de tipo genérico en una definición de estructura no son siempre el mismo 
parámetro de tipo que tú quieres usar en firmas de método de estructura. El listado
10-11 define un método `mixup` en la estructura `Point<T, U>` desde el listado 10-8.
El método toma otro `Point` como un parámetro, el que puede tener diferentes 
tipos que el `self` `Point` que estamos nombrando en `mixup`. El método crea
una nueva instancia `Point` que tiene el valor `x` del `self` `Point` (que es
de tipo `T`) y el valor `y` del `Point` aprobado (el cual es de tipo
 `W`):

<span class="filename">Nombre del archivo: src/main.rs</span>

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

<span class="caption">Listado 10-11: Metodos que usan diferentes tipos genéricos
que su definición de estructura</span>

En `main`, hemos definido un `Point` que tiene un `i32` por `x` (con valor `5`)
y un `f64` por `y` (con valor `10.4`). `p2` es un `Point` que tiene un pedazo de cordón de `x` (con valor `"Hello"`) y un `char` por `y` (con valor `c`).
El llamar `mixup` en `p1` con el argumento `p2` nos da `p3`, que tendrá
un `i32` por `x`, ya que `x` vino de `p1`. `p3` tendrá un `char` por `y`,
ya que `y` vino de `p2`. El `println!` emitirá un `p3.x = 5, p3.y = c`.

Date cuenta de que los parámetros genéricos `T` y `U` son declarados después de `impl`, ya que
ellos van con la definición de la estructura. Los parámetros genéricos `V` y `W` son
declarados luego de `fn mixup`, que ellos solo son relevantes para el método.

### El Desempeño del Código Usando Genéricos

Te estarás preguntando mientras leías esta sección si hay un tiempo de coste en el tiempo de ejecución
al usar tipos de parámetros genéricos. Tengo buenas noticias: ¡La forma en la que Rust ha implementado los
genéricos significa que tu código no se ejecutará más lento como si hubieses especificado
tipos concretos en vez de tipos de parámetros genéricos!

Rust logra esto al realizar una *monotransformación* del código usando los genéricos
al tiempo de compilar. La monotransformación es el proceso de convertir un código genérico en un
código especificado con tipos concretos que de hecho se usan rellenos.


Lo que el compilador hace es lo opuesto de los pasos que hemos realizado para crear
la función genérica en el listado 10-5. El compilador mira por todos los lugares que el
código genérico ha sido llamado y genera códigos para los tipos concretos con los que el
código genérico es llamado.

Trabajemos con un ejemplo que usa el enum `Option` en la biblioteca estándar:

```rust
let integer = Some(5);
let float = Some(5.0);
```

Cuando Rust compila este código, este realiza la monotransformación. El compilador
leerá los valores que han sido pasados a `Option` y verá que tenemos dos
tipos de `Option<T>`: uno es `i32`, y uno es `f64`. Y así, expandirá
la definición genérica de `Option<T>` en `Option_i32` y `Option_f64`,
y así reemplazando la definición genérica con las especificadas.

La versión monotransformada de nuestro código que el compilador genera lucirá
así, con los usos del genérico `Option` reemplazados con las definiciones
especificas creadas por el compilador:

<span class="filename">Nombre del archivo: src/main.rs</span>

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

Podemos escribir códigos no duplicados usando genéricos, y Rust compilará eso
en un código que especifica el tipo en cada instancia. eso significa que no pagamos
costos de tiempos de ejecución por usar genéricos; cuando el código se ejecuta, se lleva a cabo como si
lo hubiésemos duplicado cada definición particular a mano. El proceso de 
monotransformación es lo que hace los genéricos de Rust extremamente eficientes en el tiempo de ejecución.
