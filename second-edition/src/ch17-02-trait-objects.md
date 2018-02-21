## Usar objetos de rasgo que permiten valores de diferentes tipos

En el Capítulo 8, mencionamos que una limitación de los vectores es que solo pueden
almacenar elementos de un tipo. Creamos una solución alternativa en el Listado 8-10 donde
definimos una enumeración `SpreadsheetCell` que tenía variantes para contener enteros, flotantes,
y texto. Esto significaba que podíamos almacenar diferentes tipos de datos en cada celda y
 aún así tener un vector que representaba una fila de celdas. Absolutamente, esta es una correcta
solución para cuando nuestros elementos intercambiables son un conjunto fijo de tipos que conocemos
cuando nuestro código se compila.

A veces, sin embargo, queremos que el usuario de nuestra biblioteca pueda ampliar el
set de tipos que son válidos en una situación particular. Para mostrar cómo podríamos
lograr esto, crearemos un ejemplo de herramienta de interfaz gráfica de usuario que
itera a través de una lista de elementos, convocando una función 'draw' en cada uno para dibujarlo
en la pantalla; una técnica común para herramientas GUI. Vamos a crear una
cajón de biblioteca que contiene la estructura de una biblioteca de GUI llamada `rust_gui`. Este
cajón puede incluir algunos tipos para que las personas lo usen, como `Button` o
`TextField`. Además de esto, los usuarios de `rust_gui` querrán crear sus propios
tipos que se pueden dibujar en la pantalla: por ejemplo, un programador podría agregar
una `Image`, y otro agregaría una `SelectBox`.

No implementaremos una biblioteca de GUI completamente desarrollada para este ejemplo, pero mostraremos
cómo las piezas encajarían juntas. En el momento de programar la biblioteca, no podemos
conocer y definir todos los tipos que otros programadores querrán crear. Lo que conocemos
es que `rust_gui` necesita hacer un seguimiento de un montón de valores que son de
diferentes tipos, y necesita poder convocar a una función `draw` en cada uno de 
estos valores de tipo diferente. Por lo que no necesita saber exactamente lo que
sucederá cuando convocamos a una función `draw`, solo que el valor tendrá ese
método disponible para nosotros al convocar la función.

Para hacer esto en un lenguaje con herencia, podríamos definir una clase llamada
`Component` que tiene un método llamado `draw` en él. Las otras clases como
`Button`, `Image`, y `SelectBox` se heredarían de `Component` y por lo tanto
heredaría la función `draw`. Podrían anular cada uno el método `draw` para definir
su comportamiento personalizado, pero el marco podría tratar todos los tipos como si
fueran casos de `Component`  y convocaciones a funciones `draw` en ellos. Pero Rust no tiene
herencia, entonces necesitamos otra manera.

### Definición de un rasgo para el comportamiento común

Para implementar el comportamiento que queremos que tenga `rust_gui`, definiremos un rasgo
llamado `Draw` que tendrá un método llamado `draw`. Entonces podemos definir un
vector que toma un *rasgo de objeto*. Un rasgo de objecto apunta a un caso de un
tipo que implementa el rasgo que especificamos. Creamos un objeto de rasgo 
especificando algún tipo de puntero, como una referencia `&` o un `Box <T>` puntero
inteligente, y luego especificando el rasgo relevante (hablaremos del por qué
los objetos de rasgo tienen que usar un puntero en el Capítulo 19 en la sección Dinámicamente
Tipos de tamaño). Podemos usar objetos de rasgo en lugar de un tipo genérico o concreto.
Donde sea que usemos un objeto de rasgo, el sistema de tipos de Rust se asegurará en tiempo de compilación
que cualquier valor utilizado en ese contexto implementará el rasgo del objeto de rasgo.
De esta forma, no necesitamos saber todos los tipos posibles en tiempo de compilación.

<!-- What will the trait object do in this case? I've taken this last part of
the line from below, but I'm not 100% on that -->
<!-- I've moved up more and reworded a bit, hope that clarifies /Carol -->

Hemos mencionado que en Rust nos abstenemos de convocar estructuras y enumeraciones de
"Objetos" para distinguirlos de los objetos de otros lenguajes. En una estructura o
enumeración, los datos en los campos de estructura y el comportamiento en bloques `impl` es
separado, mientras que en otros lenguajes los datos y el comportamiento se combinaron en un
concepto que a menudo se etiquetan como un objeto. Los objetos de rasgo, sin embargo, *son* más como
objetos en otros idiomas, en el sentido de que combinan datos y
comportamiento. Sin embargo, los objetos de rasgo difieren de los objetos tradicionales en que
no puede agregar datos a un objeto de rasgo. Los objetos de rasgo no son tan útiles en general como
objetos en otros lenguajes: su propósito específico es permitir la abstracción
a través del comportamiento común.

