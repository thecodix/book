## Rasgo: Definiendo comportamientos compartidos

Las rasgos permiten usar otro tipo de abstracción: permiten abstraernos sobre
el comportamiento que los tipos pueden tener en común. Un *trait* hace saber al compilador de Rust sobre
la funcionalidad que tiene un tipo particular y que podría compartir con otros tipos. En
situaciones en las que se usan parámetros genéricos de tipo, puede usarse *trait bounds* para
especificar, al momento de compilar, que el tipo genérico puede tener cualquier tipo que implemente
una característica y que, por lo tanto, posee el comportamiento que deseamos usar en esa situación.

> Nota: Las *Traits* son similares a un rasgo comúmente conocida como ‘interfaces’ en otros
> lenguajes, aun que con algunas diferencias.

### Definiendo un Rasgo

El comportamiento de un tipo consiste en los métodos que se convoque en ese tipo.
Tipos distintos comparten el mismo comportamiento cuando es posible convocar los mismos métodos en todos
esos tipos. Las definiciones de rasgos son una forma de agrupar firmas de métodos
con la finalidad de definir un conjunto de comportamientos necesarios para alcanzar un propósito.

Por ejemplo, digamos que tenemos múltiples estructuras que mantienen diversos tipos y cantidades
de texto: Una estructura `NewsArticle` que mantiene un historial archivado en un lugar 
particular en el mundo, y un `Tweet` que puede tener un máximo de 140 caracteres en su
contenido dentro de los metadatos, sin importar si fue un retweet o una respuesta hecha a otro tweet.

Queremos crear una biblioteca agregadora de redes sociales que exhiba resúmenes de datos
que podrían ser almacenados en un `NewsArticle` o en un `Tweet`. Cada estructura necesita tener
un comportamiento que le permita ser resumido y que pueda
ser pedido convocando un método `summary` en una instancia. El Listado
10-12 demuestra la definición de un rasgo `Summarizable` que expresa este concepto:

<span class="filename">Filename: lib.rs</span>

```rust
pub trait Summarizable {
    fn summary(&self) -> String;
}
```

<span class="caption">Listing 10-12: Definition of a `Summarizable` trait that
consists of the behavior provided by a `summary` method</span>

Declaramos un rasgo con la palabra clave `trait` y, seguidamente, el nombre de el rasgo, en este
caso `Summarizable`. Dentro de llaves, declaramos las firmas de los métodos
que describen los comportamientos que los tipos que implementan este rasgo necesitarán, 
en este caso `fn summary(&self) -> String`. Después de la firma de método,
en lugar de proveer una implementación dentro de llaves, se usa un
punto y coma. Cada tipo que implementa este rasgo deberá proveer su propio
comportamiento personalizado para el cuerpo del método, pero el compilador se asegurará de que
cada tipo que tiene la característica `Summarizable` tenga el método `summary`
definido para el mismo con esta firma exacta.

Un rasgo puede tener múltiples métodos en su cuerpo, con las firmas del método
en una lista de una por línea y cada línea terminando en un punto y coma.

### Implementando un Rasgo en un tipo

Ahora que se ha definido el rasgo `Summarizable`, es posible implementarla en los
tipos que queramos que tengan este comportamiento en nuestro agregador de redes sociales. El Listado 10-13
muestra una implementación de el rasgo `Summarizable` en la estructura `NewsArticle`
que emplea el encabezado, el autor, y la ubicación para crear el valor de retorno
de `summary`. Para la estructura de `Tweet`, se ha escogido definir `summary` como el
nombre de usuario seguido del texto completo del tweet, asumiendo que el contenido del tweet
ya se encuentra limitado a 140 caracteres.

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

Implementar una característica en un tipo es similar a implementar métodos que no están
relacionados con un rasgo. La diferencia está en que después de `impl`, se pone el nombre de el rasgo que
se desea implementar, luego se pone `for` y el nombre del tipo al que deseamos
implementarle el rasgo. Dentro del bloque `impl`, se ponen las firmas del método
que la definición de el rasgo ha definido, pero en lugar de poner un punto y coma después de
cada firma, se ponen llaves y se llena el cuerpo del método con el
comportamiento específico que se quiera que los métodos de el rasgo tengan para el
tipo particular.

Una vez que se ha implementado el rasgo, podemos convocar los métodos en instancias de
`NewsArticle` y `Tweet` de la misma forma en que convocamos los métodos que no forman
parte de una característica:

```rust,ignore
let tweet = Tweet {
    username: String::from("horse_ebooks"),
    content: String::from("of course, as you probably already know, people"),
    reply: false,
    retweet: false,
};

println!("1 new tweet: {}", tweet.summary());
```

