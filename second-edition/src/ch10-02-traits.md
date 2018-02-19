## Traits: Defining Shared Behavior

Traits allow us to use another kind of abstraction: they let us abstract over
behavior that types can have in common. A *trait* tells the Rust compiler about
functionality a particular type has and might share with other types. In
situations where we use generic type parameters, we can use *trait bounds* to
specify, at compile time, that the generic type may be any type that implements
a trait and therefore has the behavior we want to use in that situation.

> Note: *Traits* are similar to a feature often called ‘interfaces’ in other
> languages, though with some differences.

### Defining a Trait

The behavior of a type consists of the methods we can call on that type.
Different types share the same behavior if we can call the same methods on all
of those types. Trait definitions are a way to group method signatures together
in order to define a set of behaviors necessary to accomplish some purpose.

For example, say we have multiple structs that hold various kinds and amounts
of text: a `NewsArticle` struct that holds a news story filed in a particular
place in the world, and a `Tweet` that can have at most 140 characters in its
content along with metadata like whether it was a retweet or a reply to another
tweet.

We want to make a media aggregator library that can display summaries of data
that might be stored in a `NewsArticle` or `Tweet` instance. The behavior we
need each struct to have is that it’s able to be summarized, and that we can
ask for that summary by calling a `summary` method on an instance. Listing
10-12 shows the definition of a `Summarizable` trait that expresses this
concept:

<span class="filename">Filename: lib.rs</span>

```rust
pub trait Summarizable {
    fn summary(&self) -> String;
}
```

<span class="caption">Listing 10-12: Definition of a `Summarizable` trait that
consists of the behavior provided by a `summary` method</span>

We declare a trait with the `trait` keyword, then the trait’s name, in this
case `Summarizable`. Inside curly brackets we declare the method signatures
that describe the behaviors that types that implement this trait will need to
have, in this case `fn summary(&self) -> String`. After the method signature,
instead of providing an implementation within curly brackets, we put a
semicolon. Each type that implements this trait must then provide its own
custom behavior for the body of the method, but the compiler will enforce that
any type that has the `Summarizable` trait will have the method `summary`
defined for it with this signature exactly.

A trait can have multiple methods in its body, with the method signatures
listed one per line and each line ending in a semicolon.

### Implementing a Trait on a Type

Now that we’ve defined the `Summarizable` trait, we can implement it on the
types in our media aggregator that we want to have this behavior. Listing 10-13
shows an implementation of the `Summarizable` trait on the `NewsArticle` struct
that uses the headline, the author, and the location to create the return value
of `summary`. For the `Tweet` struct, we’ve chosen to define `summary` as the
username followed by the whole text of the tweet, assuming that tweet content
is already limited to 140 characters.

<span class="filename">Filename: lib.rs</span>

```rust
# pub trait Summarizable {
#     fn summary(&self) -> String;
# }
#
pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summarizable for NewsArticle {
    fn summary(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}

impl Summarizable for Tweet {
    fn summary(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}
```

<span class="caption">Listing 10-13: Implementing the `Summarizable` trait on
the `NewsArticle` and `Tweet` types</span>

Implementing a trait on a type is similar to implementing methods that aren’t
related to a trait. The difference is after `impl`, we put the trait name that
we want to implement, then say `for` and the name of the type that we want to
implement the trait for. Within the `impl` block, we put the method signatures
that the trait definition has defined, but instead of putting a semicolon after
each signature, we put curly brackets and fill in the method body with the
specific behavior that we want the methods of the trait to have for the
particular type.

Once we’ve implemented the trait, we can call the methods on instances of
`NewsArticle` and `Tweet` in the same manner that we call methods that aren’t
part of a trait:

```rust,ignore
let tweet = Tweet {
    username: String::from("horse_ebooks"),
    content: String::from("of course, as you probably already know, people"),
    reply: false,
    retweet: false,
};

println!("1 new tweet: {}", tweet.summary());
```

This will print `1 new tweet: horse_ebooks: of course, as you probably already
know, people`.

Note that because we’ve defined the `Summarizable` trait and the `NewsArticle`
and `Tweet` types all in the same `lib.rs` in Listing 10-13, they’re all in the
same scope. If this `lib.rs` is for a crate we’ve called `aggregator`, and
someone else wants to use our crate’s functionality plus implement the
`Summarizable` trait on their `WeatherForecast` struct, their code would need
to import the `Summarizable` trait into their scope first before they could
implement it, like in Listing 10-14:

<span class="filename">Filename: lib.rs</span>

```rust,ignore
extern crate aggregator;

use aggregator::Summarizable;

struct WeatherForecast {
    high_temp: f64,
    low_temp: f64,
    chance_of_precipitation: f64,
}

impl Summarizable for WeatherForecast {
    fn summary(&self) -> String {
        format!("The high will be {}, and the low will be {}. The chance of
        precipitation is {}%.", self.high_temp, self.low_temp,
        self.chance_of_precipitation)
    }
}
```

