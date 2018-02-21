## Diseñando la interfaz del Grupo de Subproceso

Vamos a hablar sobre cómo debería verse el uso del grupo. Los autores a menudo encuentran 
que cuando se intenta diseñar algún código, escribir primero la interfaz del cliente puede 
ayudar realmente a guiar su diseño. Escriba la API del código que se estructurará de la 
manera en que le gustaría llamarlo, luego implemente la funcionalidad dentro de esa estructura 
en lugar de implementar la funcionalidad y luego diseñar la API pública.

De manera similar a como utilizamos el Desarrollo Impulsado por Prueba en el proyecto del 
Capítulo 12, vamos a usar el Desarrollo Impulsado por el Compilador aquí. Vamos a escribir 
el código que llama a las funciones que deseamos tener, luego nos apoyaremos en el compilador 
para decirnos qué debemos cambiar a continuación. Los mensajes de error del compilador guiarán 
nuestra implementación.

### Estructura del Código si Pudiéramos Usar `thread::spawn`

Primero, exploremos cómo podría ser el código para crear un nuevo subproceso para cada conexión. 
Este no es nuestro plan final debido a los problemas con el posible desarrollo de un número 
ilimitado de subprocesos de los que hablamos anteriormente, pero es un comienzo. El Listado 20-11 
muestra los cambios a `main` para engendrar un nuevo subproceso para manejar cada flujo dentro del 
bucle `for`:

<span class="filename">Nombre del archivo: src/main.rs</span>

```rust,no_run
# use std::thread;
# use std::io::prelude::*;
# use std::net::TcpListener;
# use std::net::TcpStream;
#
fn main() {
    let listener = TcpListener::bind("127.0.0.1:8080").unwrap();

    for stream in listener.incoming() {
        let stream = stream.unwrap();

        thread::spawn(|| {
            handle_connection(stream);
        });
    }
}
# fn handle_connection(mut stream: TcpStream) {}
```

<span class="caption">Listado 20-11: Engendrando un nuevo subproceso para cada
flujo</span>

Como aprendimos en el Capítulo 16, `thread::spawn` creará un nuevo subproceso y luego ejecutará 
el código en el cierre. Si ejecuta este código y carga `/sleep` y luego `/` en dos pestañas del 
navegador, verá que la solicitud a `/` no tiene que esperar a que `/sleep` termine. Pero como 
mencionamos, esto eventualmente abrumará al sistema ya que estamos creando nuevos subprocesos sin límite. 

### Creando una Interfaz Similar para `ThreadPool`

Queremos que nuestro grupo de subprocesos funcione de forma similar y familiar, de modo que el 
cambio de subprocesos a un grupo de subprocesos no requiera grandes cambios en el código que queremos 
ejecutar en el grupo. El Listado 20-12 muestra la interfaz hipotética para una estructura `ThreadPool` 
que nos gustaría usar en lugar de `thread::spawn`:

<span class="filename">Nombre del archivo: src/main.rs</span>

```rust,no_run
# use std::thread;
# use std::io::prelude::*;
# use std::net::TcpListener;
# use std::net::TcpStream;
# struct ThreadPool;
# impl ThreadPool {
#    fn new(size: u32) -> ThreadPool { ThreadPool }
#    fn execute<F>(&self, f: F)
#        where F: FnOnce() + Send + 'static {}
# }
#
fn main() {
    let listener = TcpListener::bind("127.0.0.1:8080").unwrap();
    let pool = ThreadPool::new(4);

    for stream in listener.incoming() {
        let stream = stream.unwrap();

        pool.execute(|| {
            handle_connection(stream);
        });
    }
}
# fn handle_connection(mut stream: TcpStream) {}
```

<span class="caption">Listado 20-12: Cómo queremos ser capaces de usar el `ThreadPool` vamos a implementar</span>

