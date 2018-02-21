## Usando Threads Para Ejecutar Código Simultáneamente

En la mayoría de los sistemas operativos actuales, el código de un programa se ejecuta en 
un *proceso*, y el sistema operativo gestiona varios procesos a la vez. Dentro de 
tu programa, también puedes tener partes independientes que se ejecutan simultáneamente. La 
característica que ejecuta estas partes independientes se llama *threads*.

Dividir el cálculo de tu programa en varios subprocesos puede mejorar el rendimiento
porque el programa realiza varias tareas al mismo tiempo, pero también añade
complejidad. Debido a que los threads pueden funcionar simultáneamente, no hay garantía
inherente sobre el orden en el que se ejecutarán las partes de tu código en diferentes
threads. Esto puede llevar a problemas como:

* Condiciones de carrera, donde los threads están accediendo a datos o recursos en un
  orden inconsistente.
* Bloqueos, donde dos threads se están esperando unos a otros para terminar usando un 
  recurso que tiene el otro thread, impidiendo que ambos threads continúen.
* Errores que sólo ocurren en ciertas situaciones y son difíciles de reproducir y arreglar
  con seguridad.

Rust intenta mitigar los efectos negativos del uso de threads. La programación en
un contexto multithread todavía toma un pensamiento cuidadoso y requiere una estructura
de código que es diferente de los programas que funcionan en un solo thread.

Los lenguajes de programación implementan los thread de forma diferente. Muchos sistemas
operativos proporcionan una API para crear nuevos threads de texto. Este modelo, en el que un
lenguaje llama a las API del sistema operativo para crear threads, suele llamarse *1:1*,
un thread del sistema operativo por un thread del lenguaje.

Muchos lenguajes de programación proporcionan su propia implementación especial de threads.
Los threads proporcionados por el lenguaje de programación se conocen como threads *verdes*, y
los lenguajes que usan estos threads verdes los ejecutarán en el contexto de un número
diferente de threads del sistema operativo. Por esta razón, el modelo verde subprocesado
se denomina modelo *M:N*: `M` threads verdes por cada uno de los threads del sistema operativo
`N`, donde `M` y `N` no son necesariamente el mismo número.

Cada modelo tiene sus propias ventajas y compensaciones, y la compensación más
importante para Rust es el soporte en Tiempo de ejecución (Runtime). Tiempo de ejecución es un 
término confuso y puede tener diferentes significados en diferentes contextos.

En este contexto, por *tiempo de ejecución* nos referimos al código que está incluido por el lenguaje
en cada binario. Este código puede ser grande o pequeño dependiendo del idioma, pero
cada lenguaje no ensamblado tendrá una cierta cantidad de código de tiempo de ejecución. Por esa razón,
coloquialmente cuando la gente dice que un lenguaje no tiene "tiempo de ejecución", a menudo se 
refiere a "poco tiempo de ejecución". Los tiempos de ejecución más pequeños tienen menos funciones pero 
tienen la ventaja de resultar en binarios más pequeños, lo que facilita la combinación del
lenguaje con otros lenguajes en más contextos. Aunque muchos lenguajes están de acuerdo
con aumentar el tamaño del tiempo de ejecución a cambio de más características, Rust necesita
tener casi ningún tiempo de ejecución y no puede comprometerse en poder llamar a C para
mantener el rendimiento.

El modelo verde subprocesado M:N requiere un mayor tiempo de ejecución del lenguaje para gestionar
los threads. Como tal, la biblioteca estándar de Rust sólo proporciona una implementación de 
subprocesado 1:1. Debido a que Rust es un lenguaje de tan bajo nivel, hay crates que 
implementan el subprocesado M:N si tu prefieres cambiar los gastos generales por aspectos tales
como un mayor control sobre qué threads corren cuando, y menores costos de cambio de contexto,
por ejemplo.

Ahora que hemos definido los threads en Rust, vamos a explorar cómo utilizar la
API relacionada con los threads que proporciona la biblioteca estándar.

### Creando un Nuevo Thread con `spawn`

Para crear un nuevo thread, llamamos a la función `thread::spawn` y le pasamos un
cierre (hemos hablado de cierres en el capítulo 13) que contiene el código que queremos 
ejecutar en el nuevo thread. El ejemplo del Listado 16-1 imprime un texto de un thread
principal y otro de un nuevo thread:

<span class="filename">Nombre del archivo: src/main.rs</span>

```rust
use std::thread;
use std::time::Duration;

fn main() {
    thread::spawn(|| {
        for i in 1..10 {
            println!("¡hola número {} del thread generado!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("¡hola numero {} del thread principal!", i);
        thread::sleep(Duration::from_millis(1));
    }
}
```