<span class="caption">Listing 10-14: Bringing the `Summarizable` trait from our
`aggregator` crate into scope in another crate</span>

This code also assumes `Summarizable` is a public trait, which it is because we
put the `pub` keyword before `trait` in Listing 10-12.

One restriction to note with trait implementations: we may implement a trait on
a type as long as either the trait or the type are local to our crate. In other
words, we aren’t allowed to implement external traits on external types. We
can’t implement the `Display` trait on `Vec`, for example, since both `Display`
and `Vec` are defined in the standard library. We are allowed to implement
standard library traits like `Display` on a custom type like `Tweet` as part of
our `aggregator` crate functionality. We could also implement `Summarizable` on
`Vec` in our `aggregator` crate, since we’ve defined `Summarizable` there. This
restriction is part of what’s called the *orphan rule*, which you can look up
if you’re interested in type theory. Briefly, it’s called the orphan rule
because the parent type is not present. Without this rule, two crates could
implement the same trait for the same type, and the two implementations would
conflict: Rust wouldn’t know which implementation to use. Because Rust enforces
the orphan rule, other people’s code can’t break your code and vice versa.

### Default Implementations

Sometimes it’s useful to have default behavior for some or all of the methods
in a trait, instead of making every implementation on every type define custom
behavior. When we implement the trait on a particular type, we can choose to
keep or override each method’s default behavior.

El Listado 10-15 muestra cómo podríamos haber elegido especificar una cadena de caraceteres por defecto para
el método `summary` del rasgo `Summarizable` en lugar de elegir sólo
definir la firma del método como hicimos en el Listado 10-12:

<span class="filename">Filename: lib.rs</span>

```rust
pub trait Summarizable {
    fn summary(&self) -> String {
        String::from("(Read more...)")
    }
}
```

<span class="caption">El Listado 10-15: Definición de un rasgo `Summarizable` con
una implementación por defecto del método `summary`</span>

Si quisieramos esta implementación por defecto para resumir instancias de
`NewsArticle` en lugar de definir una impplementación personalizada como hicimos en
el Listado 10-13, especificaríamos un bloque vacío `impl`:

```rust,ignore
impl Summarizable for NewsArticle {}
```

Incluso cuando ya no elegimos definir el método `summary` en
`NewsArticle` directamente, ya que el método `summary` tiene una implementación por defecto
y que especificamos que `NewsArticle` implementa el rasgo `Summarizable`, podemos
seguir convocando el método `summary` en una instancia de `NewsArticle`:

```rust,ignore
let article = NewsArticle {
    headline: String::from("Penguins win the Stanley Cup Championship!"),
    location: String::from("Pittsburgh, PA, USA"),
    author: String::from("Iceburgh"),
    content: String::from("The Pittsburgh Penguins once again are the best
    hockey team in the NHL."),
};

println!("New article available! {}", article.summary());
```

Este código pone `New article available! (Read more...)`.

Cambiar el rasgo `Summarizable` para tener una implementación por defecto de
`summary` no nos pide cambiar nada sobre las implementaciones de
`Summarizable` en `Tweet` en el Listado 10-13 o `WeatherForecast` en el Listado
10-14: la sintaxis para invalidar a una implementación por defecto es exactamente la misma
a la sintaxis para implementar un método de rasgo que no tiene una implementación
por defecto.

Las implementaciones por defetos pueden convocar los otro métodos en el mismo
rasgo, incluso si esos otros métodos no tienen una implementación por defecto. De esta
manera, un rasgo puede proporcionar un montón de funcionalidades útiles y sólo requiere
implementadores para específicar una pequeña parte de él. Podríamos elegir tener el
rasgo `Summarizable` y también tener un método `author_summary` cuya implementación
es requerida, entonces, un método `summary` que tiene una implementación por defecto que
convoca el método `author_summary`:

```rust
pub trait Summarizable {
    fn author_summary(&self) -> String;

    fn summary(&self) -> String {
        format!("(Read more from {}...)", self.author_summary())
    }
}
```

Con el fin de utilizar esta versión de `Summarizable`, sólo nos piden definir
`author_summary` cuando implementamos el rasgo en un tipo:

```rust,ignore
impl Summarizable for Tweet {
    fn author_summary(&self) -> String {
        format!("@{}", self.username)
    }
}
```

Una vez que definimos `author_summary`, podemos convocar `summary` en instancias de la
estructura `Tweet`, y la implementación por defecto de `summary` convocará la
definicion de `author_summary` que hemos proporcionado.

