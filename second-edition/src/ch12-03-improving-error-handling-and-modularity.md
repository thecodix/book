## Refactoring to Improve Modularity and Error Handling

To improve our program, we’ll fix four problems that have to do with the
program’s structure and how it’s handling potential errors.

First, our `main` function now performs two tasks: it parses arguments and
opens files. For such a small function, this isn’t a major problem. However, if
we continue to grow our program inside `main`, the number of separate tasks the
`main` function handles will increase. As a function gains responsibilities, it
becomes more difficult to reason about, harder to test, and harder to change
without breaking one of its parts. It’s best to separate functionality so each
function is responsible for one task.

This issue also ties into the second problem: although `query` and `filename`
are configuration variables to our program, variables like `f` and `contents`
are used to perform the program’s logic. The longer `main` becomes, the more
variables we’ll need to bring into scope; the more variables we have in scope,
the harder it will be to keep track of the purpose of each. It’s best to group
the configuration variables into one structure to make their purpose clear.

The third problem is that we’ve used `expect` to print an error message when
opening the file fails, but the error message just prints `file not found`.
Opening a file can fail in a number of ways besides the file being missing: for
example, the file might exist, but we might not have permission to open it.
Right now, if we’re in that situation, we’d print the `file not found` error
message that would give the user the wrong information!

Fourth, we use `expect` repeatedly to handle different errors, and if the user
runs our program without specifying enough arguments, they’ll get an `index out
of bounds` error from Rust that doesn’t clearly explain the problem. It would
be best if all the error handling code was in one place so future maintainers
have only one place to consult in the code if the error handling logic needs to
change. Having all the error handling code in one place will also ensure that
we’re printing messages that will be meaningful to our end users.

Let’s address these four problems by refactoring our project.

### Separation of Concerns for Binary Projects

The organizational problem of allocating responsibility for multiple tasks to
the `main` function is common to many binary projects. As a result, the Rust
community has developed a type of guideline process for splitting the separate
concerns of a binary program when `main` starts getting large. The process has
the following steps:

* Split your program into a *main.rs* and a *lib.rs*, and move your program’s
logic to *lib.rs*.
* While your command line parsing logic is small, it can remain in *main.rs*.
* When the command line parsing logic starts getting complicated, extract it
from *main.rs* and move it to *lib.rs*.
* The responsibilities that remain in the `main` function after this process
should be limited to:

  * Calling the command line parsing logic with the argument values
  * Setting up any other configuration
  * Calling a `run` function in *lib.rs*
  * Handling the error if `run` returns an error

This pattern is about separating concerns: *main.rs* handles running the
program, and *lib.rs* handles all the logic of the task at hand. Because we
can’t test the `main` function directly, this structure lets us test all of our
program’s logic by moving it into functions in *lib.rs*. The only code that
remains in *main.rs* will be small enough to verify its correctness by reading
it. Let’s rework our program by following this process.

#### Extracting the Argument Parser

We’ll extract the functionality for parsing arguments into a function that
`main` will call to prepare for moving the command line parsing logic to
*src/lib.rs*. Listing 12-5 shows the new start of `main` that calls a new
function `parse_config`, which we’ll define in *src/main.rs* for the moment.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
fn main() {
    let args: Vec<String> = env::args().collect();

    let (query, filename) = parse_config(&args);

    // --snip--
}

