## `mod` y el Sistema de Archivos

Empezaremos nuestro ejemplo de módulo haciendo un nuevo proyecto con Cargo, pero en lugar
de crear un cajón binario, haremos un cajón de biblioteca: un proyecto que otras
personas pueden llevar a sus proyectos como una dependencia. Por ejemplo, la caja
`rand` discutida en el Capítulo 2 es una caja de biblioteca que usamos como dependencia 
en el proyecto del juego de adivinanzas.

Vamos a crear el esqueleto de una biblioteca que proporciona algo de funcionalidad 
a la red general; nos concentraremos en la organización de los módulos y 
funciones, pero no nos preocuparemos de qué código entra en los cuerpos funcionales. 
Llamaremos a nuestra biblioteca `communicator`. De forma predeterminada, Cargo creará una biblioteca a menos
que se especifique otro tipo de proyecto: si omitimos la opción `--bin` que hemos
estado usando en todos los capítulos anteriores, nuestro proyecto será una 
biblioteca:

```text
$ cargo new communicator
$ cd communicator
```

Nota que Cargo generó *src/lib.rs* en lugar de *src/main.rs*. Dentro de
*src/lib.rs* encontraremos lo siguiente:

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

Cargo crea un ejemplo de prueba para ayudarnos a poner en marcha nuestra biblioteca, en lugar 
de "¡Hola, mundo!" binario que conseguimos cuando usamos la opción `---bin`. Veremos
la sintaxis `#[]` y `mod tests` en la sección "Usando `super` para acceder a 
un módulo padre" más adelante en este capítulo, pero por ahora, deja este código en
la parte inferior de *src/lib.rs*.

Debido a que no tenemos un archivo *src/main.rs*, no hay nada para que Cargo 
ejecute con el comando `cargo run`. Por lo tanto, usaremos el comando `cargo build`
para compilar nuestro código del cajón de la biblioteca.

Examinaremos las diferentes opciones para organizar el código de tu biblioteca, que sean
adecuadas en una variedad de situaciones, dependiendo de la intención del código.

### Definiciones del Módulo

Para nuestra biblioteca de redes `communicator`, primero definiremos un módulo 
llamado `network` que contiene la definición de una función llamada `connect`. Cada 
definición de módulo en Rust comienza con la palabra clave `mod`. Añade este código al 
principio del archivo *src/lib.rs*, encima del código de prueba:

<span class="filename">Filename: src/lib.rs</span>

```rust
mod network {
    fn connect() {
    }
}
```

Después de la palabra clave `mod`, ponemos el nombre del módulo, `network`, y luego 
un bloque de código entre llaves. Todo dentro de este bloque está dentro del
espacio de nombres `network`. En este caso, tenemos una sola función, `connect`. Si 
quisiéramos llamar esta función desde un código fuera del módulo `network`, 
tendríamos que especificar el módulo y utilizar la sintaxis del espacio de nombres `::`, así:
`network::connect()` en lugar de simplemente `connect()`.

También podemos tener múltiples módulos, uno al lado del otro, en el mismo archivo *src/lib.rs*.
Por ejemplo, para tener también un módulo `client` que tenga una función llamada `connect`
también, podemos agregarlo como se muestra en eñ Listado 7-1:

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

<span class="caption">Listado 7-1: El módulo `network` y el módulo  `client`
definidos uno al lado del otro en *src/lib. rs*</span>

Ahora tenemos una función `network::connect` y una función `client::connect`.
Éstos pueden tener una funcionalidad completamente diferente, y los nombres de las funciones 
no entran en conflicto entre sí porque están en módulos diferentes.

En este caso, como estamos construyendo una biblioteca, el archivo que sirve como
punto de entrada para construir nuestra biblioteca es *src/lib.rs*. Sin embargo, con respecto a
la creación de módulos, no hay nada especial en *src/lib.rs*. También podríamos
crear módulos en *src/main.rs* para un cajón binario de la misma manera que 
creamos módulos en *src/lib.rs* para el cajón de la biblioteca. De hecho, podemos colocar 
módulos dentro de los módulos, lo que puede ser útil a medida que los módulos crecen para mantener
las funciones relacionadas organizadas y separadas entre sí. La 
elección de cómo organizar tu código depende de cómo pienses sobre la
relación entre las partes de tu código. Por ejemplo, el código `client` 
y su función `conect` podrían tener más sentido para los usuarios de nuestra biblioteca si 
estuvieran dentro del espacio de nombres `network`, como en el Listado 7-2:

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

<span class="caption">Listado 7-2: Moviendo el módulo `client` dentro del
módulo `network`</span>

