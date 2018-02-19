## Controlling Visibility with `pub`

We resolved the error messages shown in Listing 7-5 by moving the `network` and
`network::server` code into the *src/network/mod.rs* and
*src/network/server.rs* files, respectively. At that point, `cargo build` was
able to build our project, but we still get warning messages about the
`client::connect`, `network::connect`, and `network::server::connect` functions
not being used:

```text
warning: function is never used: `connect`
 --> src/client.rs:1:1
  |
1 | / fn connect() {
2 | | }
  | |_^
  |
  = note: #[warn(dead_code)] on by default

warning: function is never used: `connect`
 --> src/network/mod.rs:1:1
  |
1 | / fn connect() {
2 | | }
  | |_^

warning: function is never used: `connect`
 --> src/network/server.rs:1:1
  |
1 | / fn connect() {
2 | | }
  | |_^
```

So why are we receiving these warnings? After all, we’re building a library
with functions that are intended to be used by our *users*, not necessarily by
us within our own project, so it shouldn’t matter that these `connect`
functions go unused. The point of creating them is that they will be used by
another project, not our own.

To understand why this program invokes these warnings, let’s try using the
`connect` library from another project, calling it externally. To do that,
we’ll create a binary crate in the same directory as our library crate by
making a *src/main.rs* file containing this code:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
extern crate communicator;

fn main() {
    communicator::client::connect();
}
```

We use the `extern crate` command to bring the `communicator` library crate
into scope. Our package now contains *two* crates. Cargo treats *src/main.rs*
as the root file of a binary crate, which is separate from the existing library
crate whose root file is *src/lib.rs*. This pattern is quite common for
executable projects: most functionality is in a library crate, and the binary
crate uses that library crate. As a result, other programs can also use the
library crate, and it’s a nice separation of concerns.

From the point of view of a crate outside the `communicator` library looking
in, all the modules we’ve been creating are within a module that has the same
name as the crate, `communicator`. We call the top-level module of a crate the
*root module*.

Also note that even if we’re using an external crate within a submodule of our
project, the `extern crate` should go in our root module (so in *src/main.rs*
or *src/lib.rs*). Then, in our submodules, we can refer to items from external
crates as if the items are top-level modules.

Right now, our binary crate just calls our library’s `connect` function from
the `client` module. However, invoking `cargo build` will now give us an error
after the warnings:

```text
error[E0603]: module `client` is private
 --> src/main.rs:4:5
  |
4 |     communicator::client::connect();
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
```

Ah ha! This error tells us that the `client` module is private, which is the
crux of the warnings. It’s also the first time we’ve run into the concepts of
*public* and *private* in the context of Rust. The default state of all code in
Rust is private: no one else is allowed to use the code. If you don’t use a
private function within your program, because your program is the only code
allowed to use that function, Rust will warn you that the function has gone
unused.

After we specify that a function like `client::connect` is public, not only
will our call to that function from our binary crate be allowed, but the
warning that the function is unused will go away. Marking a function as public
lets Rust know that the function will be used by code outside of our program.
Rust considers the theoretical external usage that’s now possible as the
function “being used.” Thus, when a function is marked public, Rust will not
require that it be used in our program and will stop warning that the function
is unused.

### Making a Function Public

To tell Rust to make a function public, we add the `pub` keyword to the start
of the declaration. We’ll focus on fixing the warning that indicates
`client::connect` has gone unused for now, as well as the `` module `client` is
private `` error from our binary crate. Modify *src/lib.rs* to make the
`client` module public, like so:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
pub mod client;

mod network;
```

La palabra clave `pub` se coloca justo antes de `mod`. Intentemos construir de nuevo:

```text
error[E0603]: function `connect` is private
 --> src/main.rs:4:5
  |
4 |     communicator::client::connect();
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
```

¡Hurra! Tenemos un error diferente! Sí, diferentes mensajes de error son motivo
de celebración. El nuevo error muestra que ``función `connect` is private``, así 
que editemos *src/client.rs* para hacer que `client::connect` también sea público:

<span class="filename">Filename: src/client.rs</span>

```rust
pub fn connect() {
}
```

Ahora vuelve a ejecutar `cargo build`:

```text
warning: function is never used: `connect`
 --> src/network/mod.rs:1:1
  |
1 | / fn connect() {
2 | | }
  | |_^
  |
  = note: #[warn(dead_code)] on by default

warning: function is never used: `connect`
 --> src/network/server.rs:1:1
  |
1 | / fn connect() {
2 | | }
  | |_^
```

El código compilado, y la advertencia de `client::connect` que no se está usando
ha desaparecido!

Las advertencias de código no utilizadas no siempre indican que un elemento de tu código debe
hacerse público: si tu *no* deseas que estas funciones formen parte de tu API
pública, las advertencias de código no utilizadas podrían estar alertándote sobre el código que ya no necesitas
para poder eliminarlo de forma segura. También podrían estar alertándote de un error si acabas
de eliminar accidentalmente todos los lugares de tu biblioteca donde se llama esta función.

