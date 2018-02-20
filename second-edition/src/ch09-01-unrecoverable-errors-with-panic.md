## Errores irrecuperables con `panic!`

Algunas veces suceden cosas malas en su código, y no hay nada que pueda hacer
al respecto. En estos casos, Rust tiene la macro `panic!`. Cuando se ejecuta la
macro `panic!`, Su programa imprimirá un mensaje de error, desenrollará y 
limpiará la pila, y luego saldrá. La situación más común en la que ocurre esto
es cuando se detecta un error de algún  tipo, y el programador no tiene claro 
cómo manejar el error.

> ### ¡Desenrollar la pila o abortar en respuesta a un `panic!`
>
> De forma predeterminada, cuando se produce un `panic!`, el programa comienza
> *desenrollarse*, lo que significa que Rust vuelve a subir la pila y limpia los
> datos de cada función que encuentra. Pero este caminar de regreso y limpiar
> es mucho trabajo. La alternativa es *abort* inmediatamente, que finaliza el 
> programa sin limpiar. La memoria que el  programa estaba usando tendrá que 
> ser limpiada por el sistema operativo. Si en su proyecto necesita hacer que el
> binario resultante sea lo más pequeño posible, puede cambiar de desenrollar a
> abortar en pánico agregando `panic='abort'` a las secciones apropiadas 
> `[profile]` en su *Cargo.toml* archivo. Por ejemplo, si desea cancelar en   
> pánico en el modo de lanzamiento, agregue esto:
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

La llamada a `pánic!` Causa el mensaje de error contenido en las últimas tres 
líneas. La primera línea muestra nuestro mensaje de pánico y el lugar en 
nuestro código fuente donde ocurrió el pánico: *src/main.rs:2:4* indica que es
el segundo carácter de la segunda línea de nuestro archivo *src/main.rs*.

En este caso, la línea indicada es parte de nuestro código, y si vamos a esa 
línea, vemos la llamada a macro `panic!`. En otros casos, la llamada `panic!`
Podría estar en el código que llama nuestro código. El nombre de archivo y el
número de línea informados por el mensaje de error serán el código de otra 
persona donde se llama a la macro `panic!`, No la línea de nuestro código que
finalmente condujo a la llamada `panic!`. Podemos usar la traza inversa de las
funciones de las que salió la llamada `panic!` Para descubrir la parte de 
nuestro código que está causando el problema. Discutiremos lo que es una traza
inversa en más detalle a continuación.

### Usando un `panic!` Backtrace

Veamos otro ejemplo para ver cómo es cuando una llamada `panic!`Viene de una
biblioteca debido a un error en nuestro código en lugar de a un código que 
llama directamente a la macro. El listado 9-1 tiene algún código que intenta 
acceder a un elemento por índice en un vector:

<span class="filename">Filename: src/main.rs</span>

```rust,should_panic
fn main() {
    let v = vec![1, 2, 3];

    v[99];
}
```

<span class="caption">Listado 9-1: Intentando acceder a un elemento más allá 
del final de un vector, lo que provocará un `panic!``</span>

Aquí, estamos intentando acceder al centésimo elemento de nuestro vector (que
está en el índice 99 porque la indexación comienza en cero), pero solo tiene 
tres elementos. En esta situación, Rust entrará en pánico. Se supone que el uso
de  `[]` devuelve un elemento, pero si pasa un índice inválido, no hay ningún 
elemento que Rust pueda devolver aquí que sea  correcto.

Otros idiomas, como C, intentarán darte exactamente lo que pediste en esta 
situación, aunque no sea lo que quieres: obtendrás lo que esté en la ubicación
en la memoria que correspondería a ese elemento en el vector , aunque la memoria
no pertenece al vector. Esto se conoce como *overread* de búfer y puede generar
vulnerabilidades de seguridad si un atacante puede manipular el índice de forma
que pueda leer datos que no deberían almacenarse después de la matriz.

Para proteger su programa de este tipo de vulnerabilidad, si intenta leer un 
elemento en un índice que no existe, Rust detendrá la ejecución y se negará a 
continuar. Probemos y veamos:

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

Este error apunta a un archivo que no escribimos, *vec.rs*. Esa es la
implementación de `Vec<T>` en la biblioteca estándar. El código que se
ejecuta cuando usamos `[]` en nuestro vector `v` está en *vec.rs*, y ahí es 
donde `panic!` Está realmente sucediendo.

La siguiente línea de notas nos dice que podemos establecer la variable de 
entorno `RUST_BACKTRACE` para obtener un seguimiento de exactamente lo que 
sucedió para causar el error. A *backtrace* es una lista de todas las
funciones que se han llamado para llegar a este punto. Los backtraces en Rust
funcionan como lo hacen en otros idiomas: la clave para leer la traza inversa
es comenzar desde la parte superior y leer hasta que vea los archivos que 
escribió. Ese es el lugar donde se originó el problema. Las líneas sobre las 
líneas que mencionan sus archivos son códigos que su código llamó; las líneas
a continuación son código que llamó su código. Estas líneas pueden incluir el
código principal de Rust, el código de biblioteca estándar o las cajas que está
utilizando. Tratemos de obtener una traza inversa: el Listado 9-2 muestra
resultados similares a los que verá:

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

<span class="caption">Listado 9-2: La traza inversa generada por una llamada
a `panic!` Que se muestra cuando se establece la variable de entorno 
`RUST_BACKTRACE`</span>

¡Eso es mucho rendimiento! La salida exacta que ve puede ser diferente 
dependiendo de su sistema operativo y la versión de Rust. Para obtener 
retrocesos con esta información, los símbolos de depuración deben estar
habilitados. Los símbolos de depuración están habilitados de forma 
predeterminada cuando se utiliza la construcción de carga o la carga sin la 
bandera de liberación, como aquí.

En el resultado del Listado 9-2, la línea 11 del backtrace apunta a la línea de
nuestro proyecto que está causando el problema: *src/main.rs* en la línea 4. Si
no queremos que nuestro programa entre en pánico, la ubicación señalado por la 
primera línea que menciona un archivo que escribimos es donde deberíamos comenzar
a investigar para descubrir cómo llegamos a esta ubicación con los valores que 
causaron el pánico. En el listado 9-1 donde deliberadamente escribimos un código 
que entraría en pánico para demostrar cómo usar trazas inversas, la forma de 
solucionar el pánico es no solicitar un elemento en el índice 99 a partir de un
vector que solo contenga tres elementos. Cuando su código entre en pánico en el
futuro, tendrá que averiguar qué acción está tomando el código con qué valores 
causa el pánico y qué debe hacer el código en su lugar.

Volveremos a `Panic!` Y cuando deberíamos y no deberíamos usar `panic!` Para 
manejar las condiciones de error más adelante en el capítulo. A continuación,
veremos cómo recuperarse de un error usando `Result`.
