## Publicando un Crate en Crates.io

Ya hemos usado crates de [crates.io](https://crates.io)<!-- ignorar --> como
dependencias de nuestros proyectos, pero usted también puede compartir su código
para que otra gente lo use publicando sus propios paquetes. El registro de crates
en [crates.io](https://crates.io)<!-- ignorar --> distribuye el código fuente de
sus paquetes, eso significa que almacena códigos de fuente pública principalmente.

Rust y Cargo tienen funcionalidades que le ayudan a usted hacer sus paquetes
publicados más accesibles y fáciles de usar para otras personas. Hablaremos sobre
algunas de estas funcionalidades luego, y explicaremos cómo publicar un paquete.

### Haciendo Comentarios Útiles de Documentación

Documentar sus paquetes de forma precisa le ayudará a otros usuarios saber cómo
y cuando usarlos, entonces sí vale la pena usar su tiempo para escribir la
documentación. En el Capítulo 3, discutimos cómo comentar códigos en Rust usando
`//`. Rust también tiene un tipo de comentario particular para la documentación,
el cual se conoce convenientemente como *comentario de documentación*, que generará
documentación en formato HTML. El formato HTML muestra el contenido de los
comentarios de documentación para objetos de API públicos creados para programadores
que estén interesados en saber cómo *usar* su crate, a diferencia de saber cómo
su crate fue *implementado*.

Los comentarios de documentación usan `///` en vez de `//` y soportan la notación
Markdown (notación reducida) para formatear el texto si usted quiere usarlo. Usted
coloca comentarios de documentación justo antes del objeto que esté documentando.
El Listado 14-1 muestra comentarios de documentación para una función `add_one`
en un crate llamado `my_crate`:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
/// Adds one to the number given.
///
/// # Examples
///
/// ```
/// let five = 5;
///
/// assert_eq!(6, my_crate::add_one(5));
/// ```
pub fn add_one(x: i32) -> i32 {
    x + 1
}
```

<span class="caption">Listing 14-1: A documentation comment for a
function</span>

Justo aquí, estamos dando una descripción de qué es lo que hace la función
`add_one`, comenzamos una sección con la cabecera `Examples`, y luego ofrecemos
código que demuestra cómo se usa la función `add_one`. Podemos generar la
documentación HTML desde este comentario de documentación si ejecutamos
`cargo doc`. Este comando ejecuta la herramienta distribuída con Rust `rustdoc`
e introduce la documentación HTML generada en el directorio *target/doc*.

Por conveniencia, ejecutar `cargo doc --open` construirá el HTML para su
documentación del crate actual (así como también construirá la documentación
para todas las dependencias de su crate) y abrirá el resultado en un buscador web.
Navegue a la función `add_one` y usted verá cómo se muestra el texto en los 
comentarios de documentación, como en la Figura 14-1:

<img alt="Rendered HTML documentation for the `add_one` function of `my_crate`" src="img/trpl14-01.png" class="center" />

<span class="caption">Figure 14-1: HTML documentation for the `add_one`
function</span>

#### Secciones Comúnmente Usadas

Nosotros usamos la cabecera Markdown `# Examples` en el Listado 14-1 para crear
una sección con el título "Examples" en el HTML. Algunas otras secciones que
los autores de crates suelen usar en sus documentaciones incluyen:

* **Pánicos**: Los escenarios en los que la función que está siendo documentada
  podría `panic!` (entrar en pánico). Los llamadores de la función que no quieran
  que sus programas entren en pánico deberán asegurarse de no llamar la función
  en estas situaciones.
* **Errores**: Si la función retorna un `Result` (resultado), describir los
  tipos de errores que podrían ocurrir y qué condiciones son las que podrían
  causar que estos errores sean retornados puede ser útil para los llamadores,
  para ellos así ser capaces de escribir código que maneje los diferentes tipos
  de errores de diferentes formas.
* **Seguridad**: Si llamar a la función es `unsafe` (inseguro) (discutiremos la
  inseguridad en el capítulo 19), entonces tendría que haber una sección explicando
  el porqué la función no es segura y cubrir las invariantes que la función espera
  que los llamadores tengan.

La mayoría de las secciones de comentarios de documentación no necesitan todas
estas secciones, pero es una buena lista para recordarle los aspectos de su código
que otras personas querrán saber a la hora de llamarlo.

#### Comentarios de Documentación como Exámenes

Añadir ejemplos en bloques de códigos en sus comentarios de documentación puede
demostrar precisamente cómo usar su librería, y hacer esto tiene un bono adicional:
ejecutar `cargo test` ejecutará los ejemplos de códigos en su documentación,
¡Examinándolos! Nada es mejor que la documentación ejemplificada. Pero nada es peor
que los ejemplos que no funcionan porque el código ha cambiado desde que la
documentación se escribió. Ejecute `cargo text` con la documentación para la función
`add_one` desde el Listado 14-1; usted debería de ver una sección en los resultados
del exámen que se verá así:

```text
   Doc-tests my_crate

running 1 test
test src/lib.rs - add_one (line 5) ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

Ahora cambie, ya sea la función o el ejemplo, para que el `asser_eq!` en el
ejemplo entre en pánico. Ejecute `cargo test` otra vez; ¡Usted verá que los exámenes
de la documentación del ejemplo y del código no están sincronizados entre ellos!

#### Comentando Objetos Contenidos

Otro tipo de comentario de documento, `//!`, añade documentación al objeto que
contiene los comentarios en vez de añadir documentación a los objetos siguientes
al comentario. Nosotros generalmente usamos estos comentarios de documento dentro
del archivo raíz crate (*src/lib.rs* por convención) o dentro de un módulo que
usamos para comentar el crate, o el módulo completo.

Por ejemplo, si queremos añadir documentación que describa el uso del crate
`my_crate` que contiene la función `add_one`, podemos añadir comentarios de
documentación  que comiencen con `//!` al principio del archivo *src/lib.rs*,
como es mostrado en el Listado 14-2:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
//! # My Crate
//!
//! `my_crate` is a collection of utilities to make performing certain
//! calculations more convenient.

/// Adds one to the number given.
// --snip--
```

<span class="caption">Listing 14-2: Documentation for the `my_crate` crate as a
whole</span>

Note que no hay ningún código luego de la última línea que comienza con `//!`.
Ya que comenzamos con el comentario con `//!` en vez de `///`, estamos documentando
el objeto que contiene este comentario en lugar de un objeto que le sigue al
comentario. En este caso, el objeto que contiene este comentario es el archivo
*src/lib.rs*, el cual es la raíz del crate. Estos comentarios describen todo el
crate.

Cuando ejecutamos `cargo doc --open`, estos comentarios se mostrarán en la portada
de la documentación para `my_crate` sobre la lista de objetos públicos en el crate,
como es mostrado en la Figura 14-2:

<img alt="Rendered HTML documentation with a comment for the crate as a whole" src="img/trpl14-02.png" class="center" />

<span class="caption">Figure 14-2: Rendered documentation for `my_crate`
including the comment describing the crate as a whole</span>

Los comentarios de documentación que estén dentro de objetos son útiles para
describir crates y módulos especialmente. Úselos para explicar el motivo general
del contenedor, para ayudar a los usuarios de su crate a entender su organización.

### Exportando un API Público Conveniente con `pub use`

En el Capítulo 7, nosotros cubrimos cómo organizar nuestro código dentro de
módulos usando la palabra reservada `mod`, cómo hacer objetos públicos usando
la palabra reservada `pub`, y cómo traer objetos al ambiente usando la palabra
reservada `use`. Sin embargo, la estructura a la que usted le consigue el sentido
mientras desarrolla un crate podría no ser muy conveniente para sus usuarios.
Usted podría querer organizar sus registros en una jerarquía conteniendo múltiples
niveles, pero las personas que quieran usar un tipo que usted haya definido en lo
profundo de la jerarquía podrían tener problemas encontrando que esos tipos existen.
También podría molestarles tener que usar `use`
`my_crate::some_module::another_module::UsefulType;` en vez de `use`
`my_crate::UsefulType;`.

The structure of your public API is a major consideration when publishing a
crate. People who use your crate are less familiar with the structure than you
are and might have difficulty finding the pieces they want to use if your crate
has a large module hierarchy.

The good news is that if the structure *isn’t* convenient for others to use
from another library, you don’t have to rearrange your internal organization:
instead, you can re-export items to make a public structure that’s different
than your private structure by using `pub use`. Re-exporting takes a public
item in one location and makes it public in another location, as if it was
defined in the other location instead.

For example, say we made a library named `art` for modeling artistic concepts.
Within this library are two modules: a `kinds` module containing two enums
named `PrimaryColor` and `SecondaryColor`, and a `utils` module containing a
function named `mix`, as shown in Listing 14-3:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
//! # Art
//!
//! A library for modeling artistic concepts.

pub mod kinds {
    /// The primary colors according to the RYB color model.
    pub enum PrimaryColor {
        Red,
        Yellow,
        Blue,
    }

    /// The secondary colors according to the RYB color model.
    pub enum SecondaryColor {
        Orange,
        Green,
        Purple,
    }
}

pub mod utils {
    use kinds::*;

    /// Combines two primary colors in equal amounts to create
    /// a secondary color.
    pub fn mix(c1: PrimaryColor, c2: PrimaryColor) -> SecondaryColor {
        // --snip--
    }
}
```

<span class="caption">Listing 14-3: An `art` library with items organized into
`kinds` and `utils` modules</span>

Figure 14-3 shows what the front page of the documentation for this crate
generated by `cargo doc` would look like:

<img alt="Rendered documentation for the `art` crate that lists the `kinds` and `utils` modules" src="img/trpl14-03.png" class="center" />

<span class="caption">Figure 14-3: Front page of the documentation for `art`
that lists the `kinds` and `utils` modules</span>

Note that the `PrimaryColor` and `SecondaryColor` types aren’t listed on the
front page, nor is the `mix` function. We have to click `kinds` and `utils` to
see them.

Another crate that depends on this library would need `use` statements that
import the items from `art`, including specifying the module structure that’s
currently defined. Listing 14-4 shows an example of a crate that uses the
`PrimaryColor` and `mix` items from the `art` crate:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
extern crate art;

use art::kinds::PrimaryColor;
use art::utils::mix;

fn main() {
    let red = PrimaryColor::Red;
    let yellow = PrimaryColor::Yellow;
    mix(red, yellow);
}
```

<span class="caption">Listing 14-4: A crate using the `art` crate’s items with
its internal structure exported</span>

The author of the code in Listing 14-4, which uses the `art` crate, had to
figure out that `PrimaryColor` is in the `kinds` module and `mix` is in the
`utils` module. The module structure of the `art` crate is more relevant to
developers working on the `art` crate than developers using the `art` crate.
The internal structure that organizes parts of the crate into the `kinds`
module and the `utils` module doesn’t contain any useful information for
someone trying to understand how to use the `art` crate. Instead, the `art`
crate’s module structure causes confusion because developers have to figure out
where to look, and the structure is inconvenient because developers must
specify the module names in the `use` statements.

To remove the internal organization from the public API, we can modify the
`art` crate code in Listing 14-3 to add `pub use` statements to re-export the
items at the top level, as shown in Listing 14-5:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
//! # Art
//!
//! A library for modeling artistic concepts.

pub use kinds::PrimaryColor;
pub use kinds::SecondaryColor;
pub use utils::mix;

pub mod kinds {
    // --snip--
}

pub mod utils {
    // --snip--
}
```

<span class="caption">Listing 14-5: Adding `pub use` statements to re-export
items</span>

The API documentation that `cargo doc` generates for this crate will now list
and link re-exports on the front page, as shown in Figure 14-4, which makes the
`PrimaryColor` and `SecondaryColor` types and the `mix` function easier to find:

<img alt="Rendered documentation for the `art` crate with the re-exports on the front page" src="img/trpl14-04.png" class="center" />

<span class="caption">Figure 14-4: Front page of the documentation for `art`
that lists the re-exports</span>

The `art` crate users can still see and use the internal structure from Listing
14-3 as demonstrated in Listing 14-4, or they can use the more convenient
structure in Listing 14-5, as shown in Listing 14-6:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
extern crate art;

use art::PrimaryColor;
use art::mix;

fn main() {
    // --snip--
}
```

<span class="caption">Listing 14-6: A program using the re-exported items from
the `art` crate</span>

In cases where there are many nested modules, re-exporting the types at the top
level with `pub use` can make a significant difference in the experience of
people who use the crate.

Creating a useful public API structure is more of an art than a science, and
you can iterate to find the API that works best for your users. Choosing `pub`
`use` gives you flexibility in how you structure your crate internally and
decouples that internal structure with what you present to your users. Look at
some of the code of crates you’ve installed to see if their internal structure
differs from their public API.

### Setting Up a Crates.io Account

Before you can publish any crates, you need to create an account on
[crates.io](https://crates.io)<!-- ignore --> and get an API token. To do so,
visit the home page at [crates.io](https://crates.io)<!-- ignore --> and log in
via a GitHub account: the GitHub account is currently a requirement, but the
site might support other ways of creating an account in the future. Once you’re
logged in, visit your account settings at
[https://crates.io/me/](https://crates.io/me/)<!-- ignore --> and retrieve your
API key. Then run the `cargo` `login` command with your API key, like this:

```text
$ cargo login abcdefghijklmnopqrstuvwxyz012345
```

This command will inform Cargo of your API token and store it locally in
*~/.cargo/credentials*. Note that this token is a *secret*: do not share it
with anyone else. If you do share it with anyone for any reason, you should
revoke it and generate a new token on [crates.io](https://crates.io)<!-- ignore
-->.

### Before Publishing a New Crate

Now that you have an account, let’s say you have a crate you want to publish.
Before publishing, you’ll need to add some metadata to your crate by adding it
to the `[package]` section of the crate’s *Cargo.toml* file.

Your crate will need a unique name. While you’re working on a crate locally,
you can name a crate whatever you’d like. However, crate names on
[crates.io](https://crates.io)<!-- ignore --> are allocated on a first-come,
first-served basis. Once a crate name is taken, no one else can publish a crate
with that name. Search for the name you want to use on the site to find out if
it has been used. If it hasn’t, edit the name in the *Cargo.toml* file under
`[package]` to use the name for publishing, like so:

<span class="filename">Filename: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
```

Even if you’ve chosen a unique name, when you run `cargo publish` to publish
the crate at this point, you’ll get a warning and then an error:

```text
$ cargo publish
    Updating registry `https://github.com/rust-lang/crates.io-index`
warning: manifest has no description, license, license-file, documentation,
homepage or repository.
--snip--
error: api errors: missing or empty metadata fields: description, license.
```

The reason is that you’re missing some crucial information: a description and
license are required so people will know what your crate does and under what
terms they can use it. To rectify this error, you need to include this
information in the *Cargo.toml* file.

Add a description that is just a sentence or two, because it will appear with
your crate in search results. For the `license` field, you need to give a
*license identifier value*. The Linux Foundation’s Software Package Data
Exchange (SPDX) at *http://spdx.org/licenses/* lists the identifiers you can
use for this value. For example, to specify that you’ve licensed your crate
using the MIT License, add the `MIT` identifier:

<span class="filename">Filename: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
license = "MIT"
```

If you want to use a license that doesn’t appear in the SPDX, you need to place
the text of that license in a file, include the file in your project, and then
use `license-file` to specify the name of that file instead of using the
`license` key.

Guidance on which license is appropriate for your project is beyond the scope
of this book. Many people in the Rust community license their projects in the
same way as Rust by using a dual license of `MIT OR Apache-2.0`, which
demonstrates that you can also specify multiple license identifiers separated
by `OR` to have multiple licenses for your project.

With a unique name, the version, the author details that `cargo new` added
when you created the crate, your description, and a license added, the
*Cargo.toml* file for a project that is ready to publish might look like this:

<span class="filename">Filename: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
version = "0.1.0"
authors = ["Your Name <you@example.com>"]
description = "A fun game where you guess what number the computer has chosen."
license = "MIT OR Apache-2.0"

[dependencies]
```

[Cargo’s documentation](https://doc.rust-lang.org/cargo/) describes other
metadata you can specify to ensure others can discover and use your crate more
easily!

### Publishing to Crates.io

Now that you’ve created an account, saved your API token, chosen a name for
your crate, and specified the required metadata, you’re ready to publish!
Publishing a crate uploads a specific version to
[crates.io](https://crates.io)<!-- ignore --> for others to use.

Be careful when publishing a crate because a publish is *permanent*. The
version can never be overwritten, and the code cannot be deleted. One major
goal of [crates.io](https://crates.io)<!-- ignore --> is to act as a permanent
archive of code so that builds of all projects that depend on crates from
[crates.io](https://crates.io)<!-- ignore --> will continue to work. Allowing
version deletions would make fulfilling that goal impossible. However, there is
no limit to the number of crate versions you can publish.

Run the `cargo publish` command again. It should succeed now:

```text
$ cargo publish
 Updating registry `https://github.com/rust-lang/crates.io-index`
Packaging guessing_game v0.1.0 (file:///projects/guessing_game)
Verifying guessing_game v0.1.0 (file:///projects/guessing_game)
Compiling guessing_game v0.1.0
(file:///projects/guessing_game/target/package/guessing_game-0.1.0)
 Finished dev [unoptimized + debuginfo] target(s) in 0.19 secs
Uploading guessing_game v0.1.0 (file:///projects/guessing_game)
```

Congratulations! You’ve now shared your code with the Rust community, and
anyone can easily add your crate as a dependency of their project.

### Publishing a New Version of an Existing Crate

When you’ve made changes to your crate and are ready to release a new version,
you change the `version` value specified in your *Cargo.toml* file and
republish. Use the [Semantic Versioning rules][semver] to decide what an
appropriate next version number is based on the kinds of changes you’ve made.
Then run `cargo publish` to upload the new version.

[semver]: http://semver.org/

### Removing Versions from Crates.io with `cargo yank`

Although you can’t remove previous versions of a crate, you can prevent any
future projects from adding them as a new dependency. This is useful when a
crate version is broken for one reason or another. In such situations, Cargo
supports *yanking* a crate version.

Yanking a version prevents new projects from starting to depend on that version
while allowing all existing projects that depend on it to continue to download
and depend on that version. Essentially, a yank means that all projects with a
*Cargo.lock* will not break, and any future *Cargo.lock* files generated will
not use the yanked version.

To yank a version of a crate, run `cargo yank` and specify which version you
want to yank:

```text
$ cargo yank --vers 1.0.1
```

By adding `--undo` to the command, you can also undo a yank and allow projects
to start depending on a version again:

```text
$ cargo yank --vers 1.0.1 --undo
```

A yank *does not* delete any code. For example, the yank feature is not
intended for deleting accidentally uploaded secrets. If that happens, you must
reset those secrets immediately.