<span class="caption">Listado 16-1: Creando un nuevo thread para imprimir una cosa 
mientras que el thread principal imprime otra cosa.</span>

Ten en cuenta que con esta función, el nuevo thread se detendrá cuando termine el 
thread principal, haya terminado o no de ejecutarse. La salida de este programa
puede ser un poco diferente cada vez, pero se verá similar a la 
siguiente:

```text
¡Hola número 1 del thread principal!
¡Hola número 1 del thread generado!
¡Hola número 2 del thread principal!
¡Hola número 2 del thread generado!
¡Hola número 3 del thread principal!
¡Hola número 3 del thread generado!
¡Hola número 4 del thread principal!
¡Hola número 4 del thread generado!
¡Hola número 5 del thread generado!
```

Las llamadas a `thread::sleep` fuerzan a un thread a detener su ejecución por un corto
periodo de tiempo, lo que permite que un thread diferente corra. Los thread probablemente
se turnarán, pero eso no está garantizado: depende de cómo tu sistema operativo los
programe. En esta ejecución, el thread principal se imprime primero, aunque la
declaración de impresión del thread generado aparezca primero en el código. Y aunque
le dijimos al thread generado que imprimiera hasta que `i` fuera 9, sólo llegó a 5 
antes de que el thread principal se apagara.
 
Si ejecutas este código y sólo ves la salida del thread principal, o no ves ningún 
solapamiento, intenta aumentar los números en los rangos para crear más oportunidades
para que el sistema operativo cambie entre los threads.

### Esperando a que Todos los Threads Terminen con los Handles `join`

El código en el Listado 16-1 no sólo detiene el thread generado prematuramente la mayor
parte del tiempo debido a la terminación del thread principal, sino que no hay garantía de que
el thread generado vaya a funcionar en absoluto. La razón es que ¡no hay garantía en el
orden de ejecución de los threads!

Podemos arreglar el problema del thread generado no consiguiendo correr, o no 
consiguiendo correr completamente, y guardando el valor de retorno de `thread::spawn` en una
variable. El tipo de retorno de `thread::spawn` es `JoinHandle`. Un `JoinHandle` es un valor
propio que, cuando llamamos el método `join` en él, esperará a que termine su thread.
El Listado 16-2 muestra cómo usar el `JoinHandle` del thread que creamos en el Listado 16-1
y llamar a `join` para asegurarse de que el thread generado termine antes de las salidas
`main`:

<span class="filename">Nombre del archivo: src/main.rs</span>

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hola número {} del thread principal!", i);
        thread::sleep(Duration::from_millis(1));
    }

    handle.join().unwrap();
}
```

<span class="caption">Listado 16-2: Guardando un `JoinHandle` de `thread::spawn`
para garantizar que el thread se ejecuta hasta completarse</span>

Llamar a `join` en el handle, bloquea el thread que se está ejecutando hasta
que finaliza el thread representado por el handle. *Bloquear* un thread significa que
el thread no puede trabajar o salir del mismo. Debido a que hemos puesto la llamada a
`join` después del loop `for` del thread principal, la ejecución del Listado 16-2 debería
producir una salida similar a esta:

```text
¡hola número 1 del thread principal!
¡hola número 2 del thread principal!
hola número 1 del thread generado!
¡hola número 3 del hilo principal!
hola número 2 del thread generado!
¡hola número 4 del hilo principal!
hola número 3 del thread generado!
hola número 4 del thread generado!
hola número 5 del thread generado!
hola número 6 del thread generado!
hola número 7 del thread generado!
hola número 8 del thread generado!
hola número 9 del thread generado!
```

The two threads continue alternating, but the main thread waits because of the
call to `handle.join()` and does not end until the spawned thread is finished.

But let’s see what happens when we instead move `handle.join()` before the
`for` loop in `main`, like this:

<span class="filename">Filename: src/main.rs</span>

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    handle.join().unwrap();

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }
}
```

The main thread will wait for the spawned thread to finish and then run its
`for` loop, so the output won’t be interleaved anymore, as shown here:

```text
hi number 1 from the spawned thread!
hi number 2 from the spawned thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 3 from the main thread!
hi number 4 from the main thread!
```

Thinking about such a small detail as where to call `join` can affect whether
or not your threads run at the same time.

### Using `move` Closures with Threads

The `move` closure, which we mentioned briefly in Chapter 13, is often used
alongside `thread::spawn` because it allows us to use data from one thread in
another thread.

In Chapter 13, we said that “If we want to force the closure to take ownership
of the values it uses in the environment, we can use the `move` keyword before
the parameter list. This technique is mostly useful when passing a closure to a
new thread to move the data so it’s owned by the new thread.”