Usamos `ThreadPool::new` para crear un nuevo grupo de subprocesos con un número configurable de 
subprocesos, en este caso cuatro. Luego, en el bucle `for`, `pool.execute` funcionará de forma 
similar a `thread::spawn`.

### Desarrollo Impulsado por el Compilador para Obtener la Compilación API

Siga adelante y realice los cambios en el Listado 20-12 a *src/main.rs*, y usemos los errores del 
compilador para impulsar nuestro desarrollo. Aquí está el primer error que obtenemos:

```text
$ cargo check
   Compiling hello v0.1.0 (file:///projects/hello)
error[E0433]: failed to resolve. Use of undeclared type or module `ThreadPool`
  --> src\main.rs:10:16
   |
10 |     let pool = ThreadPool::new(4);
   |                ^^^^^^^^^^^^^^^ Use of undeclared type or module
   `ThreadPool`

error: aborting due to previous error
```

Genial, necesitamos un `ThreadPool`. Cambiemos la caja `hello` de una caja binaria a una caja 
de biblioteca para mantener nuestra implementación `ThreadPool`, ya que la implementación del 
grupo de subprocesos será independiente del tipo particular de trabajo que estamos haciendo en 
nuestro servidor web. Una vez que tengamos la biblioteca del grupo de subprocesos escrita, podríamos 
usar esa funcionalidad para hacer el trabajo que queramos hacer, no solo para servir solicitudes web.

Entonces cree *src/lib.rs* que contenga la definición más simple de una estructura `ThreadPool` 
que podamos tener por ahora:

<span class="filename">Nombre del archivo: src/lib.rs</span>

```rust
pub struct ThreadPool;
```

A continuación, cree un nuevo directorio, *src/bin*, y mueva la caja binaria enraizada en *src/main.rs* 
dentro de *src/bin/main.rs*. Esto hará que la caja de la biblioteca sea la caja primaria en el 
directorio *hello*; aún podemos ejecutar el binario en *src/bin/main.rs* usando `cargo run`. Después 
de mover el archivo *main.rs*, edítelo para traer la caja de la biblioteca y poner `ThreadPool` en el 
alcance agregando esto en la parte superior de *src/bin/main.rs*:

<span class="filename">Nombre del archivo: src/bin/main.rs</span>

```rust,ignore
extern crate hello;
use hello::ThreadPool;
```

Y vuelva a intentarlo para obtener el siguiente error que debemos abordar:

```text
$ cargo check
   Compiling hello v0.1.0 (file:///projects/hello)
error: no associated item named `new` found for type `hello::ThreadPool` in the
current scope
  --> src\main.rs:13:16
   |
13 |     let pool = ThreadPool::new(4);
   |                ^^^^^^^^^^^^^^^
   |
```

Genial, lo siguiente es crear una función asociada llamada `new` para `ThreadPool`. También sabemos 
que `new` necesita tener un parámetro que pueda aceptar `4` como argumento, y `new` debería devolver 
una instancia `ThreadPool`. Implementemos la función `new` más simple que tendrá esas características:

<span class="filename">Nombre del archivo: src/lib.rs</span>

```rust
pub struct ThreadPool;

impl ThreadPool {
    pub fn new(size: u32) -> ThreadPool {
        ThreadPool
    }
}
```

Escogimos `u32` como el tipo del parámetro `size` ya que sabemos que un número negativo de 
subprocesos no tiene sentido. `u32` es un valor predeterminado sólido. Una vez que realmente 
implementamos `new` de verdad, reconsideraremos si esta es la opción correcta para lo que la 
implementación necesita, pero por ahora, solo estamos trabajando en los errores del compilador.

Revisemos el código nuevamente:

```text
$ cargo check
   Compiling hello v0.1.0 (file:///projects/hello)
warning: unused variable: `size`, #[warn(unused_variables)] on by default
 --> src/lib.rs:4:16
  |
4 |     pub fn new(size: u32) -> ThreadPool {
  |                ^^^^

error: no method named `execute` found for type `hello::ThreadPool` in the
current scope
  --> src/main.rs:18:14
   |
18 |         pool.execute(|| {
   |              ^^^^^^^
```

