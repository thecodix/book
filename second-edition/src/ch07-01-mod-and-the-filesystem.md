## `mod` and the Filesystem

We’ll start our module example by making a new project with Cargo, but instead
of creating a binary crate, we’ll make a library crate: a project that other
people can pull into their projects as a dependency. For example, the `rand`
crate discussed in Chapter 2 is a library crate that we used as a dependency in
the guessing game project.

We’ll create a skeleton of a library that provides some general networking
functionality; we’ll concentrate on the organization of the modules and
functions but we won’t worry about what code goes in the function bodies. We’ll
call our library `communicator`. By default, Cargo will create a library unless
another type of project is specified: if we omit the `--bin` option that we’ve
been using in all of the chapters preceding this one, our project will be a
library:

```text
$ cargo new communicator
$ cd communicator
```

Notice that Cargo generated *src/lib.rs* instead of *src/main.rs*. Inside
*src/lib.rs* we’ll find the following:

<span class="filename">Filename: src/lib.rs</span>

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}
```

Cargo creates an example test to help us get our library started, rather than
the “Hello, world!” binary that we get when we use the `--bin` option. We’ll
look at the `#[]` and `mod tests` syntax in the “Using `super` to Access a
Parent Module” section later in this chapter, but for now, leave this code at
the bottom of *src/lib.rs*.

Because we don’t have a *src/main.rs* file, there’s nothing for Cargo to
execute with the `cargo run` command. Therefore, we’ll use the `cargo build`
command to compile our library crate’s code.

We’ll look at different options for organizing your library’s code that will be
suitable in a variety of situations, depending on the intent of the code.

### Module Definitions

For our `communicator` networking library, we’ll first define a module named
`network` that contains the definition of a function called `connect`. Every
module definition in Rust starts with the `mod` keyword. Add this code to the
beginning of the *src/lib.rs* file, above the test code:

<span class="filename">Filename: src/lib.rs</span>

```rust
mod network {
    fn connect() {
    }
}
```

After the `mod` keyword, we put the name of the module, `network`, and then a
block of code in curly brackets. Everything inside this block is inside the
namespace `network`. In this case, we have a single function, `connect`. If we
wanted to call this function from code outside the `network` module, we
would need to specify the module and use the namespace syntax `::`, like so:
`network::connect()` rather than just `connect()`.

We can also have multiple modules, side by side, in the same *src/lib.rs* file.
For example, to also have a `client` module that has a function named `connect`
as well, we can add it as shown in Listing 7-1:

<span class="filename">Filename: src/lib.rs</span>

```rust
mod network {
    fn connect() {
    }
}

mod client {
    fn connect() {
    }
}
```

<span class="caption">Listing 7-1: The `network` module and the `client` module
defined side by side in *src/lib.rs*</span>

Now we have a `network::connect` function and a `client::connect` function.
These can have completely different functionality, and the function names do
not conflict with each other because they’re in different modules.

In this case, because we’re building a library, the file that serves as the
entry point for building our library is *src/lib.rs*. However, in respect to
creating modules, there’s nothing special about *src/lib.rs*. We could also
create modules in *src/main.rs* for a binary crate in the same way as we’re
creating modules in *src/lib.rs* for the library crate. In fact, we can put
modules inside of modules, which can be useful as your modules grow to keep
related functionality organized together and separate functionality apart. The
choice of how you organize your code depends on how you think about the
relationship between the parts of your code. For instance, the `client` code
and its `connect` function might make more sense to users of our library if
they were inside the `network` namespace instead, as in Listing 7-2:

<span class="filename">Filename: src/lib.rs</span>

```rust
mod network {
    fn connect() {
    }

    mod client {
        fn connect() {
        }
    }
}
```

<span class="caption">Listing 7-2: Moving the `client` module inside the
`network` module</span>

In your *src/lib.rs* file, replace the existing `mod network` and `mod client`
definitions with the ones in Listing 7-2, which have the `client` module as an
inner module of `network`. Now we have the functions `network::connect` and
`network::client::connect`: again, the two functions named `connect` don’t
conflict with each other because they’re in different namespaces.

In this way, modules form a hierarchy. The contents of *src/lib.rs* are at the
topmost level, and the submodules are at lower levels. Here’s what the
organization of our example in Listing 7-1 looks like when thought of as a
hierarchy:

```text
communicator
 ├── network
 └── client
```

And here’s the hierarchy corresponding to the example in Listing 7-2:

```text
communicator
 └── network
     └── client
```

The hierarchy shows that in Listing 7-2, `client` is a child of the `network`
module rather than a sibling. More complicated projects can have many modules,
and they’ll need to be organized logically in order to keep track of them. What
“logically” means in your project is up to you and depends on how you and your
library’s users think about your project’s domain. Use the techniques shown
here to create side-by-side modules and nested modules in whatever structure
you would like.

### Moviendo Módulos a Otros Archivos

