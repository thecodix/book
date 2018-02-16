## Graceful Shutdown and Cleanup

El código del listado 20-21 responde a las solicitudes de forma asincrónica a través del uso de un grupo de subprocesos, como pretendíamos. Recibimos algunas advertencias sobre los campos que no estamos usando de manera directa, que son un recordatorio de que no estamos limpiando todo arriba. Cuando usamos `CTRL-C` para detener el hilo principal, todos los demás los hilos también se detienen inmediatamente, incluso si están atendiendo una solicitud

Ahora vamos a implementar el atributo `Drop` para` ThreadPool` para llamar `join` en cada uno de los subprocesos del grupo y que estos finalicen las solicitudes que están trabajando. Luego implementaremos una forma para que `ThreadPool` le diga a los hilos que deben dejar de aceptar nuevas solicitudes y cerrarse. Para ver este código en acción, modificaremos nuestro servidor para que solo acepte dos solicitudes antes de cerrar su grupo de subprocesos.

Comencemos implementando `Drop` para nuestro grupo de subprocesos. Cuando el grupo esta
caído, debemos unirnos a todos nuestros hilos para asegurarnos de que terminen su
trabajo. El listado 20-22 muestra un primer intento de implementación de `Drop`; este código
aún no funcionará:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
impl Drop for ThreadPool {
    fn drop(&mut self) {
        for worker in &mut self.workers {
            println!("Shutting down worker {}", worker.id);

            worker.thread.join().unwrap();
        }
    }
}
```

<span class="caption">Listing 20-22: Joining each thread when the thread pool
goes out of scope</span>

Recorrimos cada uno de los `pool` del grupo de subprocesos, usando` & mut` porque `self` es en sí misma es una referencia mutable y también necesitamos poder mutar `worker`. Imprimimos un mensaje que dice que este worker en particular se está cerrando, y luego llamamos a `join` al hilo de ese worker. Si la llamada a `join` falla, nosotros `deploy` el error para entrar en pánico y realizar un apagado desagradable.

Aquí está el error que obtenemos si compilamos este código:

```text
error[E0507]: cannot move out of borrowed content
  --> src/lib.rs:65:13
   |
65 |             worker.thread.join().unwrap();
   |             ^^^^^^ cannot move out of borrowed content
```

Como solo tenemos un préstamo mutable de cada `worker`, no podemos llamar` join`: `join` toma posesión de su argumento. Para resolver esto, necesitamos una manera para mover el `thread` de la instancia` Worker` que posee `thread` para que `join` pueda consumir el hilo. Vimos una forma de hacer esto en el listado 17-15: si el `Worker` contiene una` Opción <thread :: JoinHandle <()> `en su lugar, podemos llamar al método `take` en la` Opción` para mover el valor fuera de la variante `Some` y dejar una variante `None` en su lugar. En otras palabras, un `Worker` que se está ejecutando tendrá una variante `Some` en` thread`, y cuando queramos limpiar un worker, reemplazaremos `Some` con` None` para que el worker no tenga un hilo para ejecutar.

Entonces sabemos que queremos actualizar la definición de `worker` de esta manera:

<span class="filename">Filename: src/lib.rs</span>

```rust
# use std::thread;
struct Worker {
    id: usize,
    thread: Option<thread::JoinHandle<()>>,
}
```

Ahora apoyémonos en el compilador para encontrar los otros lugares que se necesitan cambiar. Obtenemos dos errores:

```text
error: no method named `join` found for type
`std::option::Option<std::thread::JoinHandle<()>>` in the current scope
  --> src/lib.rs:65:27
   |
65 |             worker.thread.join().unwrap();
   |                           ^^^^

error[E0308]: mismatched types
  --> src/lib.rs:89:21
   |
89 |             thread,
   |             ^^^^^^ expected enum `std::option::Option`, found
   struct `std::thread::JoinHandle`
   |
   = note: expected type `std::option::Option<std::thread::JoinHandle<()>>`
              found type `std::thread::JoinHandle<_>`
```

El segundo error apunta al código al final de `Worker :: new`; que necesitamos
para envolver el valor `thread` en` Some` cuando creamos un nuevo `Worker`:


<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        // ...snip...

        Worker {
            id,
            thread: Some(thread),
        }
    }
}
```

El primer error está en nuestra implementación `Drop`, y mencionamos que estaremos
llamando a `take` en el valor `Option` para mover `thread` de `worker`. Aquí está
como se ve eso:


<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
impl Drop for ThreadPool {
    fn drop(&mut self) {
        for worker in &mut self.workers {
            println!("Shutting down worker {}", worker.id);

            if let Some(thread) = worker.thread.take() {
                thread.join().unwrap();
            }
        }
    }
}
```

Como vimos en el capítulo 17, el método `take` en` Option` toma la variante `Some` y deja `None` en su lugar. Estamos usando `if let` para desestructurar el `Some` y obtener el hilo, luego llamamos` join` en el hilo. Si el hilo de un worker ya es `None`, entonces sabemos que este worker ya se ha limpiado del hilo así que no hacemos nada en ese caso.

Con esto, nuestro código se compila sin advertencias. Malas noticias, este código no funciona de la manera que queremos todavía. La clave es la lógica en el cierre que ejecutan los hilos de las instancias `Worker`: llamar` join` no cerrará los hilos ya que `loop` siempre buscan trabajo. Si nosotros intentamos soltar nuestro `ThreadPool` con esta implementación, el hilo principal se bloqueará para siempre esperando que termine el primer hilo.

Para solucionar esto vamos a modificar los hilos, para escuchar un `Job` o para
ejecutar una señal de que deben dejar de escuchar y salir del bucle infinito. Así que
en lugar de instancias `Job`, nuestro canal enviará una de estas dos enumeraciones
variantes:


<span class="filename">Filename: src/lib.rs</span>

```rust
# struct Job;
enum Message {
    NewJob(Job),
    Terminate,
}
```

Esta enumeración `Message` será una variante` NewJob` que contiene el `Job` que debe ejecutarse, o será una variante `Terminate` que cambiara al hilo para salir de su ciclo y detenerse.

Necesitamos ajustar el canal para usar valores de tipo `Message` en lugar de escribir `job`, como se muestra en el Listado 20-23:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
pub struct ThreadPool {
    workers: Vec<Worker>,
    sender: mpsc::Sender<Message>,
}

// ...snip...

impl ThreadPool {
    // ...snip...
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let (sender, receiver) = mpsc::channel();

        // ...snip...
    }

    pub fn execute<F>(&self, f: F)
        where
            F: FnOnce() + Send + 'static
    {
        let job = Box::new(f);

        self.sender.send(Message::NewJob(job)).unwrap();
    }
}

// ...snip...

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Message>>>) ->
        Worker {

        let thread = thread::spawn(move ||{
            loop {
                let message = receiver.lock().unwrap().recv().unwrap();

                match message {
                    Message::NewJob(job) => {
                        println!("Worker {} got a job; executing.", id);

                        job.call_box();
                    },
                    Message::Terminate => {
                        println!("Worker {} was told to terminate.", id);

                        break;
                    },
                }
            }
        });

        Worker {
            id,
            thread: Some(thread),
        }
    }
}
```

<span class="caption">Listing 20-23: Sending and receiving `Message` values and
exiting the loop if a `Worker` receives `Message::Terminate`</span>

Necesitamos cambiar `Job` a` Message` en la definición de `ThreadPool`, en `ThreadPool :: new` donde creamos el canal, y en la firma de `worker :: new`. El método `execute` de` ThreadPool` necesita enviar trabajos envueltos en la variante `Message :: NewJob`. Luego, en `Worker :: new` donde recibimos un `Message` del canal, procesaremos el trabajo si obtenemos` NewJob` variante y salga del ciclo si obtenemos la variante `Terminate`.

Con estos cambios, el código se compilará nuevamente y continuará funcionando en el
de la misma manera que lo venía haciendo. Sin embargo, tendremos una advertencia porque no estamos usando la variante `Terminate` en cualquier mensaje. Cambiemos nuestra implementación `Drop` para que se parezca al Listado 20-24:


<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
impl Drop for ThreadPool {
    fn drop(&mut self) {
        println!("Sending terminate message to all workers.");

        for _ in &mut self.workers {
            self.sender.send(Message::Terminate).unwrap();
        }

        println!("Shutting down all workers.");

        for worker in &mut self.workers {
            println!("Shutting down worker {}", worker.id);

            if let Some(thread) = worker.thread.take() {
                thread.join().unwrap();
            }
        }
    }
}
```

<span class="caption">Listing 20-24: Sending `Message::Terminate` to the
workers before calling `join` on each worker thread</span>

Ahora estamos iterando sobre los worker dos veces, una para enviar un mensaje `Terminate` para cada worker, y otra para llamar `join` en el hilo de cada worker. Si nosotros intentamos enviar un mensaje y unirnos inmediatamente en el mismo ciclo, no se garantiza que el worker en la iteración actual será el que obtenga el mensaje del canal

Para comprender mejor por qué necesitamos dos bucles separados, imagine un escenario con
dos worker, Si iteramos a través de cada worker un bucle, en la primer
iteración le enviaremos un mensaje de terminación por el canal y llamaremos a `join` en el hilo del primer worker. Si el primero el worker estaba ocupado procesando una solicitud en ese momento, el segundo worker recoger el mensaje de terminación del canal y lo cierra. De otra manera estaríamos esperando al primer worker para cerrar, pero nunca lo hará ya que esta ocupado con el otro hilo elegido hasta el mensaje de finalización. Ahora estamos bloqueados para siempre esperando al primer worker para cerrar, y nunca enviaremos el segundo mensaje para terminar.
¡Punto muerto!


Para evitar esto, primero colocamos todos nuestros mensajes `Terminate` en el canal, y luego nos unimos a todos los hilos. Porque cada worker dejará de recibir solicitudes en el canal una vez que recibe un mensaje de terminación, podemos estar seguros de que si enviamos el mismo número de mensajes de finalización que de workers, cada uno recibirá un mensaje de terminación antes de que llamemos a `join` en su hilo.

In order to see this code in action, let's modify `main` to only accept two
requests before gracefully shutting the server down as shown in Listing 20-25:

<span class="filename">Filename: src/bin/main.rs</span>

```rust,ignore
fn main() {
    let listener = TcpListener::bind("127.0.0.1:8080").unwrap();
    let pool = ThreadPool::new(4);

    let mut counter = 0;

    for stream in listener.incoming() {
        if counter == 2 {
            println!("Shutting down.");
            break;
        }

        counter += 1;

        let stream = stream.unwrap();

        pool.execute(|| {
            handle_connection(stream);
        });
    }
}
```

<span class="caption">Listing 20-25: Shut down the server after serving two
requests by exiting the loop</span>

Only serving two requests isn't behavior you'd like a production web server to
have, but this will let us see the graceful shutdown and cleanup working since
we won't be stopping the server with `CTRL-C`.

We've added a `counter` variable that we'll increment every time we receive an
incoming TCP stream. If that counter reaches 2, we'll stop serving requests and
instead break out of the `for` loop. The `ThreadPool` will go out of scope at
the end of `main`, and we'll see the `drop` implementation run.

Start the server with `cargo run`, and make three requests. The third request
should error, and in your terminal you should see output that looks like:

```text
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished dev [unoptimized + debuginfo] target(s) in 1.0 secs
     Running `target/debug/hello`
Worker 0 got a job; executing.
Worker 3 got a job; executing.
Shutting down.
Sending terminate message to all workers.
Shutting down all workers.
Shutting down worker 0
Worker 1 was told to terminate.
Worker 2 was told to terminate.
Worker 0 was told to terminate.
Worker 3 was told to terminate.
Shutting down worker 1
Shutting down worker 2
Shutting down worker 3
```

You may get a different ordering, of course. We can see how this works from the
messages: workers zero and three got the first two requests, and then on the
third request, we stop accepting connections. When the `ThreadPool` goes out of
scope at the end of `main`, its `Drop` implementation kicks in, and the pool
tells all workers to terminate. The workers each print a message when they see
the terminate message, and then the thread pool calls `join` to shut down each
worker thread.

One interesting aspect of this particular execution: notice that we sent the
terminate messages down the channel, and before any worker received the
messages, we tried to join worker zero. Worker zero had not yet gotten the
terminate message, so the main thread blocked waiting for worker zero to
finish. In the meantime, each of the workers received the termination messages.
Once worker zero finished, the main thread waited for the rest of the workers
to finish, and they had all received the termination message and were able to
shut down at that point.

Congrats! We now have completed our project, and we have a basic web server
that uses a thread pool to respond asynchronously. We're able to perform a
graceful shutdown of the server, which cleans up all the threads in the pool.
Here's the full code for reference:

<span class="filename">Filename: src/bin/main.rs</span>

```rust,ignore
extern crate hello;
use hello::ThreadPool;

use std::io::prelude::*;
use std::net::TcpListener;
use std::net::TcpStream;
use std::fs::File;
use std::thread;
use std::time::Duration;

