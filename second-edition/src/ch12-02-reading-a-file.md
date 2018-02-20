## Leyendo un Archivo

Ahora añadiremos funcionalidad para leer el archivo especificado en el argumento de la 
línea de comandos `filename`. Primero, necesitamos un archivo de muestra para probarlo con: 
el mejor tipo de archivo a utilizar para asegurarnos de que `minigrep` está funcionando es uno 
con una pequeña cantidad de texto sobre múltiples líneas con algunas palabras repetidas. El 
Listado 12-3 tiene un poema de Emily Dickinson que funcionará bien! Crea un archivo llamado
*poem.txt* en el nivel raíz de tu proyecto e introduce el poema "No soy nadie!
¿Quién eres tú?"

<span class="filename">Nombre del Archivo: poem.txt</span>

```text
I’m nobody! Who are you?
Are you nobody, too?
Then there’s a pair of us — don’t tell!
They’d banish us, you know.

How dreary to be somebody!
How public, like a frog
To tell your name the livelong day
To an admiring bog!
```

<span class="caption">Listado 12-3: Un poema de Emily Dickinson será un buen
caso de prueba.</span>

Con el texto en su lugar, edita *src/main.rs* y añade código para abrir el archivo, como 
se muestra en Listado 12-4:

<span class="filename">Nombre del archivo: src/main.rs</span>

```rust,should_panic
use std::env;
use std::fs::File;
use std::io::prelude::*;

fn main() {
#     let args: Vec<String> = env::args().collect();
#
#     let query = &args[1];
#     let filename = &args[2];
#
#     println!("Searching for {}", query);
    // --snip--
    println!("In file {}", filename);

    let mut f = File::open(filename).expect("file not found");

    let mut contents = String::new();
    f.read_to_string(&mut contents)
        .expect("something went wrong reading the file");

    println!("With text:\n{}", contents);
}
```

<span class="caption">Listado 12-4: Lectura del contenido del archivo especificado
por el segundo argumento</span>

En primer lugar, añadimos algunas declaraciones `use` más para traer partes relevantes de la
biblioteca estándar: necesitamos `std::fs::File` para manejar archivos, y 
`std::io::prelude::*` contiene varios rasgos útiles para hacer E/S, incluyendo
el archivo E/S. De la misma manera que Rust tiene un preludio general que pone automáticamente
a tu alcance ciertos tipos y funciones, el módulo `std::io` tiene su propio preludio
de los tipos y funciones comunes que necesitarás cuando trabajes con E/S. A diferencia 
del preludio por defecto, debemos añadir explícitamente una declaración de  `use` para el preludio 
de `std::io`.

En `main`, hemos añadido tres enunciados: primero, obtenemos un control mutable del 
archivo llamando a la función `File::open` y pasándole el valor de la variable 
`filename`. En segundo lugar, creamos una variable llamada `contents` y la ponemos en
una `String` vacía y mutable. Esto retendrá el contenido del archivo después de que lo 
hayamos leído. Tercero, llamamos `read_to_string` en nuestro gestor de archivos y pasamos
una referencia mutable a `contents` como argumento.

Después de esas líneas, volvimos a añadir una declaración temporal `println!` que 
imprime el valor de `contents` después de que se lea el archivo, así podemos comprobar
que el programa está funcionando hasta el momento.

Ejecutemos este código con cualquier cadena como primer argumento de línea de comandos
(porque aún no hemos implementado la parte de búsqueda) y el archivo *poem.txt* como
segundo argumento:

```text
$ cargo run the poem.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/minigrep the poem.txt`
Searching for the
In file poem.txt
With text:
I’m nobody! Who are you?
Are you nobody, too?
Then there’s a pair of us — don’t tell!
They’d banish us, you know.

How dreary to be somebody!
How public, like a frog
To tell your name the livelong day
To an admiring bog!
```

¡Genial! El código se leyó e imprimió el contenido del archivo. Pero el código tiene
algunos defectos. La función `main` tiene múltiples responsabilidades: en general, 
las funciones son más claras y fáciles de mantener si cada función es responsable
de una sola idea. El otro problema es que no estamos manejando los errores tan bien
como podríamos hacerlo. El programa sigue siendo pequeño, por lo que estos defectos no
son un gran problema, pero a medida que el programa crezca, será más difícil corregirlos limpiamente.
Es una buena práctica empezar a refactorizar tempranamente cuando se desarrolla un programa, 
porque es mucho más fácil refactorizar cantidades más pequeñas de código. Lo haremos después.