```rust,ignore
let tweet = Tweet {
    username: String::from("horse_ebooks"),
    content: String::from("of course, as you probably already know, people"),
    reply: false,
    retweet: false,
};

println!("1 new tweet: {}", tweet.summary());
```

Esto pondrá `1 new tweet: (Read more from @horse_ebooks...)`.

Note que no es posible convocar la implementación por defecto desde una
implementación inválida.

### Los límites de los rasgos

Ahora que hemos definidos los ragos y los hemos implementado en los tipos, podemos usar
rasgos con parámetros de tipo genéricos. Podemos restringir a los tipos genéricos y así
en vez de ser cualquier tipo, el compilador se asegurará de que el tipo será
limitado a esos tipos que implementan un rasgo particular y de este modo tener el
comportamiento que necesitamos que tengan los tipos. Esto se convoca especificando *trait
bounds* en un tipo genérico.

Por ejemplo, en el Listado 10-13, implementamos el rasgo `Summarizable` en los
tipos `NewsArticle` y `Tweet`. Podemos definir una función `notify` que convoque
el método `summary` en su parámetro `item`, el cual es del tipo genérico `T`.
Para poder convocar `summary` en `item` sin tener ningún error, podemos usar
los límites del rasgo en `T` para especificar que `item` debe ser un tipo que implementa
el rasgo `Summarizable`:

```rust,ignore
pub fn notify<T: Summarizable>(item: T) {
    println!("Breaking news! {}", item.summary());
}
```

Los límites de rasgo van con la declaración del parámetro de tipo genérico, después de un
dos puntos y dentro de las comillas angulares. Por el límite de rasgo en `T`, podemos convocar
 `notify` y pasar cualquier instancia de `NewsArticle` o `Tweet`. El
código externo del Listado 10-14 que está usando nuestro cajón `aggregator` puede convocar
nuestra función `notify` y pasar una instancia de `WeatherForecast`, ya que
`Summarizable` es implementado por `WeatherForecast` también. El código que convoca
`notify` con cualquier otro tipo, como un `String` o una `i32`, no compilará, ya que
esos tipos no implementan `Summarizable`.

Podemos específicar múltiples límites de rasgo en un tipo genérico usando `+`. Si nosotros
necesitáramos ser capaces de usar el formato de salida en el tipo `T` en una funcion al
igual que el método `summary`, podemos usar los límites de rasgo `T: Summarizable +
Display`. Esto significa que `T` puede ser cualquier tipo que implemente a `Summarizable`
y a `Display`.

Para funciones que tienen multiples parámetros de tipo genéricos, cada genérico tiene su
propio límites de rasgo. Especificando montones de información de límite de rasgo en las comillas
angulares entre un nombre de función y su lista de parámetros puede ser difícil de leer,
así que hay una sintaxis alternativa para especificar límites de rasgo que nos permitan moverlos
a una condición `where` luego de la firma de función. Así que en lugar de:

```rust,ignore
fn some_function<T: Display + Clone, U: Clone + Debug>(t: T, u: U) -> i32 {
```

Podemos escribir esto en su lugar con una condición `where`:

```rust,ignore
fn some_function<T, U>(t: T, u: U) -> i32
    where T: Display + Clone,
          U: Clone + Debug
{
```

Esto es menos desordenado y hace que esta firma de función se vea más parecida a
una función sin montones de límites de rasgo, ya que el nombre de la función, la lista de 
parámetros, y el tipo de retorno son muy cercanos.

### Arreglar la función `largest` con Límites de Rasgo

Así que siempre que quieras usar un comportamiendo definido por un rasgo en un genérico, necesitas
especificar ese rasgo en el parámetro genérico de tipo de los límites de tipo. ¡Ahora podemos
arreglar la definición de la función `largest` que usa un parámetro genérico de tipo
del Listado 10-5! Cuando establecemos ese código a un lado, obtendremos este error:

```text
error[E0369]: binary operation `>` cannot be applied to type `T`
  |
5 |         if item > largest {
  |            ^^^^
  |
note: an implementation of `std::cmp::PartialOrd` might be missing for `T`
```

En el cuerpo de `largest` quisimos ser capaces de comprar dos valores del tipo `T`
usando el operador mayor que. Ese operador es definido como un método predeterminado
en la biblioteca estándar de rasgo `std::cmp::PartialOrd`. Así que con el fin de ser capaz de
usar el operador mayor que, necesitamos especificar el `PartialOrd` en los límites
de rasgo para `T` y así la función `largest` funcionará en los pedazos de cualquier tipo
que pueda ser comparado. No necesitamos traer `PartialOrd` a la mira porque
está en el preludio.

```rust,ignore
fn largest<T: PartialOrd>(list: &[T]) -> T {
```

Si intentamos compilar esto, obtendremos distintos errores:

