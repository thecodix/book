## Errores irrecuperables con `panic!`

Algunas veces suceden cosas malas en su código, y no hay nada que pueda hacer al respecto. En estos casos, Rust tiene
la macro `panic!`. Cuando se ejecuta la macro `panic!`, Su programa imprimirá un mensaje de error, desenrollará y 
limpiará la pila, y luego saldrá. La situación más común en la que ocurre esto es cuando se detecta un error de algún 
tipo, y el programador no tiene claro cómo manejar el error.

> ### ¡Desenrollar la pila o abortar en respuesta a un `panic!`
>
> De forma predeterminada, cuando se produce un `panic!`, el programa comienza *desenrollarse*, lo que significa que
Rust vuelve a subir la pila y limpia los datos de cada función que encuentra. Pero este caminar de regreso y limpiar 
es mucho trabajo. La alternativa es *abort* inmediatamente, que finaliza el programa sin limpiar. La memoria que el 
programa estaba usando tendrá que ser limpiada por el sistema operativo. Si en su proyecto necesita hacer que el binario
resultante sea lo más pequeño posible, puede cambiar de desenrollar a abortar en pánico agregando `panic='abort'` a las
secciones apropiadas `[profile]` en su *Cargo.toml* archivo. Por ejemplo, si desea cancelar en pánico en el modo de 
lanzamiento, agregue esto:
>
> ```toml
> [profile.release]
> panic = 'abort'
> ```

Tratemos de llamar "panic!" En un programa simple:

<span class="filename">Filename: src/main.rs</span>

```rust,should_panic
fn main() {
    panic!("crash and burn");
}
```

Cuando ejecutas el programa, verás algo como esto:

```text
$ cargo run
   Compiling panic v0.1.0 (file:///projects/panic)
    Finished dev [unoptimized + debuginfo] target(s) in 0.25 secs
     Running `target/debug/panic`
thread 'main' panicked at 'crash and burn', src/main.rs:2:4
note: Run with `RUST_BACKTRACE=1` for a backtrace.
```

La llamada a `pánic!` Causa el mensaje de error contenido en las últimas tres líneas. La primera línea muestra nuestro
mensaje de pánico y el lugar en nuestro código fuente donde ocurrió el pánico: *src/main.rs:2:4* indica que es el 
segundo carácter de la segunda línea de nuestro archivo *src/main.rs*.

En este caso, la línea indicada es parte de nuestro código, y si vamos a esa línea, vemos la llamada a macro `panic!`.
En otros casos, la llamada `panic!` Podría estar en el código que llama nuestro código. El nombre de archivo y el 
número de línea informados por el mensaje de error serán el código de otra persona donde se llama a la macro `panic!`,
No la línea de nuestro código que finalmente condujo a la llamada `panic!`. Podemos usar la traza inversa de las 
funciones de las que salió la llamada `panic!` Para descubrir la parte de nuestro código que está causando el problema.
Discutiremos lo que es una traza inversa en más detalle a continuación.

### Usando un `panic!` Backtrace

Veamos otro ejemplo para ver cómo es cuando una llamada `panic!`Viene de una biblioteca debido a un error en nuestro 
código en lugar de a un código que llama directamente a la macro. El listado 9-1 tiene algún código que intenta acceder
a un elemento por índice en un vector:

<span class="filename">Filename: src/main.rs</span>

```rust,should_panic
fn main() {
    let v = vec![1, 2, 3];

    v[99];
}
```