fn parse_config(args: &[String]) -> (&str, &str) {
    let query = &args[1];
    let filename = &args[2];

    (query, filename)
}
```

<span class="caption">Listing 12-5: Extracting a `parse_config` function from
`main`</span>

We’re still collecting the command line arguments into a vector, but instead of
assigning the argument value at index `1` to the variable `query` and the
argument value at index `2` to the variable `filename` within the `main`
function, we pass the whole vector to the `parse_config` function. The
`parse_config` function then holds the logic that determines which argument
goes in which variable and passes the values back to `main`. We still create
the `query` and `filename` variables in `main`, but `main` no longer has the
responsibility of determining how the command line arguments and variables
correspond.

This rework may seem like overkill for our small program, but we’re refactoring
in small, incremental steps. After making this change, run the program again to
verify that the argument parsing still works. It’s good to check your progress
often, because that will help you identify the cause of problems when they
occur.

#### Grouping Configuration Values

We can take another small step to improve the `parse_config` function further.
At the moment, we’re returning a tuple, but then we immediately break that
tuple into individual parts again. This is a sign that perhaps we don’t have
the right abstraction yet.

Another indicator that shows there’s room for improvement is the `config` part
of `parse_config`, which implies that the two values we return are related and
are both part of one configuration value. We’re not currently conveying this
meaning in the structure of the data other than grouping the two values into a
tuple: we could put the two values into one struct and give each of the struct
fields a meaningful name. Doing so will make it easier for future maintainers
of this code to understand how the different values relate to each other and
what their purpose is.

> Note: Some people call this anti-pattern of using primitive values when a
> complex type would be more appropriate *primitive obsession*.

Listing 12-6 shows the addition of a struct named `Config` defined to have
fields named `query` and `filename`. We’ve also changed the `parse_config`
function to return an instance of the `Config` struct and updated `main` to use
the struct fields rather than having separate variables:

<span class="filename">Filename: src/main.rs</span>

```rust,should_panic
# use std::env;
# use std::fs::File;
#
fn main() {
    let args: Vec<String> = env::args().collect();

    let config = parse_config(&args);

    println!("Searching for {}", config.query);
    println!("In file {}", config.filename);

    let mut f = File::open(config.filename).expect("file not found");

    // --snip--
}

struct Config {
    query: String,
    filename: String,
}

fn parse_config(args: &[String]) -> Config {
    let query = args[1].clone();
    let filename = args[2].clone();

    Config { query, filename }
}
```

<span class="caption">Listing 12-6: Refactoring `parse_config` to return an
instance of a `Config` struct</span>

The signature of `parse_config` now indicates that it returns a `Config` value.
In the body of `parse_config`, where we used to return string slices that
reference `String` values in `args`, we now define `Config` to contain owned
`String` values. The `args` variable in `main` is the owner of the argument
values and is only letting the `parse_config` function borrow them, which means
we’d violate Rust’s borrowing rules if `Config` tried to take ownership of the
values in `args`.

We could manage the `String` data in a number of different ways, but the
easiest, though somewhat inefficient, route is to call the `clone` method on
the values. This will make a full copy of the data for the `Config` instance to
own, which takes more time and memory than storing a reference to the string
data. However, cloning the data also makes our code very straightforward
because we don’t have to manage the lifetimes of the references; in this
circumstance, giving up a little performance to gain simplicity is a worthwhile
trade-off.

> ### The Trade-Offs of Using `clone`
>
> There’s a tendency among many Rustaceans to avoid using `clone` to fix
> ownership problems because of its runtime cost. In Chapter 13, you’ll learn
> how to use more efficient methods in this type of situation. But for now,
> it’s okay to copy a few strings to continue making progress because we’ll
> make these copies only once, and our filename and query string are very
> small. It’s better to have a working program that’s a bit inefficient than to
> try to hyperoptimize code on your first pass. As you become more experienced
> with Rust, it’ll be easier to start with the most efficient solution, but for
> now, it’s perfectly acceptable to call `clone`.

We’ve updated `main` so it places the instance of `Config` returned by
`parse_config` into a variable named `config`, and we updated the code that
previously used the separate `query` and `filename` variables so it now uses
the fields on the `Config` struct instead.

Now our code more clearly conveys that `query` and `filename` are related, and
their purpose is to configure how the program will work. Any code that uses
these values knows to find them in the `config` instance in the fields named
for their purpose.

#### Creating a Constructor for `Config`

So far, we’ve extracted the logic responsible for parsing the command line
arguments from `main` and placed it in the `parse_config` function, which
helped us to see that the `query` and `filename` values were related and that
relationship should be conveyed in our code. We then added a `Config` struct to
name the related purpose of `query` and `filename`, and to be able to return
the values’ names as struct field names from the `parse_config` function.

So now that the purpose of the `parse_config` function is to create a `Config`
instance, we can change `parse_config` from being a plain function to a
function named `new` that is associated with the `Config` struct. Making this
change will make the code more idiomatic: we can create instances of types in
the standard library, such as `String`, by calling `String::new`, and by
changing `parse_config` into a `new` function associated with `Config`, we’ll
be able to create instances of `Config` by calling `Config::new`. Listing 12-7
shows the changes we need to make:

<span class="filename">Filename: src/main.rs</span>

```rust,should_panic
# use std::env;
#
fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::new(&args);

    // --snip--
}

