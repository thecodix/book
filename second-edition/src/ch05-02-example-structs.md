## Un Programa de Ejemplo Utilizando Instrucciones

Para entender cuándo podríamos querer usar las estructuras, escribamos un programa que 
calcule el área de un rectángulo. Empezaremos con variables individuales y luego 
refactorizaremos el programa hasta que utilicemos estructuras.

Hagamos un nuevo proyecto binario con Cargo llamado *rectangles* que tomará 
el ancho y la altura de un rectángulo especificado en píxeles y calculará el 
área del rectángulo. El Listado 5-8 muestra un programa corto con una forma de hacer
eso en nuestro proyecto *src/main.rs*:

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

<span class="caption">Listado 5-8: Cálculo del área de un rectángulo
especificado por su anchura y altura en variables separadas.</span>

Ahora, ejecuta este programa usando `cargo run`:

```text
The area of the rectangle is 1500 square pixels.
```

### Refactorizando con Tuplas

Aunque Listing 5-8 funciona y calcula el área del rectángulo 
llamando a la función `area` con cada dimensión, podemos hacerlo mejor. El ancho
y alto se relacionan entre sí porque juntas describen un rectángulo.

El problema con este código es evidente en la firma de `area`:

```rust,ignore
fn area(width: u32, height: u32) -> u32 {
```

La función `area` se supone que calcula el área de un rectángulo, pero la
función que escribimos tiene dos parámetros. Los parámetros están relacionados, pero eso 
no se expresa en ninguna parte de nuestro programa. Sería más legible y más
manejable agrupar el ancho y la altura juntos. Ya hemos discutido una forma de 
hacerlo en la sección "Agrupando valores en Tuplas" del Capítulo 3: 
usando tuplas. El Listado 5-9 muestra otra versión de nuestro programa que usa tuplas:

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

<span class="caption">Listado 5-9: Especificación del ancho y altura del
 rectángulo con una tupla</span>

De alguna manera, este programa es mejor. Las tuplas añaden un poco de estructura, y
ahora estamos pasando sólo un argumento. Pero de otra forma esta versión es menos
clara: las tuplas no nombran sus elementos, así que nuestro cálculo se ha vuelto más
confuso porque tenemos que indexar en las partes de la tupla.

No importa si mezclamos ancho y alto para el cálculo del área, pero 
si queremos dibujar el rectángulo en la pantalla, ¡esto importaría! Deberíamos tener
en cuenta que `width` es el índice de tupla `0` y `height` es el índice de
tupla `1`. Si alguien más trabajó en este código, tendrían que averiguar esto
y tenerlo en cuenta. Sería fácil olvidar o confundir estos valores y 
causar errores, porque no hemos transmitido el significado de nuestros datos
en nuestro código.

### Refactoring with Structs: Adding More Meaning

We use structs to add meaning by labeling the data. We can transform the tuple
we’re using into a data type with a name for the whole as well as names for the
parts, as shown in Listing 5-10:

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

<span class="caption">Listing 5-10: Defining a `Rectangle` struct</span>

Here we’ve defined a struct and named it `Rectangle`. Inside the `{}` we
defined the fields as `width` and `height`, both of which have type `u32`. Then
in `main` we create a particular instance of a `Rectangle` that has a width of
30 and a height of 50.

Our `area` function is now defined with one parameter, which we’ve named
`rectangle`, whose type is an immutable borrow of a struct `Rectangle`
instance. As mentioned in Chapter 4, we want to borrow the struct rather than
take ownership of it. This way, `main` retains its ownership and can continue
using `rect1`, which is the reason we use the `&` in the function signature and
where we call the function.

The `area` function accesses the `width` and `height` fields of the `Rectangle`
instance. Our function signature for `area` now says exactly what we mean:
calculate the area of a `Rectangle`, using its `width` and `height` fields.
This conveys that the width and height are related to each other, and gives
descriptive names to the values rather than using the tuple index values of `0`
and `1`. This is a win for clarity.

### Adding Useful Functionality with Derived Traits

It’d be nice to be able to print out an instance of our `Rectangle` while we’re
debugging our program and see the values for all its fields. Listing 5-11 tries
the `println!` macro as we have used it in Chapters 2, 3, and 4:

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

<span class="caption">Listing 5-11: Attempting to print a `Rectangle`
instance</span>

When we run this code, we get an error with this core message:

```text
error[E0277]: the trait bound `Rectangle: std::fmt::Display` is not satisfied
```

The `println!` macro can do many kinds of formatting, and by default, `{}`
tells `println!` to use formatting known as `Display`: output intended for
direct end user consumption. The primitive types we’ve seen so far implement
`Display` by default, because there’s only one way you’d want to show a `1` or
any other primitive type to a user. But with structs, the way `println!` should
format the output is less clear because there are more display possibilities:
do you want commas or not? Do you want to print the curly brackets? Should all
the fields be shown? Due to this ambiguity, Rust doesn’t try to guess what we
want and structs don’t have a provided implementation of `Display`.

If we continue reading the errors, we’ll find this helpful note:

```text
`Rectangle` cannot be formatted with the default formatter; try using
`:?` instead if you are using a format string
```

Let’s try it! The `println!` macro call will now look like `println!("rect1 is
{:?}", rect1);`. Putting the specifier `:?` inside the `{}` tells `println!` we
want to use an output format called `Debug`. `Debug` is a trait that enables us
to print out our struct in a way that is useful for developers so we can see
its value while we’re debugging our code.

Run the code with this change. Drat! We still get an error:

```text
error[E0277]: the trait bound `Rectangle: std::fmt::Debug` is not satisfied
```

But again, the compiler gives us a helpful note:

```text
`Rectangle` cannot be formatted using `:?`; if it is defined in your
crate, add `#[derive(Debug)]` or manually implement it
```

Rust *does* include functionality to print out debugging information, but we
have to explicitly opt-in to make that functionality available for our struct.
To do that, we add the annotation `#[derive(Debug)]` just before the struct
definition, as shown in Listing 5-12:

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

<span class="caption">Listing 5-12: Adding the annotation to derive the `Debug`
trait and printing the `Rectangle` instance using debug formatting</span>

Now when we run the program, we won’t get any errors and we’ll see the
following output:

```text
rect1 is Rectangle { width: 30, height: 50 }
```

Nice! It’s not the prettiest output, but it shows the values of all the fields
for this instance, which would definitely help during debugging. When we have
larger structs, it’s useful to have output that’s a bit easier to read; in
those cases, we can use `{:#?}` instead of `{:?}` in the `println!` string.
When we use the `{:#?}` style in the example, the output will look like this:

```text
rect1 is Rectangle {
    width: 30,
    height: 50
}
```

Rust has provided a number of traits for us to use with the `derive` annotation
that can add useful behavior to our custom types. Those traits and their
behaviors are listed in Appendix C. We’ll cover how to implement these traits
with custom behavior as well as how to create your own traits in Chapter 10.

Our `area` function is very specific: it only computes the area of rectangles.
It would be helpful to tie this behavior more closely to our `Rectangle`
struct, because it won’t work with any other type. Let’s look at how we can
continue to refactor this code by turning the `area` function into an `area`
*method* defined on our `Rectangle` type.