Los módulos forman una estructura jerárquica, muy parecida a otra estructura de computación
a la que estás acostumbrado: ¡los sistemas de archivos! Podemos usar el sistema de módulos de Rust junto con
múltiples archivos para dividir los proyectos de Rust para que no todos vivan en
*src/lib.rs* o *src/main.rs*. Para este ejemplo, comencemos con el código en
Listado 7-3:

<span class="filename">Filename: src/lib.rs</span>

```rust
mod client {
    fn connect() {
    }
}

mod network {
    fn connect() {
    }

    mod server {
        fn connect() {
        }
    }
}
```

<span class="caption">Listado 7-3: Tres módulos,`client`,`network` y 
`network::server`, todos definidos en *src/lib.rs*.</span>

El fichero *src/lib.rs* tiene esta jerarquía de módulos:

```text
communicator
 ├── client
 └── network
     └── server
```

Si estos módulos tuvieran muchas funciones, y esas funciones se estuvieran haciendo largas,
sería difícil desplazarse por este archivo para encontrar el código con el que 
queríamos trabajar. Debido a que las funciones están anidadas dentro de uno o más bloques `mod`,
las líneas de código dentro de las funciones también empezarán a alargarse.
Estas serían buenas razones para separar los módulos `client`,`network` y `server`
de *src/lib.rs* y colocarlos en sus propios archivos.

En primer lugar, reemplaza el código de módulo `client` por sólo la declaración del 
módulo `client`, de modo que tu *src/lib.rs* se parezca al código mostrado en el Listado 7-4:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
mod client;

mod network {
    fn connect() {
    }