# struct Config {
#     query: String,
#     filename: String,
# }
#
// --snip--

impl Config {
    fn new(args: &[String]) -> Config {
        let query = args[1].clone();
        let filename = args[2].clone();

        Config { query, filename }
    }
}
```

<span class="caption">Listing 12-7: Changing `parse_config` into
`Config::new`</span>

We’ve updated `main` where we were calling `parse_config` to instead call
`Config::new`. We’ve changed the name of `parse_config` to `new` and moved it
within an `impl` block, which associates the `new` function with `Config`. Try
compiling this code again to make sure it works.

### Fixing the Error Handling

Now we’ll work on fixing our error handling. Recall that attempting to access
the values in the `args` vector at index `1` or index `2` will cause the
program to panic if the vector contains fewer than three items. Try running the
program without any arguments; it will look like this:

```text
$ cargo run
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/minigrep`
thread 'main' panicked at 'index out of bounds: the len is 1
but the index is 1', src/main.rs:29:21
note: Run with `RUST_BACKTRACE=1` for a backtrace.
```

The line `index out of bounds: the len is 1 but the index is 1` is an error
message intended for programmers. It won’t help our end users understand what
happened and what they should do instead. Let’s fix that now.

#### Improving the Error Message

In Listing 12-8, we add a check in the `new` function that will verify that the
slice is long enough before accessing index `1` and `2`. If the slice isn’t
long enough, the program panics and displays a better error message than the
`index out of bounds` message:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
// --snip--
fn new(args: &[String]) -> Config {
    if args.len() < 3 {
        panic!("not enough arguments");
    }
    // --snip--
```

<span class="caption">Listing 12-8: Adding a check for the number of
arguments</span>

This code is similar to the `Guess::new` function we wrote in Listing 9-9 where
we called `panic!` when the `value` argument was out of the range of valid
values. Instead of checking for a range of values here, we’re checking that the
length of `args` is at least `3` and the rest of the function can operate under
the assumption that this condition has been met. If `args` has fewer than three
items, this condition will be true, and we call the `panic!` macro to end the
program immediately.

With these extra few lines of code in `new`, let’s run the program without any
arguments again to see what the error looks like now:

```text
$ cargo run
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/minigrep`
thread 'main' panicked at 'not enough arguments', src/main.rs:30:12
note: Run with `RUST_BACKTRACE=1` for a backtrace.
```

This output is better: we now have a reasonable error message. However, we also
have extraneous information we don’t want to give to our users. Perhaps using
the technique we used in Listing 9-9 isn’t the best to use here: a call to
`panic!` is more appropriate for a programming problem rather than a usage
problem, as discussed in Chapter 9. Instead, we can use the other technique you
learned about in Chapter 9—returning a `Result` that indicates either success
or an error.

#### Returning a `Result` from `new` Instead of Calling `panic!`

We can instead return a `Result` value that will contain a `Config` instance in
the successful case and will describe the problem in the error case. When
`Config::new` is communicating to `main`, we can use the `Result` type to
signal there was a problem. Then we can change `main` to convert an `Err`
variant into a more practical error for our users without the surrounding text
about `thread 'main'` and `RUST_BACKTRACE` that a call to `panic!` causes.

Listing 12-9 shows the changes we need to make to the return value of
`Config::new` and the body of the function needed to return a `Result`. Note
that this won’t compile until we update `main` as well, which we’ll do in the
next listing:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
impl Config {
    fn new(args: &[String]) -> Result<Config, &'static str> {
        if args.len() < 3 {
            return Err("not enough arguments");
        }

        let query = args[1].clone();
        let filename = args[2].clone();

        Ok(Config { query, filename })
    }
}
```

<span class="caption">Listing 12-9: Returning a `Result` from
`Config::new`</span>

Our `new` function now returns a `Result` with a `Config` instance in the
success case and a `&'static str` in the error case. Recall from “The Static
Lifetime” section in Chapter 10 that `&'static str` is the type of string
literals, which is our error message type for now.

