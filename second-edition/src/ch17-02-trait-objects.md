## Usando Objetos de Rasgos que Permiten Valores de Diferentes Tipos

En el Capítulo 8, mencionamos que una limitación de los vectores es que ellos
solo pueden almacenar elementos de un tipo. Creamos una solución en el Listado
8-10 donde definimos un enumerado `SpreadsheetCell` que tenía variantes para
sostener enteros, flotantes, y texto. Esto significó que podíamos almacenar
diferentes tipos de dato en cada celda y aún tener un vector que es representado
en columnas de celdas. Esta es una solución perfectamente buena cuando nuestros
objetos intercambiables son un conjunto de tipos fijados que conocemos cuando
nuestro código es compilado.

Sin embargo, algunas veces, queremos que el usuario de nuestra librería sea
capaz de extender los conjuntos de tipos que que son válidos en una situación
particular. Para mostrar cómo podemos lograr eso, crearemos un ejemplo de
herramienta de Graphical User Interface (Interface de Usuario Gráfica) que itera
a través de una lista de objetos, llamando al método `draw` en cada uno para
dibujarlo en la pantalla; una técnica común para herramientas GUI. Crearemos un
crate de librería conteniendo la estructura de una librería GUI llamada `rust_gui`.
Este crate podría incluir algunos tipos para que las personas los usen, como
`Button` o `TextField`. Sobre estos, los usuarios de `rust_gui` querrán crear
sus propios tipos que podrán ser dibujados en la pantalla: por ejemplo, un
programador podría añadir una `Imagen`, otro podría añadir un `SelectBox`.

No podremos implementar una librería GUI completada para este ejemplo, pero
mostraremos cómo las piezas encajan. Al mismo tiempo de escribir esta librería,
no podemos conocer ni definir todos los tipos que otros programadores querrán
crear. Lo que sí sabemos es que `rust_gui` necesita llevar rastro de muchos
valores que son de diferentes tipos, y necesita ser capaz de llamar un método
`draw` en cada uno de estos diferentes tipos de valores. No necesita saber
exactamente lo que pasará cuando llamamos el método `draw`, solo que el valor
tendrá el método disponibles para nosotros llamarlo.

Para hacer esto en un lenguaje con herencia, podríamos definir una clase llamada
`Component` que tiene un método llamado `draw` en ella. Las otras clases como
`Button`, `Image`. y `SelectBox` heredarían de `Component` y así heredarían 
el método `draw`. Cada uno podría reescribir el método `draw` para definir su
propio comportamiento, pero el framework trararía todos los tipos como si fueran
instancias de `Component` y llamaría `draw` en ellos. Perro Rust no tiene herencia,
así que necesitamos otra forma.

### Definiendo un Rasgo para Comportamiento Común

Para implementar el rasgo queremos que `rust_gui` tenga, definiremos un rasgo
llamado `Draw` que tendrá un método llamado `draw`. Luego podemos definir un vector
que tome un *objeto de rasgo*. Un objeto de rasgo apunta a una instancia de un
tipo que implementa el rasgo que especificamos. Podemos crear un objeto de rasgo
especificando algún tipo de apuntador, como una referencia `&` o un apuntador
inteligente `Boxt<T>`, y luego especificando el rasgo relevante (hablaremos
sobre la razón del porqué los objetos de rasgo tienen que usar un apuntador
en el Capítulo 19, en la sección de Tipos Dimensionados Dinámicamente). Podemos
usar objetos de rasgo en lugar de un tipo genérico o concreto. Donde sea que 
usamos un objeto de rasgo, el sistema de tipos de Rust se asegurará durante el
tiempo de compilación de que cualquier valor usado en ese contexto implementará
el rasgo del objeto de rasgo. De esta forma no necesitamos saber todos los tipos
posibles durante el tiempo de compilación.

<!-- ¿Qué hará el objeto de rasgo en este caso? Tomé esta última parte
de la línea abajo, pero no tengo 100% seguridad de esto -->
<!-- Moví un poco hacia arriba y lo reescribí, espero que eso aclare /Carol -->