Listado 17-3 muestra cómo definir un rasgo llamado `Draw` con un método llamado
`draw`:

<span class="filename">Nombre de archivo: src/lib.rs</span>

```rust
pub trait Draw {
    fn draw(&self);
}
```

<span class="caption">Listado 17-3: Definición del rasgo `Draw`</span>

Esto debería resultar familiar a partir de nuestras discusiones sobre cómo definir rasgos en
Capítulo 10. Luego viene algo nuevo: el Listado 17-4 define una estructura llamada
`Screen` que contiene un vector llamado `components`. Este vector es de tipo
`Box<Draw>`,que es un objeto de rasgo: es un sustituto para cualquier tipo dentro de un
`Box` que implementa el rasgo `Draw`.

<!-- Would it be useful to let the reader know why we need a box here, or will
that be clear at this point? -->
<!-- We get into this in chapter 19; I've added a reference to the start of
this section where we talk about needing a `&` or a `Box` to be a trait object.
/Carol -->

<span class="filename">Nombre de archivo: src/lib.rs</span>

```rust
# pub trait Draw {
#     fn draw(&self);
# }
#
pub struct Screen {
    pub components: Vec<Box<Draw>>,
}
```

<span class="caption">Listado 17-4: Definición de la estructura `Screen` con un
campo de `componente` sosteniendo un vector de objetos de rasgo que implementan el rasgo
`Draw`</span>

En la estructura `Screen`, definiremos un método llamado` run` que convocará a la función
`draw` en cada uno de sus `component`, como se muestra en el Listado 17-5:

<span class="filename">Nombre de archivo: src/lib.rs</span>

```rust
# pub trait Draw {
#     fn draw(&self);
# }
#
# pub struct Screen {
#     pub components: Vec<Box<Draw>>,
# }
#
impl Screen {
    pub fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
}
```

<span class="caption">Listado 17-5: Implementando un método `run` en` Screen`
que llama al método `draw` en cada componente</span>

Esto funciona de manera diferente a la definición de una estructura que usa un parámetro de tipo genérico
con límites de rasgos. Un parámetro de tipo genérico solo puede ser sustituido por un
tipo concreto a la vez, mientras que los objetos rasgo permiten múltiples tipos concretos
para completar el objeto de rasgo en tiempo de ejecución. Por ejemplo, podríamos haber definido
la estructura `Screen` usando un tipo genérico y un rasgo acotado como en el Listado 17-6:

<span class="filename">Nombre de archivo: src/lib.rs</span>

```rust
# pub trait Draw {
#     fn draw(&self);
# }
#
pub struct Screen<T: Draw> {
    pub components: Vec<T>,
}

impl<T> Screen<T>
    where T: Draw {
    pub fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
}
```

<span class="caption">Listado 17-6: Una implementación alternativa de la estructura
'Screen' y su método `run` utilizando genéricos y límites de rasgos </ span>

Esto nos restringe a un caso de 'Pantalla' que tiene una lista de componentes de todos
los tipos de `Button` o todo el tipo` TextField`. Si solo tienes colecciones
homogeneas, utilizando genéricos y rasgos acotados es preferible ya que 
las definiciones se monomorfizarán en el momento de la compilación para usar los tipos concretos.

Con el método que usa objetos rasgo, por otro lado, una `Pantalla`
instancia puede contener un `Vec` que contiene una `Box<Button>` así como un
`Box<TextField>`. Veamos cómo funciona eso, y luego hablamos sobre el tiempo de ejecución en
implicaciones de rendimiento.

### Implementando el RasgoPartial (1/2) spanish translation ch17-02-trait-objects.md

Ahora agregaremos algunos tipos que implementan el rasgo `Draw`. Iban a
proporcionar el tipo de `Button`. Nuevamente, la implementación de una biblioteca GUI está fuera de
alcance de este libro, por lo que el método `draw` no tendrá ninguna implementación útil
en su cuerpo. Para imaginar cómo se vería la implementación, una estructura 
`Button` podría tener campos para `width`, `weight` y `label`, como se muestra en el
Listado 17-7:

<span class="filename">Filename: src/lib.rs</span>

```rust
# pub trait Draw {
#     fn draw(&self);
# }
#
pub struct Button {
    pub width: u32,
    pub height: u32,
    pub label: String,
}

impl Draw for Button {
    fn draw(&self) {
        // Code to actually draw a button
    }
}
```

