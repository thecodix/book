## Errores recuperables con 'result'

La mayoría de los errores no son lo suficientemente graves como para requerir
que el programa se detenga por completo. A veces, cuando una función falla, es
por una razón que podemos interpretar y responder fácilmente. Por ejemplo, si 
tratamos de abrir un archivo y esa operación falla porque el archivo no existe,
es posible que deseemos crear el archivo en lugar de terminar el proceso.

Recordar de “[Handling Potential Failure with the `Result`
Type][handle_failure]<!-- ignore -->” en el Capítulo 2, que la enumeración
`Result` se define como que tiene dos variantes,` Ok` y `Err`, de la siguiente 
manera:

[handle_failure]: ch02-00-guessing-game-tutorial.html#handling-potential-failure-with-the-result-type

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

El `T` y` E` son parámetros de tipo genérico: discutiremos los genéricos con 
más detalle en el Capítulo 10. Lo que necesita saber ahora es que `T` 
representa el tipo del valor que se devolverá en un éxito caso dentro de la 
variante `Ok`, y` E` representa el tipo del error que se devolverá en un caso 
de falla dentro de la variante `Err`. Como `Result` tiene estos parámetros de 
tipo genérico, podemos utilizar el tipo `Result` y las funciones que la
biblioteca estándar ha definido en él en muchas situaciones diferentes en las 
que el valor correcto y el valor de error que queremos devolver pueden diferir.

Llamemos a una función que devuelve un valor `Result` porque la función podría 
fallar: en el Listado 9-3 intentamos abrir un archivo:

<span class="filename">Nombre del archivo: src/main.rs</span>

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");
}
```

<span class="caption">Listado 9-3: Apertura de un archivo</span>

¿Cómo sabemos que `File::open` devuelve un` Result`? ¡Podríamos mirar la 
documentación estándar de la API de la biblioteca, o podríamos preguntarle al 
compilador! Si le damos a `f` una anotación de tipo de un tipo que sabemos que 
el tipo de retorno de la función *no* es y luego tratamos de compilar el código,
el compilador nos dirá que los tipos no coinciden. El mensaje de error nos dirá
qué tipo de `f` *es*. Probémoslo: sabemos que el tipo de devolución de `File::open`
no es del tipo` u32`, así que cambiemos la declaración `let f` a esto:

```rust,ignore
let f: u32 = File::open("hello.txt");
```

Intentar compilar ahora nos da el siguiente resultado:

```text
error[E0308]: mismatched types
 --> src/main.rs:4:18
  |
4 |     let f: u32 = File::open("hello.txt");
  |                  ^^^^^^^^^^^^^^^^^^^^^^^ expected u32, found enum
`std::result::Result`
  |
  = note: expected type `u32`
             found type `std::result::Result<std::fs::File, std::io::Error>`
