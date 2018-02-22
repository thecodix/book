## An Example Program Using Structs

To understand when we might want to use structs, let’s write a program that
calculates the area of a rectangle. We’ll start with single variables, and then
refactor the program until we’re using structs instead.

Let’s make a new binary project with Cargo called *rectangles* that will take
the width and height of a rectangle specified in pixels and will calculate the
area of the rectangle. Listing 5-8 shows a short program with one way of doing
just that in our project’s *src/main.rs*:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let width1 = 30;
    let height1 = 50;

    println!(
        "The area of the rectangle is {} square pixels.",
        area(width1, height1)
    );
}

fn area(width: u32, height: u32) -> u32 {
    width * height
}
```

<span class="caption">Listing 5-8: Calculating the area of a rectangle
specified by its width and height in separate variables</span>

Now, run this program using `cargo run`:

```text
The area of the rectangle is 1500 square pixels.
```

### Refactoring with Tuples

Even though Listing 5-8 works and figures out the area of the rectangle by
calling the `area` function with each dimension, we can do better. The width
and the height are related to each other because together they describe one
rectangle.

The issue with this code is evident in the signature of `area`:

```rust,ignore
fn area(width: u32, height: u32) -> u32 {
```

The `area` function is supposed to calculate the area of one rectangle, but the
function we wrote has two parameters. The parameters are related, but that’s
not expressed anywhere in our program. It would be more readable and more
manageable to group width and height together. We’ve already discussed one way
we might do that in the “Grouping Values into Tuples” section of Chapter 3: by
using tuples. Listing 5-9 shows another version of our program that uses tuples:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let rect1 = (30, 50);

    println!(
        "The area of the rectangle is {} square pixels.",
        area(rect1)
    );
}

fn area(dimensions: (u32, u32)) -> u32 {
    dimensions.0 * dimensions.1
}
```

<span class="caption">Listing 5-9: Specifying the width and height of the
rectangle with a tuple</span>

In one way, this program is better. Tuples let us add a bit of structure, and
we’re now passing just one argument. But in another way this version is less
clear: tuples don’t name their elements, so our calculation has become more
confusing because we have to index into the parts of the tuple.

It doesn’t matter if we mix up width and height for the area calculation, but
if we want to draw the rectangle on the screen, it would matter! We would have
to keep in mind that `width` is the tuple index `0` and `height` is the tuple
index `1`. If someone else worked on this code, they would have to figure this
out and keep it in mind as well. It would be easy to forget or mix up these
values and cause errors, because we haven’t conveyed the meaning of our data in
our code.

### Refactorización con Estructuras: Añadiendo más Significado

Usamos estructuras para añadir significado al etiquetar los datos. Podemos transformar la tupla 
que estamos usando en un tipo de datos con un nombre para el conjunto, así como los nombres de las 
partes, como se muestra en Listado 5-10:

<span class="filename">Filename: src/main.rs</span>

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };

    println!(
        "The area of the rectangle is {} square pixels.",
        area(&rect1)
    );
}

fn area(rectangle: &Rectangle) -> u32 {
    rectangle.width * rectangle.height
}
```

<span class="caption">Listado 5-10: Definiendo una estructura de `Rectángulo`.</span>

Aquí hemos definido una estructura y la hemos llamado `Rectangle`. Dentro del `{}` 
definimos los campos como `width` y `height`, los cuales tienen el tipo `u32`. Luego,
en `main` creamos una instancia particular de un `Rectangle` que tiene una anchura de 
30 y una altura de 50. 

Nuestra función `área` se define ahora con un parámetro, al que hemos llamado
`rectangle`, cuyo tipo es un préstamo inmutable de una instancia estructural 
`Rectangle`. Como se menciona en el capítulo 4, queremos tomar prestada la estructura en
lugar de apropiarnos de ella. De esta manera, `main` retiene su posesión y puede continuar
usando `rect1`, que es la razón por la que usamos el `&` en la función firma 
y donde llamamos la función.

La función `area` accede a los campos `width` y `height` de la instancia 
`Rectangle`. Nuestra firma de función para `área` ahora dice exactamente lo que queremos decir:
calcula el área de un `Rectangle`, usando sus campos `width` y `height`.
Esto indica que la anchura y la altura están relacionadas entre sí, y da 
nombres descriptivos a los valores en lugar de utilizar los valores del índice de tuplas `0` 
y `1`. Esto es una victoria para la claridad.

### Añadir Funcionalidad ütil con Rasgos Derivados

Sería bueno poder imprimir una instancia de nuestro `Rectangle` mientras estamos 
depurando nuestro programa y ver los valores para todos sus campos. El listado 5-11 prueba
la macro `println!` tal y como la hemos usado en los capítulos 2, 3 y 4:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };

    println!("rect1 is {}", rect1);
}
```

