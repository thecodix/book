## Creando el Grupo de Subprocesos y Almacenando Subprocesos

Las advertencias se deben a que no estamos haciendo nada con los parámetros para `new`
y `execute`. Implementemos los cuerpos de ambos con el real
comportamiento que queremos.

### Validando el Número de Subprocesos en el Grupo

Para empezar, pensemos en `new`. Anteriormente mencionamos que seleccionamos un tipo 
sin signo para el parámetro `size` ya que un grupo con un número negativo de subprocesos no tiene sentido. Sin embargo, un grupo con cero subprocesos tampoco tiene sentido, pero cero es un `u32` perfectamente válido. Comprobemos que `size` es mayor que cero antes de devolver una instancia `ThreadPool` y pánico si obtenemos cero usando la macro `assert!` Como se muestra en el Listado 20-13:

<span class="filename">Filename: src/lib.rs</span>

```rust
# pub struct ThreadPool;
impl ThreadPool {
    /// Create a new ThreadPool.
    ///
    /// The size is the number of threads in the pool.
    ///
    /// # Panics
    ///
    /// The `new` function will panic if the size is zero.
    pub fn new(size: u32) -> ThreadPool {
        assert!(size > 0);

        ThreadPool
    }

    // --snip--
}
```

<span class="caption">Listing 20-13: Implementing `ThreadPool::new` to panic if
`size` is zero</span>

Aprovechamos esta oportunidad para agregar algo de documentación para nuestro `ThreadPool` con comentarios de doc. Tenga en cuenta que seguimos buenas prácticas de documentación y agregamos una sección que indica las situaciones en las que puede entrar en pánico nuestra función, como lo discutimos en el capítulo 14. Pruebe ejecutando `cargo doc --open` y haga clic en la estructura `ThreadPool` para ver lo que generan los documentos para `new`!

En lugar de agregar el uso de la macro `assert!` como lo hemos hecho aquí, podríamos hacer que `new` devolviera un `Result` en lugar de hacerlo con `Config::new` en el proyecto de I/O en el Listado 12-9, pero hemos decidido en este caso que intentar crear un grupo de subprocesos sin que ningún subproceso deba ser un error irrecuperable. Si se siente ambicioso, intente escribir una versión de `new` con esta firma para ver cómo se siente con respecto a ambas versiones:

```rust,ignore
fn new(size: u32) -> Result<ThreadPool, PoolCreationError> {
```

### Almacenando Subprocesos en el Grupo

Ahora que sabemos que tenemos un número válido de subprocesos para almacenar en el grupo, podemos crear muchos subprocesos y almacenarlos en la estructura `ThreadPool` antes de devolverla.

Esto genera una pregunta: ¿Cómo "almacenamos" un subproceso? Echemos otro vistazo a la firma de `thread::spawn`:

```rust,ignore
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T + Send + 'static,
        T: Send + 'static
```

`spawn` devuelve un `JoinHandle<T>`, donde `T` es el tipo que es devuelto desde el cierre. Tratemos de usar `JoinHandle` también y veamos qué pasa. En nuestro caso, los cierres que estamos pasando al grupo de subprocesos manejarán la conexión y no devolverán nada, por lo que `T` será el tipo de unidad `()`.

Esto no se compilará todavía, pero consideremos el código que se muestra en el Listado 20-14. Hemos cambiado la definición de `ThreadPool` para contener un vector de instancias `thread::JoinHandle<()>`, inicializamos el vector con una capacidad de `size`, configuramos un bucle `for` que ejecutará algún código para crear los subprocesos, y devolvió una instancia `ThreadPool` que los contiene:

<span class="filename">Nombre del archivo: src/lib.rs</span>

```rust,ignore
use std::thread;

pub struct ThreadPool {
    threads: Vec<thread::JoinHandle<()>>,
}

impl ThreadPool {
    // --snip--
    pub fn new(size: u32) -> ThreadPool {
        assert!(size > 0);

        let mut threads = Vec::with_capacity(size);

        for _ in 0..size {
            // create some threads and store them in the vector
        }

        ThreadPool {
            threads
        }
    }

    // --snip--
}
```

<span class="caption">Listado 20-14: Creando un vector para `ThreadPool` para contener los subprocesos</span>

Hemos incluido `std::thread` en el alcance de la caja de la biblioteca, ya que estamos usando `thread::JoinHandle` como el tipo de elementos en el vector en `ThreadPool`.

Después de que tenemos un tamaño válido, estamos creando un nuevo vector que pueda contener elementos de `size`. Todavía no hemos usado `with_capacity` en este libro; hace lo mismo que `Vec::new`, pero con una diferencia importante: asigna previamente espacio en el vector. Como sabemos que tenemos que almacenar elementos `size` en el vector, hacer esta asignación por adelantado es ligeramente más eficiente que solo escribir `Vec::new`, ya que `Vec::new` se cambia de tamaño a medida que los elementos se insertan. Como hemos creado un vector del tamaño exacto que necesitamos por adelantado, no cambiará el tamaño del vector subyacente mientras llenemos los elementos.