fn main() {
    let listener = TcpListener::bind("127.0.0.1:8080").unwrap();
    let pool = ThreadPool::new(4);

    let mut counter = 0;

    for stream in listener.incoming() {
        if counter == 2 {
            println!("Shutting down.");
            break;
        }

        counter += 1;

        let stream = stream.unwrap();

        pool.execute(|| {
            handle_connection(stream);
        });
    }
}

fn handle_connection(mut stream: TcpStream) {
    let mut buffer = [0; 512];
    stream.read(&mut buffer).unwrap();

    let get = b"GET / HTTP/1.1\r\n";
    let sleep = b"GET /sleep HTTP/1.1\r\n";

    let (status_line, filename) = if buffer.starts_with(get) {
        ("HTTP/1.1 200 OK\r\n\r\n", "hello.html")
    } else if buffer.starts_with(sleep) {
        thread::sleep(Duration::from_secs(5));
        ("HTTP/1.1 200 OK\r\n\r\n", "hello.html")
    } else {
        ("HTTP/1.1 404 NOT FOUND\r\n\r\n", "404.html")
    };

     let mut file = File::open(filename).unwrap();
     let mut contents = String::new();

     file.read_to_string(&mut contents).unwrap();

     let response = format!("{}{}", status_line, contents);

     stream.write(response.as_bytes()).unwrap();
     stream.flush().unwrap();
}
```

<span class="filename">Filename: src/lib.rs</span>

```rust
use std::thread;
use std::sync::mpsc;
use std::sync::Arc;
use std::sync::Mutex;

enum Message {
    NewJob(Job),
    Terminate,
}

pub struct ThreadPool {
    workers: Vec<Worker>,
    sender: mpsc::Sender<Message>,
}

trait FnBox {
    fn call_box(self: Box<Self>);
}

impl<F: FnOnce()> FnBox for F {
    fn call_box(self: Box<F>) {
        (*self)()
    }
}

type Job = Box<FnBox + Send + 'static>;

impl ThreadPool {
    /// Create a new ThreadPool.
    ///
    /// The size is the number of threads in the pool.
    ///
    /// # Panics
    ///
    /// The `new` function will panic if the size is zero.
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let (sender, receiver) = mpsc::channel();

        let receiver = Arc::new(Mutex::new(receiver));

        let mut workers = Vec::with_capacity(size);

        for id in 0..size {
            workers.push(Worker::new(id, receiver.clone()));
        }

        ThreadPool {
            workers,
            sender,
        }
    }

    pub fn execute<F>(&self, f: F)
        where
            F: FnOnce() + Send + 'static
    {
        let job = Box::new(f);

        self.sender.send(Message::NewJob(job)).unwrap();
    }
}

impl Drop for ThreadPool {
    fn drop(&mut self) {
        println!("Sending terminate message to all workers.");

        for _ in &mut self.workers {
            self.sender.send(Message::Terminate).unwrap();
        }

        println!("Shutting down all workers.");

        for worker in &mut self.workers {
            println!("Shutting down worker {}", worker.id);

            if let Some(thread) = worker.thread.take() {
                thread.join().unwrap();
            }
        }
    }
}

struct Worker {
    id: usize,
    thread: Option<thread::JoinHandle<()>>,
}

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Message>>>) ->
        Worker {

        let thread = thread::spawn(move ||{
            loop {
                let message = receiver.lock().unwrap().recv().unwrap();

                match message {
                    Message::NewJob(job) => {
                        println!("Worker {} got a job; executing.", id);

                        job.call_box();
                    },
                    Message::Terminate => {
                        println!("Worker {} was told to terminate.", id);

                        break;
                    },
                }
            }
        });

        Worker {
            id,
            thread: Some(thread),
        }
    }
}
```

There's more we could do here! If you'd like to continue enhancing this
project, here are some ideas:

- Add more documentation to `ThreadPool` and its public methods
- Add tests of the library's functionality
- Change calls to `unwrap` to more robust error handling
- Use `ThreadPool` to perform some other task rather than serving web requests
- Find a thread pool crate on crates.io and implement a similar web server
  using the crate instead and compare its API and robustness to the thread pool
  we implemented

## Summary

Well done! You've made it to the end of the book! We'd like to thank you for
joining us on this tour of Rust. You're now ready to go out and implement your
own Rust projects or help with other people's. Remember there's a community of
other Rustaceans who would love to help you with any challenges you encounter
on your Rust journey.