<span class="caption">Listado 9-1: Intentando acceder a un elemento más allá del final de un vector, lo que provocará
un `panic!``</span>

Aquí, estamos intentando acceder al centésimo elemento de nuestro vector (que está en el índice 99 porque la indexación
comienza en cero), pero solo tiene tres elementos. En esta situación, Rust entrará en pánico. Se supone que el uso de 
`[]` devuelve un elemento, pero si pasa un índice inválido, no hay ningún elemento que Rust pueda devolver aquí que sea 
correcto.

Otros idiomas, como C, intentarán darte exactamente lo que pediste en esta situación, aunque no sea lo que quieres: 
obtendrás lo que esté en la ubicación en la memoria que correspondería a ese elemento en el vector , aunque la memoria
no pertenece al vector. Esto se conoce como *overread* de búfer y puede generar vulnerabilidades de seguridad si un
atacante puede manipular el índice de forma que pueda leer datos que no deberían almacenarse después de la matriz.

Para proteger su programa de este tipo de vulnerabilidad, si intenta leer un elemento en un índice que no existe, Rust
detendrá la ejecución y se negará a continuar. Probemos y veamos:

```text
$ cargo run
   Compiling panic v0.1.0 (file:///projects/panic)
    Finished dev [unoptimized + debuginfo] target(s) in 0.27 secs
     Running `target/debug/panic`
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is
99', /checkout/src/liballoc/vec.rs:1555:10
note: Run with `RUST_BACKTRACE=1` for a backtrace.
error: Process didn't exit successfully: `target/debug/panic` (exit code: 101)
```

This error points at a file we didn’t write, *vec.rs*. That’s the
implementation of `Vec<T>` in the standard library. The code that gets run when
we use `[]` on our vector `v` is in *vec.rs*, and that is where the `panic!` is
actually happening.

The next note line tells us that we can set the `RUST_BACKTRACE` environment
variable to get a backtrace of exactly what happened to cause the error. A
*backtrace* is a list of all the functions that have been called to get to this
point. Backtraces in Rust work like they do in other languages: the key to
reading the backtrace is to start from the top and read until you see files you
wrote. That’s the spot where the problem originated. The lines above the lines
mentioning your files are code that your code called; the lines below are code
that called your code. These lines might include core Rust code, standard
library code, or crates that you’re using. Let’s try getting a backtrace:
Listing 9-2 shows output similar to what you’ll see:

```text
$ RUST_BACKTRACE=1 cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/panic`
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 99', /checkout/src/liballoc/vec.rs:1555:10
stack backtrace:
   0: std::sys::imp::backtrace::tracing::imp::unwind_backtrace
             at /checkout/src/libstd/sys/unix/backtrace/tracing/gcc_s.rs:49
   1: std::sys_common::backtrace::_print
             at /checkout/src/libstd/sys_common/backtrace.rs:71
   2: std::panicking::default_hook::{{closure}}
             at /checkout/src/libstd/sys_common/backtrace.rs:60
             at /checkout/src/libstd/panicking.rs:381
   3: std::panicking::default_hook
             at /checkout/src/libstd/panicking.rs:397
   4: std::panicking::rust_panic_with_hook
             at /checkout/src/libstd/panicking.rs:611
   5: std::panicking::begin_panic
             at /checkout/src/libstd/panicking.rs:572
   6: std::panicking::begin_panic_fmt
             at /checkout/src/libstd/panicking.rs:522
   7: rust_begin_unwind
             at /checkout/src/libstd/panicking.rs:498
   8: core::panicking::panic_fmt
             at /checkout/src/libcore/panicking.rs:71
   9: core::panicking::panic_bounds_check
             at /checkout/src/libcore/panicking.rs:58
  10: <alloc::vec::Vec<T> as core::ops::index::Index<usize>>::index
             at /checkout/src/liballoc/vec.rs:1555
  11: panic::main
             at src/main.rs:4
  12: __rust_maybe_catch_panic
             at /checkout/src/libpanic_unwind/lib.rs:99
  13: std::rt::lang_start
             at /checkout/src/libstd/panicking.rs:459
             at /checkout/src/libstd/panic.rs:361
             at /checkout/src/libstd/rt.rs:61
  14: main
  15: __libc_start_main
  16: <unknown>
```

<span class="caption">Listing 9-2: The backtrace generated by a call to
`panic!` displayed when the environment variable `RUST_BACKTRACE` is set</span>

That’s a lot of output! The exact output you see might be different depending
on your operating system and Rust version. In order to get backtraces with this
information, debug symbols must be enabled. Debug symbols are enabled by
default when using cargo build or cargo run without the --release flag, as we
have here.

In the output in Listing 9-2, line 11 of the backtrace points to the line in
our project that’s causing the problem: *src/main.rs* in line 4. If we don’t
want our program to panic, the location pointed to by the first line mentioning
a file we wrote is where we should start investigating to figure out how we got
to this location with values that caused the panic. In Listing 9-1 where we
deliberately wrote code that would panic in order to demonstrate how to use
backtraces, the way to fix the panic is to not request an element at index 99
from a vector that only contains three items. When your code panics in the
future, you’ll need to figure out what action the code is taking with what
values that causes the panic and what the code should do instead.

We’ll come back to `panic!` and when we should and should not use `panic!` to
handle error conditions later in the chapter. Next, we’ll look at how to
recover from an error using `Result`.