<span class="caption">Listing 17-7: A `Button` struct that implements the
`Draw` trait</span>

The `width`, `height`, and `label` fields on `Button` will differ from the
fields on other components, such as a `TextField` type that might have those
plus a `placeholder` field instead. Each of the types we want to draw on the
screen will implement the `Draw` trait, with different code in the `draw`
method to define how to draw that particular type, like `Button` has here
(without the actual GUI code that’s out of scope of this chapter). `Button`,
for instance, might have an additional `impl` block containing methods related
to what happens if the button is clicked. These kinds of methods won’t apply to
types like `TextField`.

Someone using our library has decided to implement a `SelectBox` struct that
has `width`, `height`, and `options` fields. They implement the `Draw` trait on
the `SelectBox` type as well, as shown in Listing 17-8:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
extern crate rust_gui;
use rust_gui::Draw;

struct SelectBox {
    width: u32,
    height: u32,
    options: Vec<String>,
}

impl Draw for SelectBox {
    fn draw(&self) {
        // Code to actually draw a select box
    }
}
```

<span class="caption">Listing 17-8: Another crate using `rust_gui` and
implementing the `Draw` trait on a `SelectBox` struct</span>

The user of our library can now write their `main` function to create a
`Screen` instance. To this they can add a `SelectBox` and a `Button` by putting
each in a `Box<T>` to become a trait object. They can then call the `run`
method on the `Screen` instance, which will call `draw` on each of the
components. Listing 17-9 shows this implementation:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
use rust_gui::{Screen, Button};

fn main() {
    let screen = Screen {
        components: vec![
            Box::new(SelectBox {
                width: 75,
                height: 10,
                options: vec![
                    String::from("Yes"),
                    String::from("Maybe"),
                    String::from("No")
                ],
            }),
            Box::new(Button {
                width: 50,
                height: 10,
                label: String::from("OK"),
            }),
        ],
    };

    screen.run();
}
```

<span class="caption">Listing 17-9: Using trait objects to store values of
different types that implement the same trait</span>

When we wrote the library, we didn’t know that someone would add the
`SelectBox` type someday, but our `Screen` implementation was able to operate
on the new type and draw it because `SelectBox` implements the `Draw` type,
which means it implements the `draw` method.

This concept---of being concerned only with the messages a value responds to,
rather than the value’s concrete type---is similar to a concept in dynamically
typed languages called *duck typing*: if it walks like a duck, and quacks like
a duck, then it must be a duck! In the implementation of `run` on `Screen` in
Listing 17-5, `run` doesn’t need to know what the concrete type of each
component is. It doesn’t check to see if a component is an instance of a
`Button` or a `SelectBox`, it just calls the `draw` method on the component. By
specifying `Box<Draw>` as the type of the values in the `components` vector,
we’ve defined `Screen` to need values that we can call the `draw` method on.

<!-- I may be slow on the uptake here, but it seems like we're saying that
responsibility for how the type trait object behaves with the draw method is
called on it belongs to the trait object, and not to the draw method itself. Is
that an accurate summary? I want to make sure I'm clearly following the
argument! -->
<!-- Each type (like `Button` or `SelectBox`) that implements the `Draw` trait
can customize what happens in the body of the `draw` method. The trait object
is just responsible for making sure that the only things that are usable in
that context are things that implement the `Draw` trait. Does this clear it up
at all? Is there something we should clarify in the text? /Carol -->

The advantage of using trait objects and Rust’s type system to do something
similar to duck typing is that we never have to check that a value implements a
particular method at runtime or worry about getting errors if a value doesn’t
implement a method but we call it anyway. Rust won’t compile our code if the
values don’t implement the traits that the trait objects need.

For example, Listing 17-10 shows what happens if we try to create a `Screen`
with a `String` as a component:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
extern crate rust_gui;
use rust_gui::Screen;

fn main() {
    let screen = Screen {
        components: vec![
            Box::new(String::from("Hi")),
        ],
    };

    screen.run();
}
```

<span class="caption">Listing 17-10: Attempting to use a type that doesn’t
implement the trait object’s trait</span>

We’ll get this error because `String` doesn’t implement the `rust_gui::Draw` trait:

```text
error[E0277]: the trait bound `std::string::String: rust_gui::Draw` is not satisfied
  -->
   |
 4 |             Box::new(String::from("Hi")),
   |             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^ the trait `rust_gui::Draw` is not
   implemented for `std::string::String`
   |
   = note: required for the cast to the object type `rust_gui::Draw`
