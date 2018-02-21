## Aceptando Argumentos de Línea de Comandos

Vamos a crear un nuevo proyecto con, como siempre, `cargo new`. Llamaremos a nuestro proyecto 
`minigrep` para distinguirlo de la herramienta `grep` que ya tienes 
en tu sistema.

```text
$ cargo new --bin minigrep
     Created binary (application) `minigrep` project
$ cd minigrep
```

La primera tarea es hacer que `minigrep` acepte sus dos argumentos de línea de comandos: el
nombre de archivo y una cadena para buscar. Es decir, queremos ser capaces de ejecutar nuestro
programa con `cargo run`, una cadena de caracteres a buscar, y una ruta a un fichero para 
buscar, así:

```text
$ cargo run searchstring example-filename.txt
```

En este momento, el programa generado por `cargo new` no puede procesar los argumentos que
le demos. Sin embargo, algunas bibliotecas existentes en [Crates.io](https://crates.io/)
pueden ayudarnos a escribir un programa que acepte argumentos de línea de comandos, pero
como tu estás aprendiendo este concepto, vamos a implementar esta capacidad
nosotros mismos.

### Lectura de los Valores Argumentales

Para asegurarnos de que `minigrep` sea capaz de leer los valores de los argumentos de línea de comandos 
que le pasamos, necesitaremos una función incluida en la biblioteca estándar de Rust, que es 
`std::env::args`. Esta función devuelve un *iterador* de los argumentos de la línea de comandos
que fueron dados a `minigrep`. Todavía no hemos discutido los iteradores (los cubriremos 
completamente en el capítulo 13), pero por ahora sólo necesitas saber dos detalles
sobre iteradores: los iteradores producen una serie de valores, y podemos llamar la
función `collect` a un iterador para convertirlo en una colección, como un vector,
que contiene todos los elementos que produce el iterador.

Utiliza el código en el Listado 12-1 para permitir que tu programa `minigrep` lea cualquier
argumento de línea de comandos que le pases y luego reúne los valores en un vector:

<span class="filename">Nombre del archivo: src/main.rs</span>

```rust
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();
    println!("{:?}", args);
}
```

<span class="caption">Listado 12-1: Recopilar los argumentos de la línea de comandos en 
un vector e imprimirlos</span>

Primero, ponemos el módulo `std::env` en scope con una declaración `use` para que
podamos usar la función `args`. Observa que la función `std::env::args` está 
anidada en dos niveles de módulos. Como hemos comentado en el capítulo 7, en los casos en
los que la función deseada está anidada en más de un módulo, es convencional incluir
el scope del módulo padre en lugar de la función. Como resultado, podemos utilizar
fácilmente otras funciones de `std::env`. También es menos ambiguo que añadir 
`use std::env::args` y luego llamar a la función sólo con `args` porque `args`
podría confundirse fácilmente con una función definida en el módulo
actual.

> ### La Función `args` y el Unicode Inválido
>
> Ten en cuenta que `std::env::args` entrará en pánico si algún argumento contiene 
> Unicode inválido. Si tu programa necesita aceptar argumentos que contengan Unicode
> inválido, usa `std::env::args_os` en su lugar. Esta función devuelve valores `OsString` 
> en lugar de valores `String`. Hemos elegido usar `std::env::args` aquí para simplificar
> porque los valores `OsString` difieren por plataforma y son más complejos de
> trabajar que los valores `String`.

En la primera línea de `main`, llamamos `env::args`, e inmediatamente utilizamos `collect`
para convertir el iterador en un vector que contiene todos los valores producidos por el 
iterador. Podemos utilizar la función `collect` para crear muchos tipos de 
colecciones, por lo que anotamos explícitamente el tipo de `args` para especificar que
queremos un vector de cadenas. Aunque en muy raras ocasiones necesitamos anotar tipos en 
Rust, `collect` es una función que a menudo necesitas anotar porque Rust no es capaz 
de inferir el tipo de colección que deseas.

Finalmente, imprimimos el vector usando el formateador de depuración, `:?` Tratemos de ejecutar
el código sin argumentos, y luego con dos argumentos:

```text
$ cargo run
--snip--
["target/debug/minigrep"]

$ cargo run needle haystack
--snip--
["target/debug/minigrep", "needle", "haystack"]
```

Ten en cuenta que el primer valor en el vector es `"target/debug/minigrep"`, que es el
nombre de nuestro binario. Esto coincide con el comportamiento de la lista de argumentos
en C, permitiendo a los programas usar el nombre con el que fueron invocados en su ejecución.
A menudo es conveniente tener acceso al nombre del programa en caso de que queramos 
imprimirlo en mensajes o cambiar el comportamiento del programa basado en el alias de la
línea de comandos que se usó para invocar el programa. Pero para los propósitos de este
capítulo, lo ignoraremos y sólo guardaremos los dos argumentos que necesitamos.

### Guardando los Valores Argumentales en Variables

Al imprimir el valor del vector de argumentos se ilustró que el programa puede
acceder a los valores especificados como argumentos de línea de comandos. Ahora necesitamos 
guardar los valores de los dos argumentos en variables para que podamos usar los valores
en el resto del programa. Lo hacemos en el listado 12-2:

<span class="filename">Nombre del archivo: src/main.rs</span>

```rust,should_panic
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();

    let query = &args[1];
    let filename = &args[2];

    println!("Buscando {}", consulta);
    println!("En el Archivo {}", nombre del archivo);
}
```

<span class="caption">Listado 12-2: Creación de variables para contener el
 argumento de consulta y el argumento de nombre de archivo</span>

Como vimos cuando imprimimos el vector, el nombre del programa toma el primer 
valor del vector en `args[0]`, así que estamos comenzando en el índice `1`. El primer
argumento que toma `minigrep` es la cadena que estamos buscando, así que ponemos
una referencia al primer argumento en la variable `query`. El segundo argumento 
será el nombre del archivo, por lo que pondremos una referencia al segundo argumento en 
la variable `filename`.

Imprimimos temporalmente los valores de estas variables para demostrar que el código está 
funcionando como pretendemos. Vamos a ejecutar este programa de nuevo con los argumentos 
`test` y `sample.txt`:

```text
$ cargo run test sample.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/minigrep test sample.txt`
Searching for test
In file sample.txt
```

Genial, el programa está funcionando! Los valores de los argumentos que necesitamos están
siendo guardados en las variables correctas. Más adelante añadiremos el manejo de errores para 
tratar ciertas situaciones potencialmente erróneas, como cuando el usuario no proporciona 
argumentos; por ahora, ignoraremos esa situación y trabajemos en añadir capacidades de 
lectura de archivos.