De acuerdo, una advertencia y un error. Ignorando la advertencia por un momento, el error es 
porque no tenemos un método `execute` en `ThreadPool`. Definamos uno, y necesitamos que tome un 
cierre. Si recuerda del Capítulo 13, podemos tomar cierres como argumentos con tres rasgos 
diferentes: `Fn`, `FnMut`, y `FnOnce`. ¿Qué tipo de cierre deberíamos usar? Bueno, sabemos que 
vamos a terminar haciendo algo similar a `thread::spawn`; ¿Qué límites tiene la firma de `thread::spawn` 
en su argumento? Miremos la documentación, que dice:

```rust,ignore
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T + Send + 'static,
        T: Send + 'static
```

`F` es el parámetro que nos importa aquí; `T` está relacionado con el valor de retorno y no estamos 
preocupados con eso. Dado que `spawn` usa `FnOnce` como el rasgo ligado en `F`, es probablemente lo 
que también queremos, ya que eventualmente pasaremos el argumento que obtenemos en `execute` a `spawn`. 
Podemos estar más seguros de que `FnOnce` es el rasgo que queremos usar, ya que el subproceso para 
ejecutar una solicitud solo ejecutará el cierre de esa solicitud una sola vez.

`F` también tiene el atributo `Send` y el límite de vida `'static`, que también tienen sentido para 
nuestra situación: necesitamos `Send` para transferir el cierre de un subproceso a otro, y `'static` 
porque no sé cuánto tiempo se ejecutará el subproceso. Vamos a crear un método `execute` en `ThreadPool` 
que tomará un parámetro genérico `F` con estos límites:

<span class="filename">Nombre del archivo: src/lib.rs</span>

```rust
# pub struct ThreadPool;
impl ThreadPool {
    // --snip--

    pub fn execute<F>(&self, f: F)
        where
            F: FnOnce() + Send + 'static
    {

    }
}
```

El rasgo `FnOnce` todavía necesita el `()` después de este, ya que este `FnOnce` representa un cierre 
que no toma parámetros y no devuelve un valor. Al igual que las definiciones de función, el tipo de 
devolución puede omitirse de la firma, pero incluso si no tenemos parámetros, aún necesitamos los paréntesis.

Nuevamente, dado que estamos trabajando para obtener la compilación de la interfaz, estamos agregando la 
implementación más simple del método `execute`, el cual no hace nada. Revisemos nuevamente:

```text
$ cargo check
   Compiling hello v0.1.0 (file:///projects/hello)
warning: unused variable: `size`, #[warn(unused_variables)] on by default
 --> src/lib.rs:4:16
  |
4 |     pub fn new(size: u32) -> ThreadPool {
  |                ^^^^

warning: unused variable: `f`, #[warn(unused_variables)] on by default
 --> src/lib.rs:8:30
  |
8 |     pub fn execute<F>(&self, f: F)
  |                              ^
```

¡Solo advertencias ahora! ¡Compila! Tenga en cuenta que si prueba `cargo run` y realiza una solicitud 
en el navegador, verá los errores en el navegador de nuevo que vimos al principio del capítulo. 
¡Nuestra biblioteca no está llamando al cierre pasado a `execute` todavía!

> Un dicho que puede escuchar sobre idiomas con compiladores estrictos como Haskell
> y Rust es "si el código se compila, funciona". Este es un buen momento para recordar
> que esta es solo una frase y un sentimiento que la gente a veces tiene, no es
> universalmente verdadero. Nuestro proyecto compila, ¡pero no hace absolutamente
> nada! Si estuviéramos construyendo un proyecto real y completo, sería gran
> hora de comenzar a escribir pruebas unitarias para verificar que el código compila *y* tiene
> el comportamiento que queremos.