Esto pondrá `1 new tweet: horse_ebooks: of course, as you probably already
know, people`.

Tome en cuenta que debido a que hemos definido el rasgo `Summarizable` y los tipos `NewsArticle`
y `Tweet` dentro del mismo `lib.rs` en el Listado 10-13, todos están dentro
del mismo campo de acción. Si este `lib.rs` es para un cajón que hemos llamado `aggregator` y
alguien más desea usar la funcionabilidad de nuestro cajón e implementar el rasgo
`Summarizable` en su estructura `WeatherForecast`, su código necesitaría
importar el rasgo `Summarizable` en su campo de acción antes de poder
implementarlo, como en el Listado 10-14:

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

<span class="caption"> El Listado 10-14: Trayendo el rasgo `Summarizable` desde nuestro
cajón `aggregator` dentro del campo de acción en otro cajón</span>

Este código también asume que `Summarizable` es una característica pública, lo cual se debe a que
pone la palabra clave `pub` antes de `trait` en el Listado 10-12.

Una reestricción a tener en cuenta con la implementación de rasgo: se puede implementar un rasgo en 
un tipo siempre y cuando el rasgo o el tipo sean locales para nuestro cajón. En otras
palabras, no está permitido implementar rasgos externos en tipos externos. No se puede
implementar la característica `Display` en `Vec`, por ejemplo, ya que `Display`
y `Vec` son definidos en la biblioteca estándar. Está permitido implementar
rasgos de la biblioteca estándar como `Display` en un tipo personalizado como `Tweet` como parte de
la funcionalidad de cajón de nuestro `aggregator`. También podría implementarse `Summarizable` en
`Vec` en nuestro cajón `aggregator`, ya que allí se ha definido `Summarizable`. Esta
restricción es parte de lo que se conoce como *orphan rule*, la cual puede ser consultada
si este tipo de teorías son de su interés. En resumen, se llama regla huérfana
porque el tipo parental no está presente. Sin esta regla, dos cajones pueden
implementar el mismo rasgo para el mismo tipo, y las dos implementaciones entrarían
en conflicto: el Rust no sabría cuál implementación usar. Ya que el Rust hace cumplir
la regla huérfana, los códigos de otras personas no pueden romper su código y viceversa.

### Implementaciones por defecto

Algunas veces, es útil tener un comportamiento por defecto para algunos o todos los métodos
en un rasgo,  en lugar de crear cada implementación en cada uno de los comportamientos personalizados de tipos
definidos. Cuando se implementa el rasgo en un tipo particular, se puede elegir conservar 
o saltarse el comportamiento por defecto de cada método.

El Listado 10-15 muestra cómo podríamos haber elegido especificar una cadena de caracteres por defecto para
el método `summary` de el rasgo `Summarizable` en lugar de elegir solo definir 
la firma de un método como se hizo en el Listado 10-12:

<span class="filename">Filename: lib.rs</span>

```rust
pub trait Summarizable {
    fn summary(&self) -> String {
        String::from("(Read more...)")
    }
}
```

<span class="caption">Listing 10-15: Definition of a `Summarizable` trait with
a default implementation of the `summary` method</span>

If we wanted to use this default implementation to summarize instances of
`NewsArticle` instead of defining a custom implementation like we did in
Listing 10-13, we would specify an empty `impl` block:

```rust,ignore
impl Summarizable for NewsArticle {}
```

Even though we’re no longer choosing to define the `summary` method on
`NewsArticle` directly, since the `summary` method has a default implementation
and we specified that `NewsArticle` implements the `Summarizable` trait, we can
still call the `summary` method on an instance of `NewsArticle`:

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

This code prints `New article available! (Read more...)`.

Changing the `Summarizable` trait to have a default implementation for
`summary` does not require us to change anything about the implementations of
`Summarizable` on `Tweet` in Listing 10-13 or `WeatherForecast` in Listing
10-14: the syntax for overriding a default implementation is exactly the same
as the syntax for implementing a trait method that doesn’t have a default
implementation.

Default implementations are allowed to call the other methods in the same
trait, even if those other methods don’t have a default implementation. In this
way, a trait can provide a lot of useful functionality and only require
implementors to specify a small part of it. We could choose to have the
`Summarizable` trait also have an `author_summary` method whose implementation
is required, then a `summary` method that has a default implementation that
calls the `author_summary` method:

```rust
pub trait Summarizable {
    fn author_summary(&self) -> String;

    fn summary(&self) -> String {
        format!("(Read more from {}...)", self.author_summary())
    }
}
```

In order to use this version of `Summarizable`, we’re only required to define
`author_summary` when we implement the trait on a type:

