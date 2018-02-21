## Como Los Requerimientos Lentos Afectan El Rendimiento 

Ahora mismo, el servidor procesará cada requerimiento en turno. Este funciona para
servicios como el nuestros que no esperan recibir muchos requerimientos, pero como 
las aplicaciones se vuelven mas complejas, este tipo de ejecuciones en serie no es óptima.

Debido a que nuestro programa actual procesa las conexiones secuencialmente, este no 
procesará una segunda conexión hasta que se haya completado el procesamiento de la primera. Si nosotros 
tenemos un requerimiento que tome mucho tiempo procesar, los requerimientos recibidos durante 
ese tiempo tendrán que esperar hasta que es terminado el requerimiento largo, aún si el nuevo 
requerimiento pueda ser procesado rápidamente. Veamos esto en acción. 

### Simulación de un Requerimiento Lento en la Implementación del Servidor Actual 

Veamos el efecto de un requerimiento que tome un largo tiempo para procesar en requerimientos 
hechos en nuestra implementacion del servidor actual. El listado 20-10 muestra el código para  
responder a otro requerimiento,`/sleep`, este causará que el servidor se duerma por
cinco segundos antes de responder. Esto simulará un requerimiento lento para que nosotros
podamos ver que nuestros servidor procesa los requerimientos en serie.

<span class="filename">Filename: src/main.rs</span>

```rust
use std::thread;
use std::time::Duration;
# use std::io::prelude::*;
# use std::net::TcpStream;
# use std::fs::File;
// --snip--

fn handle_connection(mut stream: TcpStream) {
#     let mut buffer = [0; 512];
#     stream.read(&mut buffer).unwrap();
    // --snip--

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

    // --snip--
}
```

<span class="caption">Listing 20-10: Simulating a slow request by recognizing
`/sleep` and sleeping for 5 seconds</span>

Este código es un poco desordenado, pero es suficientemente bueno para nuestros propósitos de simulación! Nosotros hemos
creado un segundo requerimiento `sleep`, cuya data nosotros reconoceremos. Nosotros añadimos un `else
if` despues del bloque `if` para verificar que el requerimiento para `/sleep`, y cuando nosotros vemos      
ese requerimiento, Nosotros dormiremos por cinco segundos antes de reproducir la pagina de saludo.

Usted puede realmente ver cuan primitivo es nuestro servidor aquí;las librerías reales podrían 
manejar el reconocimiento de múltiples requerimientos en una manera menos verbosa!

Inicie el servidor con `cargo run`, y luego abra dos ventanas del navegador: una
para `http://localhost:8080/` y una para `http://localhost:8080/sleep`. Si
Usted toca unas pocas veces `/`, como antes, Usted verá que responderá rápidamente. Pero si Ud
toca `/sleep`, y luego carga `/`, Ud. verá que `/` espera hasta `sleep`
ha dormido por sus cinco segundos completos antes de continuar.

Existen multiples vías en que nosotros podríamos cambiar como funciones nuestro servidor web con la finalidad de 
evitar tener todos los requerimientos de respaldo detrás de un requerimiento lento; el que vamos a 
implementar es un grupo de subprocesos.

### Mejorando el Rendimiento con un Grupo de Subprocesos

Un *grupo de Subprocesos* es un grupo de procesos generados que estan listos para manejar algunas
tareas. Cuando el programa recibe una nueva tarea, uno de los hilos del grupo será
asignado a la tarea e irá y la procesará. El resto
de los subprocesos estan disponibles para manejar cualquier otra tarea que venga mientras el primero
esta procesando. Cuendo el primer hilo ha terminado de procesar la tarea, el
regresa al grupo de subprocesos inactivos listo para manejar la próxima tarea.

Un grupo de sibprocesos nos permitirá procesar conexiones al mismo tiempo: nosotros podemos iniciar 
el procesamiento de una nueva conexión antes que una conexión mas antigua este terminada. Esto
incrementa el rendimiento de nuestro servidor.

Aqui esta lo que vamos a implementar: en lugar de esperar a que cada requerimiento sea
procesado antes de empezar con el siguiente, nosotros enviaremos el procesamiento de cada
conexión a un subgrupo diferente. Los subgrupos vendrán de un grupo de cuatro 
subgrupos que nosotros engendraremos cuando nosotros iniciemos nuestro programa. La razón por la cual nosotros estamos limitando
el número de subgrupos a un pequeño número es que si nosotros creamos un nuevo subgrupo
para cada requermimiento mientras los requerimientos lleguen, alguien haciendo diez millones de requerimientos para
nuestro servidor podría crear estragos mediante el uso de todos los recursos de nuestro servidor y
moler el procesamiento de todos los requerimientos a un alto.

Antes que engendrar subgrupos ilimitadamente, nosotros tendremos un número fijo de subgrupos 
esperando en la piscina. A medida que un requerimiento entre, nosotros enviaremos un requerimiento al grupo
para el procesamiento. El grupo mantendrá una cola de requerimientos entrantes. Cada uno 
de los subgrupos en la piscina sacará un requerimiento de esta cola, manejando el
requerimiento, y luego preguntando a la cola por otro requerimiento. Con este diseño, nosotros podemos
procesar `N` requerimientos a la vez, donde `N` es el número de subgrupos. Esto
todavía significa que `N` requerimientos de largo procesamiento pueden causar que los requerimientos de respaldo en la 
cola, pero nosotros hemos incrementado el número de el número de requerimientos de largo procesamientos que podemos manejar
antes de ese puntos desde uno hasta `N`.

Este diseño es una de las muchas vías para mejorar el procesamiento de nuestros servidor web.
Este no es un libro sobre servidores web,aunque, es el único que vamos a 
cubrir. Otras opciones son el modelo tenedor/unión y el modelo I/O asincrónico de un solo subgrupo.
Si Ud. esta interesado en este tópico, Ud deseará leer más sobre
otras soluciones y tratar de implementarlas en Rust; con un lenguaje de bajo nivel
como Rust, todas estas opciones son posibles.