Pero en este caso, *queremos* que las otras dos funciones formen parte de la
API pública de nuestro cajón, así que marquémoslas como `pub` y deshagámonos de las 
advertencias restantes. Modifica *src/network/mod.rs* para que luzca de la siguiente forma:

<span class="filename">Nombre del archivo: src/network/mod.rs</span>

```rust,ignore
pub fn connect() {
}

mod server;
```

Luego compila el código:

```text
warning: function is never used: `connect`
 --> src/network/mod.rs:1:1
  |
1 | / pub fn connect() {
2 | | }
  | |_^
  |
  = note: #[warn(dead_code)] on by default

warning: function is never used: `connect`
 --> src/network/server.rs:1:1
  |
1 | / fn connect() {
2 | | }
  | |_^
```

Hmmm, todavía estamos recibiendo una advertencia de función no utilizada, aunque
`network::connect` está configurada en `pub`. La razón es que la función es pública
dentro del módulo, pero el módulo de `network` en el que reside la función no es
pública. Estamos trabajando desde el interior de la biblioteca esta vez, mientras que
con `client::connect` trabajamos desde el exterior. Necesitamos cambiar
*src/lib.rs* para hacer también a `network` pública, así:

<span class="filename">Nombre del archivo: src/lib.rs</span>

```rust,ignore
pub mod client;

pub mod network;
```

Ahora cuando compilamos, esa advertencia desaparece:

```text
warning: function is never used: `connect`
 --> src/network/server.rs:1:1
  |
1 | / fn connect() {
2 | | }
  | |_^
  |
  = note: #[warn(dead_code)] on by default
```

Sólo queda una advertencia! ¡Intenta arreglar esto por tu cuenta!

### Reglas de Privacidad

En general, estas son las reglas para la visibilidad de los artículos:

1. Si un elemento es público, se puede acceder a él a través de cualquiera de sus módulos padre.
2. Si un elemento es privado, sólo puede ser accedido por su módulo padre inmediato
y por cualquiera de los módulos hijo del padre.

### Ejemplos de Privacidad

Veamos algunos ejemplos más de privacidad para tener un poco de práctica. Crea un nuevo
proyecto de biblioteca e introduce el código en Listado 7-6 en el archivo *src/lib.rs*
de tu nuevo proyecto:

<span class="filename">Nombre del archivo: src/lib.rs</span>

```rust,ignore
mod outermost {
    pub fn middle_function() {}

    fn middle_secret_function() {}

    mod inside {
        pub fn inner_function() {}

        fn secret_function() {}
    }
}

fn try_me() {
    outermost::middle_function();
    outermost::middle_secret_function();
    outermost::inside::inner_function();
    outermost::inside::secret_function();
}
```

<span class="caption">Listado 7-6: Ejemplos de funciones privadas y públicas,
algunas de las cuales son incorrectas</span>

Antes de intentar compilar este código, adivina qué líneas de la función
`try_me` tendrán errores. Luego, intenta compilar el código para ver si
tenías razón, y siga leyendo para la discusión de los errores!

#### Mirando los Errores

La función `try_me` está en el módulo raíz de nuestro proyecto. El módulo llamado
`outermost` es privado, pero la segunda regla de privacidad establece que la función
`try_me` permite el acceso al módulo `outermost` porque `outermost` está en
el módulo (raíz) actual, como es `try_me`.

La llamada a `outermost::middle_function` funcionará porque `middle_function` es 
pública, y `try_me` está accediendo a `middle_function` a través de su módulo
padre `outermost`. Hemos determinado en el párrafo anterior que este módulo
es accesible.

La llamada a la función `outermost::middle_secret_function` causará un error de compilación.
`middle_secret_function` es privada, por lo que se aplica la segunda regla. El módulo 
raíz no es ni el módulo actual de la función `middle_secret_function` (es `outermost`),
ni es un módulo hijo del módulo actual de la función `middle_secret_function`.

El módulo llamado `inside` es privado y no tiene módulos hijo, por lo que sólo
se puede acceder al módulo actual `outermost`. Esto significa que la función `try_me`
no puede llamar a la función `outermost::inside::inner_function` o 
`outermost::inside::secret_function`.

#### Corrección de los Errores

Aquí están algunas sugerencias para cambiar el código en un intento de corregir los
errores. Antes de probar cada una de ellas, adivina si corregirá los 
errores y luego compila el código para ver si tienes razón o no, usando las
reglas de privacidad para entender por qué.

* ¿Y si el módulo `inside` fuera público?
* ¿Qué pasaría si `outermost` fuera público e `inside` fuera privado?
* ¿Qué pasa si, en el cuerpo de `inner_function`, llamaste a
  `::outermost::middle_secret_function()`? (Los dos puntos al principio significan
  que queremos referirnos a los módulos empezando por el módulo raíz.)

Siéntete libre de diseñar más experimentos y probarlos!

A continuación, hablemos de poner los artículos en scope con la palabra clave `use`.
