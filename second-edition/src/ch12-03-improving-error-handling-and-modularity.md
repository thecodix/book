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
* Mientras que tu lógica de análisis de línea de comandos es pequeña, puede permanecer en *main.rs*.
* Cuando la lógica de análisis de la línea de comandos empiece a complicarse, extrála de
*main.rs* y muevela a *lib.rs*.
* Las responsabilidades que permanecen en la función `main` después de este proceso
  deben limitarse a:
  
* Llamar a la lógica de análisis de línea de comandos con los valores del argumento
* Configuración de cualquier otra configuración
* Llamar a una función de `run` en *lib.rs*
* Manejar el error si `run` devuelve un error

Este patrón trata de separar los problemas: *main.rs* maneja la ejecución del 
programa, y *lib.rs* maneja toda la lógica de la tarea a mano. Debido a que no
podemos probar la función `main` directamente, esta estructura nos permite probar toda
la lógica de nuestro programa moviéndola a funciones en *lib.rs*. El único código que 
queda en *main.rs* será lo suficientemente pequeño como para verificar su exactitud al 
leerlo. Repasemos nuestro programa siguiendo este proceso.

#### Extracción del Analizador de Argumentos

Extraeremos la funcionalidad para analizar argumentos en una función
que `main` llamará a preparar para mover el análisis lógico de línea
de comandos a *src/lib.rs*. El listado 12-5 muestra el nuevo inicio de `main`
que llama a una nueva función `parse_config`, que definiremos en *src/main.rs* por el momento.

<span class="filename">Nombre del archivo: src/main.rs</span>

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

<span class="caption">Listado 12-5: Extracción de una función `parse_config` de 
`main`.</span>

Seguimos recopilando los argumentos de la línea de comandos en un vector, pero en lugar de
asignar el valor del argumento en el índice `1` a la variable `query` y el 
valor del argumento en el índice `2` a la variable `filename` dentro de la función
`main`, pasamos el vector entero a la función `parse_config`. La función `parse_config` 
mantiene entonces la lógica que determina qué argumento va en qué variable
y devuelve los valores a `main`. Seguimos creando las variables `query` 
y `filename` en `main`, pero `main` ya no tiene la responsabilidad de
determinar cómo se corresponden los argumentos y las variables de la 
línea de comandos.

Este retrabajo puede parecer exagerado para nuestro pequeño programa, pero estamos refactorizando
en pequeños pasos incrementales. Después de hacer este cambio, ejecuta el programa nuevamente para
verificar que el análisis de argumentos sigue funcionando. Es bueno revisar tu progreso con 
frecuencia, porque eso te ayudará a identificar la causa de los problemas cuando 
ocurran.

#### Clasificación de Valores de Configuración

Podemos dar otro pequeño paso para mejorar aún más la función `parse_config`.
Por el momento, estamos devolviendo una tupla, pero de inmediato la dividimos
en partes individuales. Esta es una señal de que quizás todavía no tenemos la
abstracción correcta.

Otro indicador que muestra que hay espacio para mejorar es la parte `config` 
de `parse_config`, lo que implica que los dos valores que devolvemos están relacionados
y forman parte de un valor de configuración. Actualmente no estamos transmitiendo este
significado en la estructura de los datos más que agrupando los dos valores en una 
tupla: podríamos poner los dos valores en una sola estructura y darle a cada uno de los campos
de la estructura un nombre significativo. Hacerlo así facilitará a los futuros mantenedores
de este código entender cómo se relacionan los diferentes valores entre sí y 
cuál es su propósito.

> Nota: Algunas personas llaman a este anti-patrón de usar valores primitivos cuando un 
> tipo complejo sería más apropiado *obsesión primitiva*.

El listado 12-6 muestra la adición de una estructura llamada `Config` definida para tener 
campos con el nombre `query` y `filename`. También hemos cambiado la función `parse_config` 
para devolver una instancia de la `Config` estructurada y actualizada `main` para usar los
campos de estructura en lugar de tener variables separadas:

<span class="filename">Nombre del archivo: src/main.rs</span>

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

<span class="caption">Listado 12-6: Refactorizando `parse_config` para devolver 
una instancia de una estructura `Config`.</span>