```text
error[E0508]: cannot move out of type `[T]`, a non-copy array
 --> src/main.rs:4:23
  |
4 |     let mut largest = list[0];
  |         -----------   ^^^^^^^ cannot move out of here
  |         |
  |         hint: to prevent move, use `ref largest` or `ref mut largest`

error[E0507]: cannot move out of borrowed content
 --> src/main.rs:6:9
  |
6 |     for &item in list.iter() {
  |         ^----
  |         ||
  |         |hint: to prevent move, use `ref item` or `ref mut item`
  |         cannot move out of borrowed content
```

La clave para este error es `cannot move out of type [T], a non-copy array`.
Con nuestras versiones no genéricas de la función `largest`, sólo estamos intentando
encontrar el más largo `i32` o `char`. Como discutimos en el Capítulo Chapter 4, los tipos como
`i32` y `char` que tienen un tamaño conocido pueden ser almacenadas en el montón, así que ellos
implementan el rasgo `Copy`. Cuando cambiamos la función `largest` a ser
genérica, ahora es posible que el parámetro `list` podría tener tipos en él
que no implementan el rasgo `Copy`, lo que significa que no seremos capaces de mover
valor fuera de `list[0]` y dentro de la variable `largest`.

If we only want to be able to call this code with types that are `Copy`, we can
add `Copy` to the trait bounds of `T`! Listing 10-16 shows the complete code of
a generic `largest` function that will compile as long as the types of the
values in the slice that we pass into `largest` implement both the `PartialOrd`
and `Copy` traits, like `i32` and `char`:

<span class="filename">Filename: src/main.rs</span>

```rust
fn largest<T: PartialOrd + Copy>(list: &[T]) -> T {
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

<span class="caption">Listing 10-16: A working definition of the `largest`
function that works on any generic type that implements the `PartialOrd` and
`Copy` traits</span>

If we don’t want to restrict our `largest` function to only types that
implement the `Copy` trait, we could specify that `T` has the trait bound
`Clone` instead of `Copy` and clone each value in the slice when we want the
`largest` function to have ownership. Using the `clone` function means we’re
potentially making more heap allocations, though, and heap allocations can be
slow if we’re working with large amounts of data. Another way we could
implement `largest` is for the function to return a reference to a `T` value in
the slice. If we change the return type to be `&T` instead of `T` and change
the body of the function to return a reference, we wouldn’t need either the
`Clone` or `Copy` trait bounds and we wouldn’t be doing any heap allocations.
Try implementing these alternate solutions on your own!

### Using Trait Bounds to Conditionally Implement Methods

By using a trait bound with an `impl` block that uses generic type parameters,
we can conditionally implement methods only for types that implement the
specified traits. For example, the type `Pair<T>` in listing 10-17 always
implements the `new` method, but `Pair<T>` only implements the `cmp_display` if
its inner type `T` implements the `PartialOrd` trait that enables comparison
and the `Display` trait that enables printing:

```rust
use std::fmt::Display;

struct Pair<T> {
    x: T,
    y: T,
}

impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self {
        Self {
            x,
            y,
        }
    }
}

impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
}
```

<span class="caption">Listing 10-17: Conditionally implement methods on a
generic type depending on trait bounds</span>

We can also conditionally implement a trait for any type that implements a
trait. Implementations of a trait on any type that satisfies the trait bounds
are called *blanket implementations*, and are extensively used in the Rust
standard library. For example, the standard library implements the `ToString`
trait on any type that implements the `Display` trait. This `impl` block looks
similar to this code:

```rust,ignore
impl<T: Display> ToString for T {
    // --snip--
}
```

Because the standard library has this blanket implementation, we can call the
`to_string` method defined by the `ToString` trait on any type that implements
the `Display` trait. For example, we can turn integers into their corresponding
`String` values like this since integers implement `Display`:

```rust
let s = 3.to_string();
```

Blanket implementations appear in the documentation for the trait in the
“Implementors” section.

Traits and trait bounds let us write code that uses generic type parameters in
order to reduce duplication, but still specify to the compiler exactly what
behavior our code needs the generic type to have. Because we’ve given the trait
bound information to the compiler, it can check that all the concrete types
used with our code provide the right behavior. In dynamically typed languages,
if we tried to call a method on a type that the type didn’t implement, we’d get
an error at runtime. Rust moves these errors to compile time so that we’re
forced to fix the problems before our code is even able to run. Additionally,
we don’t have to write code that checks for behavior at runtime since we’ve
already checked at compile time, which improves performance compared to other
languages without having to give up the flexibility of generics.

There’s another kind of generics that we’ve been using without even realizing
it called *lifetimes*. Rather than helping us ensure that a type has the
behavior we need it to have, lifetimes help us ensure that references are valid
as long as we need them to be. Let’s learn how lifetimes do that.