We’ve made two changes in the body of the `new` function: instead of calling
`panic!` when the user doesn’t pass enough arguments, we now return an `Err`
value, and we’ve wrapped the `Config` return value in an `Ok`. These changes
make the function conform to its new type signature.

Returning an `Err` value from `Config::new` allows the `main` function to
handle the `Result` value returned from the `new` function and exit the process
more cleanly in the error case.

#### Calling `Config::new` and Handling Errors

To handle the error case and print a user-friendly message, we need to update
`main` to handle the `Result` being returned by `Config::new`, as shown in
Listing 12-10. We’ll also take the responsibility of exiting the command line
tool with a nonzero error code from `panic!` and implement it by hand. A
nonzero exit status is a convention to signal to the process that called our
program that the program exited with an error state.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
use std::process;

fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::new(&args).unwrap_or_else(|err| {
        println!("Problem parsing arguments: {}", err);
        process::exit(1);
    });

    // --snip--
```

<span class="caption">Listing 12-10: Exiting with an error code if creating a
new `Config` fails</span>

In this listing, we’ve used a method we haven’t covered before:
`unwrap_or_else`, which is defined on `Result<T, E>` by the standard library.
Using `unwrap_or_else` allows us to define some custom, non-`panic!` error
handling. If the `Result` is an `Ok` value, this method’s behavior is similar
a `unwrap`: devuelve el valor interno `Ok` que está envolviendo. Sin embargo, si el valor
es un valor `Err`, este método llama al código en el *cierre*, que es una 
función anónima que definimos y pasamos como argumento a `unwrap_or_else`. Cubriremos los
cierres con más detalle en el capítulo 13. Por ahora, sólo necesitas saber que
`unwrap_or_else` pasará el valor interno del `Err`, que en este caso es la cadena 
estática `not enought arguments` que añadimos en el Listado 12-9, a nuestro
cierre en el argumento `err` que aparece entre los tubos verticales. El código
en el cierre puede entonces usar el valor `err` cuando se ejecuta.

Hemos añadido una nueva línea de `use` para importar `process` de la biblioteca estándar. El
código del cierre que se ejecutará en caso de error sólo tiene dos líneas: imprimimos
el valor `err` y luego llamamos `proceso::exit`. La función `process::exit`
detendrá inmediatamente el programa y devolverá el número pasado como código
de estado de salida. Esto es similar al manejo basado en `panic!` que usamos
en Listado 12-8, pero ya no obtenemos toda la salida extra.
Intentémoslo:

```text
$ cargo run
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.48 secs
     Running `target/debug/minigrep`
Problem parsing arguments: not enough arguments
```

¡Grandioso! Este resultado es mucho más amigable para nuestros usuarios.

### Extrayendo la lógica de `main`

Ahora que hemos terminado de refactorizar la configuración, volvamos a la
lógica del programa. Como dijimos en "Separación de Problemas para Proyectos 
Binarios", extraeremos una función llamada `run` que contendrá toda la lógica
actualmente en la función `main` que no está involucrada en la configuración
o manejo de errores. Cuando terminemos, `main` será conciso y fácil de 
verificar por inspección, y seremos capaces de escribir pruebas para toda la
otra lógica.

El listado 12-11 muestra la función `Run` extraída. Por ahora, sólo estamos haciendo
la pequeña, mejora incremental de extraer la función. Seguimos definiendo
la función en *src/main.rs*:

<span class="filename">Nombre del Archivo: src/main.rs</span>

```rust,ignore
fn main() {
    // --snip--

    println!("Searching for {}", config.query);
    println!("In file {}", config.filename);

    run(config);
}