Hemos mencionado que en Rust nos abstenemos de llamar "objetos" a los registros
y enumerados para distinguirlos de objetos de otros lenguajes. En un registro
o enumerado, los datos en los campos del registro y el comportamiento en los
bloques `impl` está separado, mientras que cuando en otros lenguajes los datos y
el comportamiento se mezclan se le es llamado un objeto. Los objetos de rasgo,
*son* mucho más como objetos de otros lenguajes, en el sentido deque ellos
combinan datos y comportamiento. Sin embargo, los objetos de rasgo difieren de
objetos tradicionales ya que no podemos añadir datos en un objeto de rasgo. Los
objetos de rasgo generalmente no son tan útiles como lo son los objetos en otros
lenguajes: su objetivo específico es permitir la abstracción a través del
comportamiento común.

El Listado 17-3 muestra cómo definir un rasgo llamado `Draw` con un método llamado
`draw`:

<span class="filename">Filename: src/lib.rs</span>

```rust
pub trait Draw {
    fn draw(&self);
}
```

<span class="caption">Listing 17-3: Definition of the `Draw` trait</span>

Esto debería de verse familiar desde nuestra discusión sobre cómo definir rasgos
en el Capítulo 10. A continuación viene algo nuevo: El Listado 17-4 define un
registro llamado `Screen` que contiene un vector llamado `compononents`. Este
vector es del tipo `Box<Draw>`, el cual es un objeto de rasgo: es un soporte
para cualquier tipo dentro de un `Box` que implemente el rasgo `Draw`

<!-- ¿Sería útil hacerle saber al lector el porqué necesitamos una caja aquí?
¿O ya eso sería evidente en este punto? -->
<!-- Hablamos de esto en el capítulo 19; añadí una referencia al comienzo de
esta sección donde hablamos acerca de necesitar un `&` o un `Box` para ser un
objeto de rasgo. /Carol -->

<span class="filename">Filename: src/lib.rs</span>

```rust
# pub trait Draw {
#     fn draw(&self);
# }
#
pub struct Screen {
    pub components: Vec<Box<Draw>>,
}
```

<span class="caption">Listing 17-4: Definition of the `Screen` struct with a
`components` field holding a vector of trait objects that implement the `Draw`
trait</span>

En el registro `Screen`, definiremos un método llamado `run` que llamará el
método `draw` en cada uno de sus `components`, como se muestra en el Listado
17-5:

<span class="filename">Filename: src/lib.rs</span>

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

<span class="caption">Listing 17-5: Implementing a `run` method on `Screen`
that calls the `draw` method on each component</span>

Esto funciona de forma diferente a definir un registro que usa un parámetro
de tipo genérico con límites de rasgo. Un parámetro de tipo genérico sólo puede
ser substituído con un tipo concreto al momento, mientras que los objetos de
rasgo permiten que varios tipos concretos substituyan al objeto de rasgo durante
el tiempo de ejecución. Por ejemplo, pudimos haber definido los registros `Screen`
usando un tipo genérico y un límite de rasgo como se muestra en el Listado 17-6:

<span class="filename">Filename: src/lib.rs</span>

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

<span class="caption">Listing 17-6: An alternate implementation of the `Screen`
struct and its `run` method using generics and trait bounds</span>

Esto nos restringe a una instancia de `Screen` que tiene una lista de componentes,
todos del tipo `Button` o todos del tipo `TextField`. Si usted tendrá únicamente
colecciones homogéneas, usando genéricos y límites de rasgo, es preferible ya que
las definiciones serán transformadas durante el tiempo de compilación para usar
los tipos concretos.

With the method using trait objects, on the other hand, one `Screen`
instance can hold a `Vec` that contains a `Box<Button>` as well as a
`Box<TextField>`. Let’s see how that works, and then talk about the runtime
performance implications.

### Implementing the Trait

Now we’ll add some types that implement the `Draw` trait. We’re going to
provide the `Button` type. Again, actually implementing a GUI library is out of
scope of this book, so the `draw` method won’t have any useful implementation
in its body. To imagine what the implementation might look like, a `Button`
struct might have fields for `width`, `height`, and `label`, as shown in
Listing 17-7:

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