En tu archivo *src/lib.rs*, reemplaza las definiciones existentes de `mod network` y `mod client`
con las definiciones del Listado 7-2, que tienen el módulo `client` como módulo 
interno de `network`. Ahora tenemos las funciones `network::connect` y 
`network::client::connect`: de nuevo, las dos funciones llamadas `connect` no
entran en conflicto entre sí porque están en diferentes espacios de nombres.

De este modo, los módulos forman una jerarquía. El contenido de *src/lib.rs* se encuentra en
el nivel superior, y los submódulos en los niveles inferiores. Aquí se muestra cómo se 
ve la organización de nuestro ejemplo en el Listado 7-1 cuando se piensa que es una
jerarquía:

```text
communicator
 ├── network
 └── client
```

Y aquí está la jerarquía correspondiente al ejemplo del Listado 7-2:

```text
communicator
 └── network
     └── client
```

La jerarquía muestra que en el Listado 7-2, `client` es un hijo del módulo 
`network` en lugar de un hermano. Los proyectos más complicados pueden tener muchos módulos,
y necesitarán organizarse lógicamente para poder hacer un seguimiento de ellos. Lo que
"lógicamente" significa que tu proyecto depende de ti y depende de cómo tu y los
usuarios de tu biblioteca piensen sobre el dominio de tu proyecto. Utiliza las técnicas mostradas
aquí para crear módulos lado a lado y módulos anidados en cualquier estructura
que desees.

### Moving Modules to Other Files

Modules form a hierarchical structure, much like another structure in computing
that you’re used to: filesystems! We can use Rust’s module system along with
multiple files to split up Rust projects so not everything lives in
*src/lib.rs* or *src/main.rs*. For this example, let’s start with the code in
Listing 7-3:

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

<span class="caption">Listing 7-3: Three modules, `client`, `network`, and
`network::server`, all defined in *src/lib.rs*</span>

The file *src/lib.rs* has this module hierarchy:

```text
communicator
 ├── client
 └── network
     └── server
```

If these modules had many functions, and those functions were becoming lengthy,
it would be difficult to scroll through this file to find the code we wanted to
work with. Because the functions are nested inside one or more `mod` blocks,
the lines of code inside the functions will start getting lengthy as well.
These would be good reasons to separate the `client`, `network`, and `server`
modules from *src/lib.rs* and place them into their own files.

First, replace the `client` module code with only the declaration of the
`client` module, so that your *src/lib.rs* looks like code shown in Listing 7-4:

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

<span class="caption">Listing 7-4: Extracting the contents of the `client` module but leaving the declaration in *src/lib.rs*</span>

We’re still *declaring* the `client` module here, but by replacing the block
with a semicolon, we’re telling Rust to look in another location for the code
defined within the scope of the `client` module. In other words, the line `mod
client;` means:

```rust,ignore
mod client {
    // contents of client.rs
}
```