```

Esto nos dice que el tipo de retorno de la función `File::open` es `Result<T,E>`.
El parámetro genérico `T` se ha rellenado aquí con el tipo de valor de éxito, 
`std::fs::File`, que es un manejador de archivo. El tipo de `E` utilizado en el 
valor de error es `std::io::Error`.

Este tipo de devolución significa que la llamada a `File::open` puede tener 
éxito y nos devuelve un manejador de archivo desde el que podemos leer o escribir.
La llamada a la función también puede fallar: por ejemplo, es posible que el 
archivo no exista o que no tengamos permiso para acceder al archivo. La función 
`File::open` necesita tener una forma de decirnos si tuvo éxito o no, y al mismo
tiempo proporcionarnos el identificador del archivo o la información del error. 
Esta información es exactamente lo que transmite el enum de `Results`.

En el caso de que `File::open` tenga éxito, el valor que tendremos en la 
variable `f` será una instancia de `Ok` que contiene un manejador de archivo. En
el caso donde falla, el valor en `f` será una instancia de` Err` que contiene más
información sobre el tipo de error que ocurrió.

Necesitamos agregar al código en el Listado 9-3 para tomar diferentes acciones 
dependiendo del valor `File::open` devuelto. El Listado 9-4 muestra una forma de
manejar el `Result` utilizando una herramienta básica: la expresión` match` que 
analizamos en el Capítulo 6.

<span class="filename">Filename: src/main.rs</span>

```rust,should_panic
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => {
            panic!("There was a problem opening the file: {:?}", error)
        },
    };
}
```

<span class="caption">Listado 9-4: Usando una expresión `match` para manejar las
variantes` Result` podríamos tener</span>

Tenga en cuenta que, al igual que la enumeración `Option`, el enum `Result` y 
sus variantes se han importado en el preludio, por lo que no es necesario que
especifique `Result::` antes de las variantes` Ok` y `Err` en el `emparejar` los 
brazos.

Aquí le decimos a Rust que cuando el resultado es `Ok`, devuelve el valor `file`
interno de la variante `Ok`, y luego asignamos ese valor de manejo de archivo a
la variable `f`. Después del `match`, podemos usar el manejador del archivo para
leer o escribir.

El otro brazo del `match` maneja el caso donde obtenemos un valor `Err` de 
`File::open`. En este ejemplo, hemos elegido llamar a la macro `panic!`. Si no 
hay un archivo llamado *hello.txt* en nuestro directorio actual y ejecutamos 
este código, veremos el siguiente resultado de la macro `panic!`:

```text
thread 'main' panicked at 'There was a problem opening the file: Error { repr:
Os { code: 2, message: "No such file or directory" } }', src/main.rs:9:12
```

As usual, this output tells us exactly what has gone wrong.

### Matching on Different Errors

The code in Listing 9-4 will `panic!` no matter the reason that `File::open`
failed. What we want to do instead is take different actions for different
failure reasons: if `File::open` failed because the file doesn’t exist, we want
to create the file and return the handle to the new file. If `File::open`
failed for any other reason, for example because we didn’t have permission to
open the file, we still want the code to `panic!` in the same way as it did in
Listing 9-4. Look at Listing 9-5, which adds another arm to the `match`:

<span class="filename">Filename: src/main.rs</span>

<!-- ignore this test because otherwise it creates hello.txt which causes other
tests to fail lol -->

```rust,ignore
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(ref error) if error.kind() == ErrorKind::NotFound => {
            match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => {
                    panic!(
                        "Tried to create file but there was a problem: {:?}",
                        e
                    )
                },
            }
        },
        Err(error) => {
            panic!(
                "There was a problem opening the file: {:?}",
                error
            )
        },
    };
}
```

<span class="caption">Listing 9-5: Handling different kinds of errors in
different ways</span>

The type of the value that `File::open` returns inside the `Err` variant is
`io::Error`, which is a struct provided by the standard library. This struct
has a method `kind` that we can call to get an `io::ErrorKind` value.
`io::ErrorKind` is an enum provided by the standard library that has variants
representing the different kinds of errors that might result from an `io`
operation. The variant we want to use is `ErrorKind::NotFound`, which indicates
the file we’re trying to open doesn’t exist yet.

The condition `if error.kind() == ErrorKind::NotFound` is called a *match
guard*: it’s an extra condition on a `match` arm that further refines the arm’s
pattern. This condition must be true for that arm’s code to be run; otherwise,
the pattern matching will move on to consider the next arm in the `match`. The
`ref` in the pattern is needed so `error` is not moved into the guard condition
but is merely referenced by it. The reason `ref` is used to take a reference in
a pattern instead of `&` will be covered in detail in Chapter 18. In short, in
the context of a pattern, `&` matches a reference and gives us its value, but
`ref` matches a value and gives us a reference to it.

The condition we want to check in the match guard is whether the value returned
by `error.kind()` is the `NotFound` variant of the `ErrorKind` enum. If it is,
we try to create the file with `File::create`. However, because `File::create`
could also fail, we need to add an inner `match` statement as well. When the
file can’t be opened, a different error message will be printed. The last arm
of the outer `match` stays the same so the program panics on any error besides
the missing file error.

### Shortcuts for Panic on Error: `unwrap` and `expect`

Using `match` works well enough, but it can be a bit verbose and doesn’t always
communicate intent well. The `Result<T, E>` type has many helper methods
defined on it to do various tasks. One of those methods, called `unwrap`, is a
shortcut method that is implemented just like the `match` statement we wrote in
Listing 9-4. If the `Result` value is the `Ok` variant, `unwrap` will return
the value inside the `Ok`. If the `Result` is the `Err` variant, `unwrap` will
call the `panic!` macro for us. Here is an example of `unwrap` in action:

<span class="filename">Filename: src/main.rs</span>

```rust,should_panic
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").unwrap();
}
```

If we run this code without a *hello.txt* file, we’ll see an error message from
the `panic!` call that the `unwrap` method makes:

```text
thread 'main' panicked at 'called `Result::unwrap()` on an `Err` value: Error {
repr: Os { code: 2, message: "No such file or directory" } }',
src/libcore/result.rs:906:4
```

Another method, `expect`, which is similar to `unwrap`, lets us also choose the
`panic!` error message. Using `expect` instead of `unwrap` and providing good
error messages can convey your intent and make tracking down the source of a
panic easier. The syntax of `expect` looks like this:

<span class="filename">Filename: src/main.rs</span>

```rust,should_panic
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").expect("Failed to open hello.txt");
}
```

We use `expect` in the same way as `unwrap`: to return the file handle or call
the `panic!` macro. The error message used by `expect` in its call to `panic!`
will be the parameter that we pass to `expect`, rather than the default
`panic!` message that `unwrap` uses. Here’s what it looks like:

```text
thread 'main' panicked at 'Failed to open hello.txt: Error { repr: Os { code:
2, message: "No such file or directory" } }', src/libcore/result.rs:906:4
```

Because this error message starts with the text we specified, `Failed to open
hello.txt`, it will be easier to find where in the code this error message is
coming from. If we use `unwrap` in multiple places, it can take more time to
figure out exactly which `unwrap` is causing the panic because all `unwrap`
calls that panic print the same message.

### Propagating Errors

When you’re writing a function whose implementation calls something that might
fail, instead of handling the error within this function, you can return the
error to the calling code so that it can decide what to do. This is known as
*propagating* the error and gives more control to the calling code where there
might be more information or logic that dictates how the error should be
handled than what you have available in the context of your code.

For example, Listing 9-6 shows a function that reads a username from a file. If
the file doesn’t exist or can’t be read, this function will return those errors
to the code that called this function:

<span class="filename">Filename: src/main.rs</span>

```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let f = File::open("hello.txt");

    let mut f = match f {
        Ok(file) => file,
        Err(e) => return Err(e),
    };

    let mut s = String::new();

    match f.read_to_string(&mut s) {
        Ok(_) => Ok(s),
        Err(e) => Err(e),
    }
}
```

<span class="caption">Listing 9-6: A function that returns errors to the
calling code using `match`</span>

Let’s look at the return type of the function first: `Result<String,
io::Error>`. This means the function is returning a value of the type
`Result<T, E>` where the generic parameter `T` has been filled in with the
concrete type `String`, and the generic type `E` has been filled in with the
concrete type `io::Error`. If this function succeeds without any problems, the
code that calls this function will receive an `Ok` value that holds a
`String`—the username that this function read from the file. If this function
encounters any problems, the code that calls this function will receive an
`Err` value that holds an instance of `io::Error` that contains more
information about what the problems were. We chose `io::Error` as the return
type of this function because that happens to be the type of the error value
returned from both of the operations we’re calling in this function’s body that
might fail: the `File::open` function and the `read_to_string` method.

The body of the function starts by calling the `File::open` function. Then we
handle the `Result` value returned with a `match` similar to the `match` in
Listing 9-4, only instead of calling `panic!` in the `Err` case, we return
early from this function and pass the error value from `File::open` back to the
calling code as this function’s error value. If `File::open` succeeds, we store
the file handle in the variable `f` and continue.

Then we create a new `String` in variable `s` and call the `read_to_string`
method on the file handle in `f` to read the contents of the file into `s`. The
`read_to_string` method also returns a `Result` because it might fail, even
though `File::open` succeeded. So we need another `match` to handle that
`Result`: if `read_to_string` succeeds, then our function has succeeded, and we
return the username from the file that’s now in `s` wrapped in an `Ok`. If
`read_to_string` fails, we return the error value in the same way that we
returned the error value in the `match` that handled the return value of
`File::open`. However, we don’t need to explicitly say `return`, because this
is the last expression in the function.

The code that calls this code will then handle getting either an `Ok` value
that contains a username or an `Err` value that contains an `io::Error`. We
don’t know what the calling code will do with those values. If the calling code
gets an `Err` value, it could call `panic!` and crash the program, use a
default username, or look up the username from somewhere other than a file, for
example. We don’t have enough information on what the calling code is actually
trying to do, so we propagate all the success or error information upwards for
it to handle appropriately.

This pattern of propagating errors is so common in Rust that Rust provides the
question mark operator `?` to make this easier.

#### A Shortcut for Propagating Errors: `?`

Listing 9-7 shows an implementation of `read_username_from_file` that has the
same functionality as it had in Listing 9-6, but this implementation uses the
question mark operator:

<span class="filename">Filename: src/main.rs</span>

```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut f = File::open("hello.txt")?;
    let mut s = String::new();
    f.read_to_string(&mut s)?;
    Ok(s)
}
```

<span class="caption">Listing 9-7: A function that returns errors to the
calling code using `?`</span>

The `?` placed after a `Result` value is defined to work in almost the same way
as the `match` expressions we defined to handle the `Result` values in Listing
9-6. If the value of the `Result` is an `Ok`, the value inside the `Ok` will
get returned from this expression and the program will continue. If the value
is an `Err`, the value inside the `Err` will be returned from the whole
function as if we had used the `return` keyword so the error value gets
propagated to the calling code.

There is a difference between what the `match` expression from Listing 9-6 and
the question mark operator do: error values used with `?` go through the `from`
function, defined in the `From` trait in the standard library, which is used to
convert errors from one type into another. When the question mark calls the
`from` function, the error type received is converted into the error type
defined in the return type of the current function. This is useful when a
function returns one error type to represent all the ways a function might
fail, even if parts might fail for many different reasons. As long as each
error type implements the `from` function to define how to convert itself to
the returned error type, the question mark operator takes care of the
conversion automatically.

In the context of Listing 9-7, the `?` at the end of the `File::open` call will
return the value inside an `Ok` to the variable `f`. If an error occurs, `?`
will return early out of the whole function and give any `Err` value to the
calling code. The same thing applies to the `?` at the end of the
`read_to_string` call.

The `?` eliminates a lot of boilerplate and makes this function’s
implementation simpler. We could even shorten this code further by chaining
method calls immediately after the `?` as shown in Listing 9-8:

<span class="filename">Filename: src/main.rs</span>

```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut s = String::new();

    File::open("hello.txt")?.read_to_string(&mut s)?;

    Ok(s)
}
```

<span class="caption">Listing 9-8: Chaining method calls after the question
mark operator</span>

We’ve moved the creation of the new `String` in `s` to the beginning of the
function; that part hasn’t changed. Instead of creating a variable `f`, we’ve
chained the call to `read_to_string` directly onto the result of
`File::open("hello.txt")?`. We still have a `?` at the end of the
`read_to_string` call, and we still return an `Ok` value containing the
username in `s` when both `File::open` and `read_to_string` succeed rather than
returning errors. The functionality is again the same as in Listing 9-6 and
Listing 9-7; this is just a different, more ergonomic way to write it.

#### `?` Can Only Be Used in Functions That Return Result

The `?` can only be used in functions that have a return type of `Result`,
because it is defined to work in the same way as the `match` expression we
defined in Listing 9-6. The part of the `match` that requires a return type of
`Result` is `return Err(e)`, so the return type of the function must be a
`Result` to be compatible with this `return`.

Let’s look at what happens if we use `?` in the `main` function, which you’ll
recall has a return type of `()`:

```rust,ignore
use std::fs::File;

fn main() {
    let f = File::open("hello.txt")?;
}
```

When we compile this code, we get the following error message:

```text
error[E0277]: the trait bound `(): std::ops::Try` is not satisfied
 --> src/main.rs:4:13
  |
4 |     let f = File::open("hello.txt")?;
  |             ------------------------
  |             |
  |             the `?` operator can only be used in a function that returns
  `Result` (or another type that implements `std::ops::Try`)
  |             in this macro invocation
  |
  = help: the trait `std::ops::Try` is not implemented for `()`
  = note: required by `std::ops::Try::from_error`
```

This error points out that we’re only allowed to use the question mark operator
in a function that returns `Result`. In functions that don’t return `Result`,
when you call other functions that return `Result`, you’ll need to use a
`match` or one of the `Result` methods to handle it instead of using `?` to
potentially propagate the error to the calling code.

Now that we’ve discussed the details of calling `panic!` or returning `Result`,
let’s return to the topic of how to decide which is appropriate to use in which
cases.