La firma de `parse_config` ahora indica que devuelve un valor `Config`.
En el cuerpo de `parse_config`, donde solíamos devolver las slices de cadena que
hacen referencia a los valores `String` en `args`, ahora definimos `Config` para contener 
los valores propios de `String`. La variable `args` en `main` es la dueña de los valores 
del argumento y sólo deja que la función `parse_config` los tome prestados, lo que significa 
que violaríamos las reglas de préstamo de Rust si `Config` intentara tomar posesión de los
valores en `args`.

Podríamos manejar los datos de `String` de varias maneras diferentes, pero la
ruta más fácil, aunque algo ineficiente, es llamar al método `clone` en los
valores. Esto hará una copia completa de los datos para que la instancia `Config` los 
posea, lo que toma más tiempo y memoria que almacenar una referencia a los datos de
la cadena. Sin embargo, la clonación de los datos también hace que nuestro código sea
muy sencillo, porque no tenemos que gestionar la vida útil de las referencias; en esta
circunstancia, renunciar a un poco de rendimiento para ganar simplicidad es un compromiso
que vale la pena.

> ### Las Ventajas y Desventajas de Usar `clone`.
>
> Hay una tendencia entre muchos rustaceos a evitar el uso de `clone` para arreglar
> problemas de posesión debido a su costo de tiempo de ejecución. En el Capítulo 13, aprenderás
> a usar métodos más eficientes en este tipo de situaciones. Pero por ahora, está bien
> copiar unas cuantas cadenas para seguir progresando porque vamos a hacer estas
> copias sólo una vez, y nuestro nombre de archivo y la cadena de consulta son muy 
> pequeños. Es mejor tener un programa de trabajo que es un poco ineficiente que tratar
> de hiperoptimizar el código en tu primera pasada. A medida que tengas más experiencia 
> con Rust, será más fácil empezar con la solución más eficiente, pero por ahora, es 
> perfectamente aceptable llamar a `clone`.

Hemos actualizado `main` para que coloque la instancia de `Config` devuelta por
`parse_config` en una variable llamada `config`, y hemos actualizado el código que
antes utilizaba las variables `query` y `filename` separadas, por lo que ahora usa
los campos de la estructura `Config`.

Ahora nuestro código transmite con más claridad que `query` y `filename` están relacionados, 
y su propósito es configurar cómo funcionará el programa. Cualquier código que utilice 
estos valores sabe que los encontrará en la instancia `config` en los campos nombrados 
para su propósito.

#### Creación de un Constructor para `Config`.

Hasta ahora, hemos extraído la lógica responsable de analizar los argumentos 
de la línea de comandos de `main` y la hemos colocado en la función `parse_config`, que
nos ayudó a ver que los valores `query` y `filename` estaban relacionados y que la 
relación debería ser transmitida en nuestro código. Luego añadimos una estructura `Config`
para nombrar el propósito relacionado de `query` y `filename`, y para poder devolver los
nombres de los valores como nombres de campo de estructura desde la función `parse_config`.

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
to `unwrap`: it returns the inner value `Ok` is wrapping. However, if the value
is an `Err` value, this method calls the code in the *closure*, which is an
anonymous function we define and pass as an argument to `unwrap_or_else`. We’ll
cover closures in more detail in Chapter 13. For now, you just need to know
that `unwrap_or_else` will pass the inner value of the `Err`, which in this
case is the static string `not enough arguments` that we added in Listing 12-9,
to our closure in the argument `err` that appears between the vertical pipes.
The code in the closure can then use the `err` value when it runs.

We’ve added a new `use` line to import `process` from the standard library. The
code in the closure that will be run in the error case is only two lines: we
print the `err` value and then call `process::exit`. The `process::exit`
function will stop the program immediately and return the number that was
passed as the exit status code. This is similar to the `panic!`-based handling
we used in Listing 12-8, but we no longer get all the extra output. Let’s try
it:

```text
$ cargo run
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.48 secs
     Running `target/debug/minigrep`
Problem parsing arguments: not enough arguments
```

Great! This output is much friendlier for our users.

### Extracting Logic from `main`