<span class="caption">Listado 5-11: Intentar imprimir una instancia de 
`Rectangle`</span>

Cuando ejecutamos este código, obtenemos un error con este mensaje central:

```text
error[E0277]: the trait bound `Rectangle: std::fmt::Display` is not satisfied
```

La macro `println!` puede hacer muchas clases de formateo, y por defecto, `{}`
le indica a `println!` que utilice el formato conocido como `Display`: salida destinado al
consumo directo del usuario final. Los tipos primitivos que hemos visto hasta ahora implementan 
`Display` por defecto, porque sólo hay una forma de mostrar un `1` o 
cualquier otro tipo primitivo a un usuario. Pero con structuras, la forma en que `println! ` debería
formatear la salida es menos clara porque hay más posibilidades de visualización: 
¿quieres o no quieres comas? ¿Quieres imprimir las llaves? ¿Deberían 
mostrarse todos los campos? Debido a esta ambigüedad, Rust no intenta adivinar lo que
queremos y las estructuras no tienen una implementación provista de `Display`.

Si continuamos leyendo los errores, encontraremos esta útil nota:

```text
`Rectangle` cannot be formatted with the default formatter; try using
`:?` instead if you are using a format string
```

Probémoslo! La llamada a la macro `println!` ahora parecerá `println!("rect1 es
{:?}", rect1);`. Poniendo el especificador `:?` dentro del `{}` le dice a `imprintln!` que 
queremos usar un formato de salida llamado `Debug`. `Debug` es un rasgo que nos permite
imprimir nuestra estructura de una manera que es útil para los desarrolladores para que podamos
ver su valor mientras depuramos nuestro código.

Ejecuta el código con este cambio. ¡Maldición! Todavía tenemos un error:

```text
error[E0277]: the trait bound `Rectangle: std::fmt::Debug` is not satisfied
```

Pero de nuevo, el compilador nos da una nota útil:

```text
`Rectangle` cannot be formatted using `:?`; if it is defined in your
crate, add `#[derive(Debug)]` or manually implement it
```

Rust *hace* que se incluya la funcionalidad para imprimir la información de depuración, pero
tenemos que optar explícitamente por hacer que esa funcionalidad esté disponible para nuestra estructura.
Para ello, añadimos la anotación `#[derive(Debug)]` justo antes de la definición de la 
estructura, como se muestra en el Listado 5-12:

<span class="filename">Filename: src/main.rs</span>

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };

    println!("rect1 is {:?}", rect1);
}
```

<span class="caption">Listado 5-12: Añadiendo la anotación para derivar el rasgo `Debug`
 e imprimir la instancia `Rectangle` usando el formato de depuración</span>

Ahora cuando ejecutamos el programa, no tendremos ningún error y veremos la
siguiente salida:

```text
rect1 is Rectangle { width: 30, height: 50 }
```

¡Bien! No es la salida más bonita, pero muestra los valores de todos los campos
para esta instancia, lo que definitivamente ayudaría durante la depuración. Cuando tenemos
estructuras más grandes, es útil tener una salida que sea un poco más fácil de leer; en
esos casos, podemos usar `{:#?}`en lugar de `{:?}`en la cadena `println!`.
Cuando usamos el estilo `{:#?}` en el ejemplo, la salida se verá así:

```text
rect1 is Rectangle {
    width: 30,
    height: 50
}
```

Rust nos ha proporcionado una serie de rasgos que podemos utilizar con la anotación `derive`
que puede añadir un comportamiento útil a nuestros tipos personalizados. Estos rasgos y sus
comportamientos se enumeran en el Apéndice C. En el Capítulo 10 explicaremos cómo 
implementar estos rasgos con un comportamiento personalizado, y también de cómo crear tus propios rasgos.

Nuestra función `area` es muy específica: sólo calcula el área de rectángulos.
Sería útil vincular este comportamiento más estrechamente a nuestra estructura del
`Rectangle`, porque no funcionará con ningún otro tipo. Veamos cómo podemos 
continuar refactorizando este código al convertir la función de `área` en un *método* 
de `área` definido en nuestro tipo de `Rectangle`.