```rust,ignore
impl Summarizable for Tweet {
    fn author_summary(&self) -> String {
        format!("@{}", self.username)
    }
}
```

Once we define `author_summary`, we can call `summary` on instances of the
`Tweet` struct, and the default implementation of `summary` will call the
definition of `author_summary` that we’ve provided.

```rust,ignore
let tweet = Tweet {
    username: String::from("horse_ebooks"),
    content: String::from("of course, as you probably already know, people"),
    reply: false,
    retweet: false,
};

println!("1 new tweet: {}", tweet.summary());
```

This will print `1 new tweet: (Read more from @horse_ebooks...)`.

Note that it is not possible to call the default implementation from an
overriding implementation.

### Trait Bounds

Now that we’ve defined traits and implemented those traits on types, we can use
traits with generic type parameters. We can constrain generic types so that
rather than being any type, the compiler will ensure that the type will be
limited to those types that implement a particular trait and thus have the
behavior that we need the types to have. This is called specifying *trait
bounds* on a generic type.

For example, in Listing 10-13, we implemented the `Summarizable` trait on the
types `NewsArticle` and `Tweet`. We can define a function `notify` that calls
the `summary` method on its parameter `item`, which is of the generic type `T`.
To be able to call `summary` on `item` without getting an error, we can use
trait bounds on `T` to specify that `item` must be of a type that implements
the `Summarizable` trait:

```rust,ignore
pub fn notify<T: Summarizable>(item: T) {
    println!("Breaking news! {}", item.summary());
}
```

Trait bounds go with the declaration of the generic type parameter, after a
colon and within the angle brackets. Because of the trait bound on `T`, we can
call `notify` and pass in any instance of `NewsArticle` or `Tweet`. The
external code from Listing 10-14 that’s using our `aggregator` crate can call
our `notify` function and pass in an instance of `WeatherForecast`, since
`Summarizable` is implemented for `WeatherForecast` as well. Code that calls
`notify` with any other type, like a `String` or an `i32`, won’t compile, since
those types do not implement `Summarizable`.

We can specify multiple trait bounds on a generic type by using `+`. If we
needed to be able to use display formatting on the type `T` in a function as
well as the `summary` method, we can use the trait bounds `T: Summarizable +
Display`. This means `T` can be any type that implements both `Summarizable`
and `Display`.

For functions that have multiple generic type parameters, each generic has its
own trait bounds. Specifying lots of trait bound information in the angle
brackets between a function’s name and its parameter list can get hard to read,
so there’s an alternate syntax for specifying trait bounds that lets us move
them to a `where` clause after the function signature. So instead of:

```rust,ignore
fn some_function<T: Display + Clone, U: Clone + Debug>(t: T, u: U) -> i32 {
```

We can write this instead with a `where` clause:

```rust,ignore
fn some_function<T, U>(t: T, u: U) -> i32
    where T: Display + Clone,
          U: Clone + Debug
{
```

This is less cluttered and makes this function’s signature look more similar to
a function without lots of trait bounds, in that the function name, parameter
list, and return type are close together.

### Fixing the `largest` Function with Trait Bounds

So any time you want to use behavior defined by a trait on a generic, you need
to specify that trait in the generic type parameter’s type bounds. We can now
fix the definition of the `largest` function that uses a generic type parameter
from Listing 10-5! When we set that code aside, we were getting this error:

```text
error[E0369]: binary operation `>` cannot be applied to type `T`
  |
5 |         if item > largest {
  |            ^^^^
  |
note: an implementation of `std::cmp::PartialOrd` might be missing for `T`
```

In the body of `largest` we wanted to be able to compare two values of type `T`
using the greater-than operator. That operator is defined as a default method
on the standard library trait `std::cmp::PartialOrd`. So in order to be able to
use the greater-than operator, we need to specify `PartialOrd` in the trait
bounds for `T` so that the `largest` function will work on slices of any type
that can be compared. We don’t need to bring `PartialOrd` into scope because
it’s in the prelude.

```rust,ignore
fn largest<T: PartialOrd>(list: &[T]) -> T {
```

If we try to compile this, we’ll get different errors:

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

The key to this error is `cannot move out of type [T], a non-copy array`.
With our non-generic versions of the `largest` function, we were only trying to
find the largest `i32` or `char`. As we discussed in Chapter 4, types like
`i32` and `char` that have a known size can be stored on the stack, so they
implement the `Copy` trait. When we changed the `largest` function to be
generic, it’s now possible that the `list` parameter could have types in it
that don’t implement the `Copy` trait, which means we wouldn’t be able to move
the value out of `list[0]` and into the `largest` variable.

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