Es decir, si este código funciona, ¡lo cual aún no hace! Si revisamos este código, obtenemos un error:

```text
$ cargo check
   Compiling hello v0.1.0 (file:///projects/hello)
error[E0308]: mismatched types
  --> src\main.rs:70:46
   |
70 |         let mut threads = Vec::with_capacity(size);
   |                                              ^^^^ expected usize, found u32

error: aborting due to previous error
```

`size` es un `u32`, pero `Vec::with_capacity` necesita un `usize`. Aquí tenemos dos opciones: podemos cambiar la firma de nuestra función, o podemos moldear el `u32` como un `usize`. Si ustéd recuerda cuando definimos `new`, no pensamos demasiado sobre qué tipo de número tenía sentido, simplemente elegimos uno. Vamos a darle un poco más de pensamiento ahora. Dado que `size` es la longitud de un vector, `usize` tiene mucho sentido. ¡Casi comparten un nombre! Cambiemos la firma de `new`, que obtendrá el código en el Listado 20-14 para compilar:

```rust,ignore
fn new(size: usize) -> ThreadPool {
```

Si ejecuta `cargo check` nuevamente, obtendrá algunas advertencias más, pero debería tener éxito.

Dejamos un comentario en el bucle `for` en el Listado 20-14 con respecto a la creación de subprocesos. ¿Cómo creamos subprocesos? Esta es una pregunta difícil. ¿Qué debería ir en estos subprocesos? No sabemos qué trabajo tienen que hacer en este punto, ya que el método `execute` toma el cierre y se lo da al grupo.

Vamos a refactorizar un poco: en lugar de almacenar un vector de instancias `JoinHandle<()>`, creemos una nueva estructura para representar el concepto de un *trabajador*. Un trabajador será el que reciba un cierre en el método `execute`, y se encargará de llamar al cierre. Además de permitirnos almacenar un número `size` fijo de instancias `Worker` que aún no conocen los cierres que van a ejecutar, también podemos darle a cada trabajador un `id` para que podamos distinguir los diferentes trabajadores en el grupo aparte cuando inician sesión o depuración.

Hagamos estos cambios:

1. Defina una estructura `Worker` que contenga un `id` y un `JoinHandle<()>`
2. Cambie `ThreadPool` para contener un vector de instancias `Worker`
3. Defina una función `Worker::new` que toma un número `id` y devuelve una instancia `Worker` con ese `id` y un subproceso generado con un cierre vacío, que reparará pronto
4. En `ThreadPool::new`, use el contador de bucles `for` para generar un `id`, cree un nuevo `Worker` con ese `id`, y almacene el trabajador en el vector

Si está preparado para un desafío, intente implementar estos cambios por su cuenta antes de echar un vistazo al código en el Listado 20-15.

Listo? Aquí está el Listado 20-15 con una forma de hacer estas modificaciones:

<span class="filename">Filename: src/lib.rs</span>

```rust
use std::thread;

pub struct ThreadPool {
    workers: Vec<Worker>,
}

impl ThreadPool {
    // --snip--
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let mut workers = Vec::with_capacity(size);

        for id in 0..size {
            workers.push(Worker::new(id));
        }

        ThreadPool {
            workers
        }
    }
    // --snip--
}

struct Worker {
    id: usize,
    thread: thread::JoinHandle<()>,
}

impl Worker {
    fn new(id: usize) -> Worker {
        let thread = thread::spawn(|| {});

        Worker {
            id,
            thread,
        }
    }
}
```

<span class="caption">Listado 20-15: Modificando `ThreadPool` para contener instancias `Worker` en vez de subprocesos directamente</span>

Hemos elegido cambiar el nombre del campo en `ThreadPool` de `threads` a `workers` ya que hemos cambiado lo que estamos manteniendo, que ahora son instancias `Worker` en lugar de instancias `JoinHandle<()>`. Usamos el contador en el bucle `for` como argumento para `Worker::new`, y almacenamos cada `Worker` nuevo en el vector llamado `workers`.

La estructura `Worker` y su función `new` son privadas ya que el código externo (como nuestro servidor en *src/bin/main.rs*) no necesita conocer el detalle de la implementación de que estamos usando una estructura `Worker` dentro de `ThreadPool`. La función `Worker::new` usa el `id` dado y almacena un `JoinHandle<()>` creado al generar un nuevo subproceso usando un cierre vacío.

Este código compila y almacena el número de instancias `Worker` que especificamos como argumento para` ThreadPool::new`, pero *todavía* no procesamos el cierre que obtenemos en `execute`. Hablemos de cómo hacer eso a continuación.