```

This lets us know that either we’re passing something to `Screen` we didn’t
mean to pass, and we should pass a different type, or implement `Draw` on
`String` so that `Screen` is able to call `draw` on it.

### Trait Objects Perform Dynamic Dispatch

Recall from Chapter 10 our discussion on the monomorphization process performed
by the compiler when we use trait bounds on generics: the compiler generates
non-generic implementations of functions and methods for each concrete type
that we use in place of a generic type parameter. The code that results from
monomorphization is doing *static dispatch*. Static dispatch is when the
compiler knows what method you’re calling at compile time. This is opposed to
*dynamic dispatch*, when the compiler can’t tell at compile time which method
you’re calling. In these cases, the compiler emits code that will figure out at
runtime which method to call.

<!--I'm struggling to follow the static dispatch definition, can you expand
that a little? Which part of that is the static dispatch, pre-determining the
code called with a method and storing it? -->
<!-- Yes, in a way. We've expanded and moved the definitions of static and
dynamic dispatch together to better contrast, hopefully this helps? /Carol -->

When we use trait objects, Rust has to use dynamic dispatch. The compiler
doesn’t know all the types that might be used with the code using trait
objects, so it doesn’t know which method implemented on which type to call.
Instead, Rust uses the pointers inside of the trait object at runtime to know
which specific method to call. There’s a runtime cost when this lookup happens,
compared to static dispatch. Dynamic dispatch also prevents the compiler from
choosing to inline a method’s code which in turn prevents some optimizations.
We did get extra flexibility in the code that we wrote and were able to
support, though, so it’s a tradeoff to consider.

### Object Safety is Required for Trait Objects

<!-- Liz: we're conflicted on including this section. Not being able to use a
trait as a trait object because of object safety is something that
beginner/intermediate Rust developers run into sometimes, but explaining it
fully is long and complicated. Should we just cut this whole section? Leave it
(and finish the explanation of how to fix the error at the end)? Shorten it to
a quick caveat, that just says something like "Some traits can't be trait
objects. Clone is an example of one. You'll get errors that will let you know
if a trait can't be a trait object, look up object safety if you're interested
in the details"? Thanks! /Carol -->
<!-- That sounds like a good solution, since the compiler will warn them in any
case. I read through, editing a little, and I agree we could afford to cut it,
I'm not sure it brings practical skills to the user -->
<!-- Ok, I've cut section way down to the practical pieces, but still explained
a little bit /Carol -->

Only *object safe* traits can be made into trait objects. There are some
complex rules around all the properties that make a trait object safe, but in
practice, there are only two rules that are relevant. A trait is object safe if
all of the methods defined in the trait have the following properties:

- The return type isn’t `Self`
- There aren’t any generic type parameters

The `Self` keyword is an alias for the type we’re implementing traits or
methods on. Object safety is required for trait objects because once you have a
trait object, you no longer know what the concrete type implementing that trait
is. If a trait method returns the concrete `Self` type, but a trait object
forgets the exact type that it is, there’s no way that the method can use the
original concrete type that it’s forgotten. Same with generic type parameters
that are filled in with concrete type parameters when the trait is used: the
concrete types become part of the type that implements the trait. When the type
is erased by the use of a trait object, there’s no way to know what types to
fill in the generic type parameters with.

An example of a trait whose methods are not object safe is the standard
library’s `Clone` trait. The signature for the `clone` method in the `Clone`
trait looks like this:

```rust
pub trait Clone {
    fn clone(&self) -> Self;
}
```

`String` implements the `Clone` trait, and when we call the `clone` method on
an instance of `String` we get back an instance of `String`. Similarly, if we
call `clone` on an instance of `Vec`, we get back an instance of `Vec`. The
signature of `clone` needs to know what type will stand in for `Self`, since
that’s the return type.

The compiler will tell you if you’re trying to do something that violates the
rules of object safety in regards to trait objects. For example, if we had
tried to implement the `Screen` struct in Listing 17-4 to hold types that
implement the `Clone` trait instead of the `Draw` trait, like this:

```rust,ignore
pub struct Screen {
    pub components: Vec<Box<Clone>>,
}
```

We’ll get this error:

```text
error[E0038]: the trait `std::clone::Clone` cannot be made into an object
 -->
  |
2 |     pub components: Vec<Box<Clone>>,
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ the trait `std::clone::Clone` cannot be
  made into an object
  |
  = note: the trait cannot require that `Self : Sized`
```

This means you can’t use this trait as a trait object in this way. If you’re
interested in more details on object safety, see [Rust RFC 255].

[Rust RFC 255]: https://github.com/rust-lang/rfcs/blob/master/text/0255-object-safety.md