    mod server {
        fn connect() {
        }
    }
}
```

<span class="caption">Listado 7-4: Extracción del contenido del módulo `client` pero dejando la declaración en *src/lib.rs*</span>

Todavía estamos *declarando* el módulo `client` aquí, pero reemplazando el bloque
por un punto y coma, le estamos diciendo a Rust que busque en otra ubicación el código 
definido dentro del alcance del módulo `client`. En otras palabras, la línea `mod
cliente`; significa:

```rust,ignore
mod client {
    // contents of client.rs
}
```

Ahora tenemos que crear el archivo externo con ese nombre de módulo. Crea un
archivo *client.rs* en tu directorio *src/* y ábrelo. A continuación, introduce
lo siguiente, que es la función `connect` en el módulo `client` que
eliminamos en el paso anterior:

<span class="filename">Filename: src/client.rs</span>

```rust
fn connect() {
}
```

Ten en cuenta que no necesitamos una declaración `mod` en este archivo porque ya hemos
declarado el módulo `client` con `mod` en *src/lib.rs*. Este archivo sólo
proporciona los *contenidos* del módulo `client`. Si pusiéramos un `mod client` aquí, 
estaríamos dando al módulo de `client` su propio submódulo llamado `client`!

Rust por defecto sólo sabe mirar en *src/lib.rs*. Si queremos añadir más
archivos a nuestro proyecto, necesitamos decirle a Rust en *src/lib.rs* que busque en otros 
archivos; por eso es necesario definir `mod client` en *src/lib.rs* y no se puede
definir en *src/client.rs*.

Ahora el proyecto debería compilar con éxito, aunque obtendrás algunas
advertencias. Recuerde usar `cargo build` en lugar de `cargo run` porque tenemos
un cajón de biblioteca en lugar de un cajon binario:

```text
$ cargo build
   Compiling communicator v0.1.0 (file:///projects/communicator)
warning: function is never used: `connect`
 --> src/client.rs:1:1
  |
1 | / fn connect() {
2 | | }
  | |_^
  |
  = note: #[warn(dead_code)] on by default

warning: function is never used: `connect`
 --> src/lib.rs:4:5
  |
4 | /     fn connect() {
5 | |     }
  | |_____^

warning: function is never used: `connect`
 --> src/lib.rs:8:9
  |
8 | /         fn connect() {
9 | |         }
  | |_________^
```

Estas advertencias nos dicen que tenemos funciones que nunca se utilizan. No te preocupes
por estas advertencias por ahora; las trataremos más adelante en este capítulo en la
sección "Controlar la visibilidad con `pub`". La buena noticia es que son sólo 
advertencias, ¡nuestro proyecto se construyó con éxito!

A continuación, extraigamos el módulo `network` en su propio archivo usando el mismo
patrón. En *src/lib.rs*, borra el cuerpo del módulo `network` y añade un
punto y coma a la declaración, así:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
mod client;

mod network;
```

A continuación, crea un nuevo archivo *src/network.rs* e introduce lo siguiente:

<span class="filename">Filename: src/network.rs</span>

```rust
fn connect() {
}

mod server {
    fn connect() {
    }
}
```

Nota que todavía tenemos una declaración `mod` dentro de este archivo de módulo; esto es
porque todavía queremos que el `server` sea un submódulo de `network`.

Ejecuta `cargo build` de nuevo. Éxito! Tenemos un módulo más para extraer: `server`.
Porque es un submódulo, es decir,  un módulo dentro de un módulo, nuestra táctica
actual de extraer un módulo en un archivo nombrado después de ese módulo no funcionará. Lo
intentaremos de todos modos para que puedas ver el error. Primero, cambia *src/network.rs* para tener 
`mod server;` en lugar del contenido del módulo `server`:

<span class="filename">Filename: src/network.rs</span>

```rust,ignore
fn connect() {
}

mod server;
```

Luego crea un archivo *src/server.rs* e introduce el contenido del módulo `server` 
que hemos extraído:

<span class="filename">Filename: src/server.rs</span>

```rust
fn connect() {
}
```

Cuando intentemos `cargo build`, tendremos el error mostrado en Listing 7-5:

```text
$ cargo build
   Compiling communicator v0.1.0 (file:///projects/communicator)
error: cannot declare a new module at this location
 --> src/network.rs:4:5
  |
4 | mod server;
  |     ^^^^^^
  |
note: maybe move this module `src/network.rs` to its own directory via `src/network/mod.rs`
 --> src/network.rs:4:5
  |
4 | mod server;
  |     ^^^^^^
note: ... or maybe `use` the module `server` instead of possibly redeclaring it
 --> src/network.rs:4:5
  |
4 | mod server;
  |     ^^^^^^
```

<span class="caption">Listado 7-5: Error al intentar extraer el submódulo `server`
en *src/server.rs</span>

El error dice que `cannot declare a new module at this location` y está
apuntando a `mod server; `linea en *src/network.rs*. Así que *src/network.rs* es
diferente a *src/lib.rs* de alguna manera: sigue leyendo para entender por qué.

La nota en medio del Listado 7-5 es realmente muy útil porque 
señala algo por hacer de lo que que aún no hemos hablado:

```text
note: maybe move this module `network` to its own directory via
`network/mod.rs`
```

En lugar de seguir el mismo patrón de nombres de archivo que usamos 
anteriormente, podemos hacer lo que la nota sugiere:

1. Haz un nuevo *directory* llamado *network*, el nombre del módulo padre.
2. Mueve el archivo *src/network.rs* en el nuevo directorio *network* y 
   renómbralo a *src/network/mod.rs*.
3. Mueve el archivo del submódulo *src/server.rs* en el directorio *network*.

Aquí están los comandos para llevar a cabo estos pasos:

```text
$ mkdir src/network
$ mv src/network.rs src/network/mod.rs
$ mv src/server.rs src/network
```

Ahora, cuando intentemos ejecutar `cargo build`, la compilación funcionará (aunque seguiremos 
teniendo advertencias). Nuestro diseño de módulo todavía se ve así, que es exactamente
igual que cuando teníamos todo el código en *src/lib.rs* en Listado 7-3:

```text
communicator
 ├── client
 └── network
     └── server
```

El diseño de archivo correspondiente ahora se ve así:

```text
├── src
│   ├── client.rs
│   ├── lib.rs
│   └── network
│       ├── mod.rs
│       └── server.rs
```

So when we wanted to extract the `network::server` module, why did we have to
also change the *src/network.rs* file to the *src/network/mod.rs* file and put
the code for `network::server` in the *network* directory in
*src/network/server.rs* instead of just being able to extract the
`network::server` module into *src/server.rs*? The reason is that Rust wouldn’t
be able to recognize that `server` was supposed to be a submodule of `network`
if the *server.rs* file was in the *src* directory. To clarify Rust’s behavior
here, let’s consider a different example with the following module hierarchy,
where all the definitions are in *src/lib.rs*:

```text
communicator
 ├── client
 └── network
     └── client
```

In this example, we have three modules again: `client`, `network`, and
`network::client`. Following the same steps we did earlier for extracting
modules into files, we would create *src/client.rs* for the `client` module.
For the `network` module, we would create *src/network.rs*. But we wouldn’t be
able to extract the `network::client` module into a *src/client.rs* file
because that already exists for the top-level `client` module! If we could put
the code for *both* the `client` and `network::client` modules in the
*src/client.rs* file, Rust wouldn’t have any way to know whether the code was
for `client` or for `network::client`.

Therefore, in order to extract a file for the `network::client` submodule of
the `network` module, we needed to create a directory for the `network` module
instead of a *src/network.rs* file. The code that is in the `network` module
then goes into the *src/network/mod.rs* file, and the submodule
`network::client` can have its own *src/network/client.rs* file. Now the
top-level *src/client.rs* is unambiguously the code that belongs to the
`client` module.

### Rules of Module Filesystems

Let’s summarize the rules of modules with regard to files:

* If a module named `foo` has no submodules, you should put the declarations
  for `foo` in a file named *foo.rs*.
* If a module named `foo` does have submodules, you should put the declarations
  for `foo` in a file named *foo/mod.rs*.

These rules apply recursively, so if a module named `foo` has a submodule named
`bar` and `bar` does not have submodules, you should have the following files
in your *src* directory:

```text
├── foo
│   ├── bar.rs (contains the declarations in `foo::bar`)
│   └── mod.rs (contains the declarations in `foo`, including `mod bar`)
```

The modules should be declared in their parent module’s file using the `mod`
keyword.

Next, we’ll talk about the `pub` keyword and get rid of those warnings!