fn run(config: Config) {
    let mut f = File::open(config.filename).expect("file not found");

    let mut contents = String::new();
    f.read_to_string(&mut contents)
        .expect("something went wrong reading the file");

    println!("With text:\n{}", contents);
}

// --snip--
```

<span class="caption">Listado 12-11: Extracción de una función `run` que contiene el 
resto de la lógica del programa</span>

La función `run` ahora contiene toda la lógica restante de `main`, empezando por
leer el archivo. La función `run` toma la instancia `Config` como un
argumento.

#### Devolución de Errores de la Función `run`

Con la lógica del programa restante separada en la función `run`, podemos
mejorar el manejo de errores, como lo hicimos con `Config::new` en el Listado 12-9.
En lugar de permitir que el programa entre en pánico llamando a `expect`, la función
`run` devuelve un `Result<T, E>` cuando algo sale mal. Esto nos permitirá consolidar
aún más en `main` la lógica de manejar errores de una forma amigable.
El listado 12-12 muestra los cambios que necesitamos hacer en la
firma y cuerpo de `run`:

<span class="filename">Nombre del Archivo: src/main.rs</span>

```rust,ignore
use std::error::Error;

// --snip--

fn run(config: Config) -> Result<(), Box<Error>> {
    let mut f = File::open(config.filename)?;

    let mut contents = String::new();
    f.read_to_string(&mut contents)?;

    println!("With text:\n{}", contents);

    Ok(())
}
```

<span class="caption">Listado 12-12: Cambiar la función `run` para devolver 
`Result`.</span>

Hemos hecho tres cambios significativos aquí. Primero, cambiamos el tipo de retorno de 
la función `run` por `Result<(), Box<Error>>`. Esta función anteriormente 
devolvía el tipo de unidad, `()`, y lo mantenemos como el valor devuelto en el
caso `Ok`.

Para el tipo de error, usamos el *trait object* `Box<Error>` (y hemos traído 
`std::error::Error` al alcance con una declaración `use` en la parte superior). Cubriremos
objetos de rasgos en el capítulo 17. Por ahora, sólo debes saber que `Box<Error>` significa que
la función devolverá un tipo que implementa el rasgo `Error`, pero no tenemos que 
especificar qué tipo particular será el valor de retorno. Esto nos da flexibilidad
para devolver valores de error que pueden ser de diferentes tipos en diferentes
casos de error.

Segundo, hemos quitado las llamadas a `expect` a favor de `?`, como hablamos en 
el Capítulo 9. En lugar de `panic!` en un error, `?` devolverá el valor de error 
de la función actual para que la persona que lo llama lo maneje.

En tercer lugar, la función `run` ahora devuelve un valor `Ok` en caso de éxito. Hemos 
declarado el tipo de éxito de la función `run` como `()` en la firma, lo que significa
que necesitamos envolver el valor del tipo de unidad en el valor `Ok`. Esta sintaxis
`Ok(())` puede parecer un poco extraña al principio, pero usar `()` como este es la
manera idiomática de indicar que estamos llamando `run` por sus efectos secundarios;
no devuelve un valor que necesitamos.

Cuando ejecutes este código, compilará pero mostrará una advertencia:

```text
warning: unused `std::result::Result` which must be used
  --> src/main.rs:18:5
   |
18 |     run(config);
   |     ^^^^^^^^^^^^