Now that we’ve finished refactoring the configuration parsing, let’s turn to
the program’s logic. As we stated in “Separation of Concerns for Binary
Projects”, we’ll extract a function named `run` that will hold all the logic
currently in the `main` function that isn’t involved with setting up
configuration or handling errors. When we’re done, `main` will be concise and
easy to verify by inspection, and we’ll be able to write tests for all the
other logic.

Listing 12-11 shows the extracted `run` function. For now, we’re just making
the small, incremental improvement of extracting the function. We’re still
defining the function in *src/main.rs*:

<span class="filename">Filename: src/main.rs</span>

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

<span class="caption">Listing 12-11: Extracting a `run` function containing the
rest of the program logic</span>

The `run` function now contains all the remaining logic from `main`, starting
from reading the file. The `run` function takes the `Config` instance as an
argument.

#### Returning Errors from the `run` Function

With the remaining program logic separated into the `run` function, we can
improve the error handling, as we did with `Config::new` in Listing 12-9.
Instead of allowing the program to panic by calling `expect`, the `run`
function will return a `Result<T, E>` when something goes wrong. This will let
us further consolidate into `main` the logic around handling errors in a
user-friendly way. Listing 12-12 shows the changes we need to make to the
signature and body of `run`:

<span class="filename">Filename: src/main.rs</span>

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

<span class="caption">Listing 12-12: Changing the `run` function to return
`Result`</span>

We’ve made three significant changes here. First, we changed the return type of
the `run` function to `Result<(), Box<Error>>`. This function previously
returned the unit type, `()`, and we keep that as the value returned in the
`Ok` case.

For the error type, we used the *trait object* `Box<Error>` (and we’ve brought
`std::error::Error` into scope with a `use` statement at the top). We’ll cover
trait objects in Chapter 17. For now, just know that `Box<Error>` means the
function will return a type that implements the `Error` trait, but we don’t
have to specify what particular type the return value will be. This gives us
flexibility to return error values that may be of different types in different
error cases.

Second, we’ve removed the calls to `expect` in favor of `?`, as we talked about
in Chapter 9. Rather than `panic!` on an error, `?` will return the error value
from the current function for the caller to handle.

Third, the `run` function now returns an `Ok` value in the success case. We’ve
declared the `run` function’s success type as `()` in the signature, which
means we need to wrap the unit type value in the `Ok` value. This `Ok(())`
syntax might look a bit strange at first, but using `()` like this is the
idiomatic way to indicate that we’re calling `run` for its side effects only;
it doesn’t return a value we need.

When you run this code, it will compile but will display a warning:

```text
warning: unused `std::result::Result` which must be used
  --> src/main.rs:18:5
   |
18 |     run(config);
   |     ^^^^^^^^^^^^
= note: #[warn(unused_must_use)] on by default
```

Rust tells us that our code ignored the `Result` value, and the `Result` value
might indicate that an error occurred. But we’re not checking to see whether or
not there was an error, and the compiler reminds us that we probably meant to
have some error handling code here! Let’s rectify that problem now.

#### Handling Errors Returned from `run` in `main`

We’ll check for errors and handle them using a technique similar to the way we
handled errors with `Config::new` in Listing 12-10, but with a slight
difference:

<span class="filename">Filename: src/main.rs</span>

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

We use `if let` rather than `unwrap_or_else` to check whether `run` returns an
`Err` value and call `process::exit(1)` if it does. The `run` function doesn’t
return a value that we want to `unwrap` in the same way that `Config::new`
returns the `Config` instance. Because `run` returns `()` in the success case,
we only care about detecting an error, so we don’t need `unwrap_or_else` to
return the unwrapped value because it would only be `()`.

The bodies of the `if let` and the `unwrap_or_else` functions are the same in
both cases: we print the error and exit.

### Splitting Code into a Library Crate

Our `minigrep` project is looking good so far! Now we’ll split the
*src/main.rs* file and put some code into the *src/lib.rs* file so we can test
it and have a *src/main.rs* file with fewer responsibilities.

Let’s move all the code that isn’t the `main` function from *src/main.rs* to
*src/lib.rs*:

* The `run` function definition
* The relevant `use` statements
* The definition of `Config`
* The `Config::new` function definition

The contents of *src/lib.rs* should have the signatures shown in Listing 12-13
(we’ve omitted the bodies of the functions for brevity). Note that this won't
compile until we modify *src/main.rs* in the listing after this one:

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