Now that we’re creating new threads, we’ll talk about capturing values in
closures.

Notice in Listing 16-1 that the closure we pass to `thread::spawn` takes no
arguments: we’re not using any data from the main thread in the spawned
thread’s code. To do so, the spawned thread’s closure must capture the values
it needs. Listing 16-3 shows an attempt to create a vector in the main thread
and use it in the spawned thread. However, this won’t yet work, as you’ll see
in a moment:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(|| {
        println!("Here's a vector: {:?}", v);
    });

    handle.join().unwrap();
}
```

<span class="caption">Listing 16-3: Attempting to use a vector created by the
main thread in another thread</span>

The closure uses `v`, so it will capture `v` and make it part of the closure’s
environment. Because `thread::spawn` runs this closure in a new thread, we
should be able to access `v` inside that new thread. But when we compile this
example, we get the following error:

```text
error[E0373]: closure may outlive the current function, but it borrows `v`,
which is owned by the current function
 --> src/main.rs:6:32
  |
6 |     let handle = thread::spawn(|| {
  |                                ^^ may outlive borrowed value `v`
7 |         println!("Here's a vector: {:?}", v);
  |                                           - `v` is borrowed here
  |
help: to force the closure to take ownership of `v` (and any other referenced
variables), use the `move` keyword
  |
6 |     let handle = thread::spawn(move || {
  |                                ^^^^^^^
```

Rust *infers* how to capture `v`, and because `println!` only needs a reference
to `v`, the closure tries to borrow `v`. However, there’s a problem: Rust can’t
tell how long the spawned thread will run, so it doesn’t know if the reference
to `v` will always be valid.

Listing 16-4 provides a scenario that’s more likely to have a reference to `v`
that won’t be valid:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(|| {
        println!("Here's a vector: {:?}", v);
    });

    drop(v); // oh no!

    handle.join().unwrap();
}
```

<span class="caption">Listing 16-4: A thread with a closure that attempts to
capture a reference to `v` from a main thread that drops `v`</span>

If we were allowed to run this code, there’s a possibility the spawned thread
will be immediately put in the background without running at all. The spawned
thread has a reference to `v` inside, but the main thread immediately drops
`v`, using the `drop` function we discussed in Chapter 15. Then, when the
spawned thread starts to execute, `v` is no longer valid, so a reference to it
is also invalid. Oh no!

To fix the compiler error in Listing 16-3, we can use the error message’s
advice:

```text
help: to force the closure to take ownership of `v` (and any other referenced
variables), use the `move` keyword
  |
6 |     let handle = thread::spawn(move || {
  |                                ^^^^^^^
```

By adding the `move` keyword before the closure, we force the closure to take
ownership of the values it’s using rather than allowing Rust to infer that it
should borrow the values. The modification to Listing 16-3 shown in Listing
16-5 will compile and run as we intend:

<span class="filename">Filename: src/main.rs</span>

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(move || {
        println!("Here's a vector: {:?}", v);
    });

    handle.join().unwrap();
}
```

<span class="caption">Listing 16-5: Using the `move` keyword to force a closure
to take ownership of the values it uses</span>

What would happen to the code in Listing 16-4 where the main thread called
`drop` if we use a `move` closure? Would `move` fix that case? Unfortunately,
no; we would get a different error because what Listing 16-4 is trying to do
isn’t allowed for a different reason. If we add `move` to the closure, we would
move `v` into the closure’s environment, and we could no longer call `drop` on
it in the main thread. We would get this compiler error instead:

```text
error[E0382]: use of moved value: `v`
  --> src/main.rs:10:10
   |
6  |     let handle = thread::spawn(move || {
   |                                ------- value moved (into closure) here
...
10 |     drop(v); // oh no!
   |          ^ value used here after move
   |
   = note: move occurs because `v` has type `std::vec::Vec<i32>`, which does
   not implement the `Copy` trait
```

Rust’s ownership rules have saved us again! We got an error from the code in
Listing 16-3 because Rust was being conservative and only borrowing `v` for the
thread, which meant the main thread could theoretically invalidate the spawned
thread’s reference. By telling Rust to move ownership of `v` to the spawned
thread, we’re guaranteeing Rust that the main thread won’t use `v` anymore. If
we change Listing 16-4 in the same way, we’re then violating the ownership
rules when we try to use `v` in the main thread. The `move` keyword overrides
Rust’s conservative default of borrowing; it doesn’t let us violate the
ownership rules.

With a basic understanding of threads and the thread API, let’s look at what we
can *do* with threads.