= note: #[warn(unused_must_use)] on by default
```

Rust nos dice que nuestro código ignoró el valor `Result`, y el valor `Result` 
podría indicar que ocurrió un error. Pero no estamos comprobando si hubo o 
no un error, y el compilador nos recuerda que probablemente quisiéramos 
tener algún código de manejo de errores aquí! Vamos a rectificar ese problema ahora.

#### Manejo de Errores Devueltos por `run` en` main`

Verificaremos errores y los manejaremos usando una técnica similar a la forma en que
manejamos errores con `Config::new` en el Listado 12-10, pero con una ligera
diferencia:

<span class="filename">Nombre del archivo: src/main.rs</span>

```rust,ignore
fn main() {
    // --snip--

    println!("Searching for {}", config.query);
    println!("In file {}", config.filename);

    if let Err(e) = run(config) {
        println!("Application error: {}", e);

        process::exit(1);
    }
}
```

Usamos `if let` en lugar de `unwrap_o_else` para comprobar si `run` devuelve un 
valor `Err` y llama a `process::exit(1)` si es así. La función `run` no devuelve
un valor que queremos `unwrap` de la misma manera que `Config::new`
devuelve la instancia `Config`. Debido a que en caso de éxito `run` devuelve `()`,
sólo nos importa detectar un error, por lo que no necesitamos `unwrap_or_else` para
devolver el valor desenvuelto porque sólo sería `()`.

Los cuerpos de las funciones `if let` y `unwrap_or_else` son los mismos en 
ambos casos: imprimimos el error y salimos.

### División de Código en una Crate de Biblioteca

Nuestro proyecto `minigrep` se ve bien hasta ahora! Ahora dividiremos el
archivo *src/main.rs* y pondremos algún código en el archivo *src/lib.rs* para que podamos
probarlo y tener un archivo *src/main.rs* con menos responsabilidades.

Movamos todo el código que no es la función `main` de *src/main.rs* a 
*src/lib.rs*:

* La definición de la función `run`
* Las declaraciones de `use` relevantes
* La definición de `Config`.
* La definición de la función `Config::new`

El contenido de *src/lib.rs* debe tener las firmas mostradas en el Listado 12-13
(hemos omitido los cuerpos de las funciones por brevedad). Ten en cuenta que esto no 
se compilará hasta que modifiquemos *src/main.rs* en el listado después de este:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
use std::error::Error;
use std::fs::File;
use std::io::prelude::*;

pub struct Config {
    pub query: String,
    pub filename: String,
}

impl Config {
    pub fn new(args: &[String]) -> Result<Config, &'static str> {
        // --snip--
    }
}

pub fn run(config: Config) -> Result<(), Box<Error>> {
    // --snip--
}
```

<span class="caption">Listing 12-13: Moving `Config` and `run` into
*src/lib.rs*</span>

We’ve made liberal use of `pub` here: on `Config`, its fields and its `new`
method, and on the `run` function. We now have a library crate that has a
public API that we can test!

Now we need to bring the code we moved to *src/lib.rs* into the scope of the
binary crate in *src/main.rs*, as shown in Listing 12-14:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
extern crate minigrep;

use std::env;
use std::process;

use minigrep::Config;

fn main() {
    // --snip--
    if let Err(e) = minigrep::run(config) {
        // --snip--
    }
}
```

<span class="caption">Listing 12-14: Bringing the `minigrep` crate into the
scope of *src/main.rs*</span>

To bring the library crate into the binary crate, we use `extern crate
minigrep`. Then we’ll add a `use minigrep::Config` line to bring the `Config`
type into scope, and we’ll prefix the `run` function with our crate name. Now
all the functionality should be connected and should work. Run the program with
`cargo run` and make sure everything works correctly.

Whew! That was a lot of work, but we’ve set ourselves up for success in the
future. Now it’s much easier to handle errors, and we’ve made the code more
modular. Almost all of our work will be done in *src/lib.rs* from here on out.

Let’s take advantage of this newfound modularity by doing something that would
have been difficult with the old code but is easy with the new code: we’ll
write some tests!