Now we need to create the external file with that module name. Create a
*client.rs* file in your *src/* directory and open it. Then enter the
following, which is the `connect` function in the `client` module that we
removed in the previous step:

<span class="filename">Filename: src/client.rs</span>

```rust
fn connect() {
}
```

Note that we don’t need a `mod` declaration in this file because we already
declared the `client` module with `mod` in *src/lib.rs*. This file just
provides the *contents* of the `client` module. If we put a `mod client` here,
we’d be giving the `client` module its own submodule named `client`!

Rust only knows to look in *src/lib.rs* by default. If we want to add more
files to our project, we need to tell Rust in *src/lib.rs* to look in other
files; this is why `mod client` needs to be defined in *src/lib.rs* and can’t
be defined in *src/client.rs*.

Now the project should compile successfully, although you’ll get a few
warnings. Remember to use `cargo build` instead of `cargo run` because we have
a library crate rather than a binary crate:

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

These warnings tell us that we have functions that are never used. Don’t worry
about these warnings for now; we’ll address them later in this chapter in the
“Controlling Visibility with `pub`” section. The good news is that they’re just
warnings; our project built successfully!

Next, let’s extract the `network` module into its own file using the same
pattern. In *src/lib.rs*, delete the body of the `network` module and add a
semicolon to the declaration, like so:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
mod client;

mod network;
```

Then create a new *src/network.rs* file and enter the following:

<span class="filename">Filename: src/network.rs</span>

```rust
fn connect() {
}

mod server {
    fn connect() {
    }
}
```

Notice that we still have a `mod` declaration within this module file; this is
because we still want `server` to be a submodule of `network`.

Run `cargo build` again. Success! We have one more module to extract: `server`.
Because it’s a submodule—that is, a module within a module—our current tactic
of extracting a module into a file named after that module won’t work. We’ll
try anyway so you can see the error. First, change *src/network.rs* to have
`mod server;` instead of the `server` module’s contents:

<span class="filename">Filename: src/network.rs</span>

```rust,ignore
fn connect() {
}

mod server;
```

Then create a *src/server.rs* file and enter the contents of the `server`
module that we extracted:

<span class="filename">Filename: src/server.rs</span>

```rust
fn connect() {
}
```

When we try to `cargo build`, we’ll get the error shown in Listing 7-5:

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

<span class="caption">Listing 7-5: Error when trying to extract the `server`
submodule into *src/server.rs*</span>

The error says we `cannot declare a new module at this location` and is
pointing to the `mod server;` line in *src/network.rs*. So *src/network.rs* is
different than *src/lib.rs* somehow: keep reading to understand why.

The note in the middle of Listing 7-5 is actually very helpful because it
points out something we haven’t yet talked about doing:

```text
note: maybe move this module `network` to its own directory via
`network/mod.rs`
```

Instead of continuing to follow the same file naming pattern we used
previously, we can do what the note suggests:

1. Make a new *directory* named *network*, the parent module’s name.
2. Move the *src/network.rs* file into the new *network* directory, and
   rename it to *src/network/mod.rs*.
3. Move the submodule file *src/server.rs* into the *network* directory.

Here are commands to carry out these steps:

```text
$ mkdir src/network
$ mv src/network.rs src/network/mod.rs
$ mv src/server.rs src/network
```

Now when we try to run `cargo build`, compilation will work (we’ll still have
warnings though). Our module layout still looks like this, which is exactly the
same as it did when we had all the code in *src/lib.rs* in Listing 7-3:

```text
communicator
 ├── client
 └── network
     └── server
```

The corresponding file layout now looks like this:

```text
├── src
│   ├── client.rs
│   ├── lib.rs
│   └── network
│       ├── mod.rs
│       └── server.rs
```

Así que cuando quisimos extraer el módulo `network::server`, ¿por qué tuvimos que 
cambiar también el archivo *src/network.rs* al archivo *src/network/mod.rs* y poner el
código para `network::server` en el directorio *network* en 
*src/network/server.rs* en lugar de simplemente poder extraer el
módulo `network::server` en *src/server.rs*? La razón es que Rust no sería 
capaz de reconocer que el `server` debía ser un submódulo de `network`
si el archivo *server.rs* estuviera en el directorio *src*. Para aclarar el comportamiento de Rust 
aquí, vamos a considerar un ejemplo diferente con la siguiente jerarquía de módulos, 
donde todas las definiciones están en *src/lib.rs*:

```text
communicator
 ├── client
 └── network
     └── client
```

En este ejemplo, tenemos de nuevo tres módulos: `client`, `network` y 
`network::client`. Siguiendo los mismos pasos que hicimos anteriormente para extraer 
módulos en archivos, crearíamos *src/client.rs* para el módulo `client`.
Para el módulo `network`, crearíamos *src/network.rs*. Pero no podríamos
extraer el módulo `network::client` en un archivo *src/client.rs*
porque ya existe para el módulo `client` de alto nivel! Si pudiéramos poner
el código para *ambos* módulos `client` y `network::client` en el
archivo *src/client.rs*, Rust no tendría forma de saber si el código era 
para `client` o para `network::client`.

Por lo tanto, para extraer un archivo para el submódulo `networl::client` del
módulo `network`, necesitábamos crear un directorio para el módulo `network`
en lugar de un archivo *src/network.rs*. El código que se encuentra en el módulo `network`
entra en el archivo *src/network/mod.rs*, y el submódulo 
`network::client` puede tener su propio archivo *src/network/client.rs*. Ahora el
nivel superior *src/client.rs* es inequívocamente el código que pertenece al
módulo `client`.

### Reglas de los Sistemas de Ficheros del Módulo

Resumamos las reglas de los módulos con respecto a los archivos:

* Si un módulo llamado `foo` no tiene submódulos, debes poner las declaraciones
para `foo` en un archivo llamado *foo.rs*.
* Si un módulo llamado `foo` tiene submódulos, debes poner las declaraciones
para `foo` en un archivo llamado *foo/mod.rs*.

Estas reglas se aplican recursivamente, así que si un módulo llamado `foo` tiene un submódulo llamado
`bar` y `bar` no tiene submódulos, deberías tener los siguientes archivos
en tu directorio *src*:

```text
├── foo
│   ├── bar.rs (contains the declarations in `foo::bar`)
│   └── mod.rs (contains the declarations in `foo`, including `mod bar`)
```

Los módulos deben declararse en el archivo del módulo padre utilizando la palabra 
clave `mod`.

A continuación, hablaremos de la palabra clave `pub` y nos desharemos de esas advertencias!
