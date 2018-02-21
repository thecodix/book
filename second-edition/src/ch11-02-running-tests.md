## Controlando Cómo son Ejecutadas las Pruebas 

Así como el `cargo run` compila tu código y luego ejecuta el sistema binario que resulta de la operación,
`cargo test` compila tu código en modo de prueba y corre el sistema binario de prueba
resultante. Puedes especificar unas opciones de lineas de comando para cambiar el comportamiento por defecto del
`cargo test`. Por ejemplo, el comportamiento por defecto del sistema binario producido por
`cargo test` es ejecutar todas las pruebas en paralelo y capturar las respuestas generadas
durante la ejecución de la misma, contando con que la salida va a ser mostrada y haciendo
más fácil el leer lo que ha sido procesado relacionado a los resultados de la prueba.

Algunas opciones de lineas de comando van hacia `cargo test` y otras van al binario que resulta de la
prueba. Para separar estos dos tipos de argumentos, haz una lista de argumentos que
vayan a `cargo test` seguidos por el separador `--` y luego los argumentos que 
van al binario a prueba. El correr `cargo test --help` muestra las opciones que puedes
usar con `cargo test`, y el correr `cargo test -- --help` muestra las opciones
que puedes usar luego de colocar el separador `--`.

### Corriendo Pruebas en Paralelo o de Forma Consecutiva

Cuando ejecutas multiples pruebas, estas se ejecutan en paralelo usando, por defecto, hilos.
Esto significa que las pruebas se terminarán de ejecutar más velozmente para que tengas una retroalimentación más rápida
sobre tu código, si está trabajando o no. Ya que las pruebas están ejecutandose al
mismo tiempo, asegurate de que tus pruebas no dependan la una de la otra o no estén en ningún estado
compartido, incluyendo un ambiente compartido como el directorio de trabajo actual u
otras variables ambientales.

Por ejemplo, digamos que cada una de nuestras pruebas ejecuta algun código que cree un archivo en el disco
llamado *test-output.txt* y escribe algunos datos en ese archivo. Entonces cada prueba lee
los datos en ese archivo y afirma que el archivo contiene un valor particular,
el cual es diferente en cada prueba que se ejecuta. Ya que las pruebas corren al mismo tiempo, una
prueba puede sobreescribir el archivo entre que una prueba escribe y lee el
archivo en cuestión. La segunda prueba entonces fallará, no porque el código sea incorrecto, sino
porque las pruebas interfirieron entre sí mientras se estaban ejecutando en paralelo.
Una solución es asegurar que cada prueba escriba un archivo distinto; otra
solución es ejecutar una prueba a la vez.

Si no quieres correr las pruebas en paralelo o si quieres un control
más detallado sobre el número de hilos usados, puedes enviar el marcador `--test-threads`
y el número de hilos que quieres usar en el sistema binario de la prueba. Observa el
siguiente ejemplo:

```text
$ cargo test -- --test-threads=1
```

Colocamos que el número de hilos de pruebas sea `1`, diciendole al programa que no use ningún
paralelismo en el. El correr las pruebas usando hilos tomará más tiempo que el ejecutarlas
en paralelo, pero así nos aseguraremos de que las mismas no interferiran entre sí si comparten
estado.

### Mostrando la Función de Respuesta

Por defecto, si una prueba es satisfactoria, la biblioteca de pruebas de Rust capturará cualquier cosa que sea emitida a
una salida estandar. Por ejemplo, si llamamos `println!` en una prueba y la misma
es aprobada, no veremos la salida `println!` en el terminal: solamente veremos la 
linea que indica que la prueba aprobó. Si una prueba falla, veremos lo que haya sido
emitido a la salida estandar con el resto del mensaje de fallo en ella.

Como un ejemplo, el Listado 11-10 tiene una función tonta que emite el valor de su
parametro y devuelve 10, así como una prueba que es aprobada y otra que es fallida.

<span class="filename">Nombre del archivo: src/lib.rs</span>

```rust
fn prints_and_returns_10(a: i32) -> i32 {
    println!("I got the value {}", a);
    10
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn this_test_will_pass() {
        let value = prints_and_returns_10(4);
        assert_eq!(10, value);
    }

    #[test]
    fn this_test_will_fail() {
        let value = prints_and_returns_10(8);
        assert_eq!(5, value);
    }
}
```

<span class="caption">Listado 11-10: Prueba para una función que llama a
`println!`</span>

Cuando ejecutamos estas pruebas con `cargo test`, vamos a ver la siguiente respuesta:

```text
running 2 tests
test tests::this_test_will_pass ... ok
test tests::this_test_will_fail ... FAILED

failures:

---- tests::this_test_will_fail stdout ----
        I got the value 8
thread 'tests::this_test_will_fail' panicked at 'assertion failed: `(left == right)`
  left: `5`,
 right: `10`', src/lib.rs:19:8
note: Run with `RUST_BACKTRACE=1` for a backtrace.

failures:
    tests::this_test_will_fail

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out
```

Nota que en ningun lado de esta respuesta vemos a `I got the value 4`, justo lo que 
se emite cuando la prueba que pasa es ejecutada. Esa respuesta ha sido capturada. La
salida de la prueba fallida, `I got the value 8`, aparece en la sección
del resumen de la respuesta de la prueba, el cual también muestra la causa del fallo de la misma.

Si de igual forma queremos ver los valores emitidos para las pruebas que son aprobadas, podemos deshabilitar el
comportamiento de captura de la respuesta usando el marcador `--nocapture`:

```text
$ cargo test -- --nocapture
```

Cuando ejecutamos la prueba del Listado 11-10 de nuevo usando el marcador `--nocapture`,
recibiremos la siguiente respuesta:

```text
running 2 tests
I got the value 4
I got the value 8
test tests::this_test_will_pass ... ok
thread 'tests::this_test_will_fail' panicked at 'assertion failed: `(left == right)`
  left: `5`,
 right: `10`', src/lib.rs:19:8
note: Run with `RUST_BACKTRACE=1` for a backtrace.
test tests::this_test_will_fail ... FAILED

failures:

failures:
    tests::this_test_will_fail

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out
```

Nota que la salida para la prueba y sus resultados son intercalados; la
razón para que esto ocurra es que las pruebas están ejecutandose, al mismo tiempo, en paralelo, como hemos hablado en la 
sección anterior. Intenta usando la opción `--test-threads=1` con el marcador `--nocapture`,
¡Y entonces ve cómo luce la respuesta!

### Ejecutando un Subconjunto de Pruebas por su Nombre

Algunas veces el correr un proceso de prueba puede tomar un largo tiempo. Si estás trabajando en un 
código en un área en particular, querrás ejecutar sólo las pruebas que le pertenezcan únicamente a
ese código para ahorrar tiempo. Puedes escoger cuál prueba ejecutar al enviarle al comando  `cargo test` el nombre 
o nombres de la(s) prueba(s) que quieres ejecutar como argumento.

Para demostrar cómo ejecutar un conjunto de pruebas, crearemos tres de ellas para nuestra
función `add_two`, como se muestra en el Listado 11-11, y escogeremos cuales ejecutar:

<span class="filename">Nombre del archivo: src/lib.rs</span>

```rust
pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn add_two_and_two() {
        assert_eq!(4, add_two(2));
    }

    #[test]
    fn add_three_and_two() {
        assert_eq!(5, add_two(3));
    }

    #[test]
    fn one_hundred() {
        assert_eq!(102, add_two(100));
    }
}
```

<span class="caption">Listado 11-11: Tres pruebas con tres nombres
distintos</span>

Si ejecutamos las pruebas sin enviar ningun argumento en su código, como ya vimos antes, todas 
se ejecutaran en paralelo:

```text
running 3 tests
test tests::add_two_and_two ... ok
test tests::add_three_and_two ... ok
test tests::one_hundred ... ok

test result: ok. 3 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

#### Ejecutando Pruebas en Solitario

Podemos enviar el nombre de cualquier función de prueba a `cargo test` para ejecutar sólo esa:

```text
$ cargo test one_hundred
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running target/debug/deps/adder-06a75b4a1f2515e9

running 1 test
test tests::one_hundred ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 2 filtered out
```

Solo la prueba con el nombre `one_hundred` fue ejecutada; las otras dos no tenían 
el mismo nombre. La respuesta de la prueba nos deja saber que teníamos más pruebas de las que
este comando ejecutó mostrando `2 filtered out` al final de la linea de resumen como muestra de ello.

No podemos especificar de esta manera los nombres de multiples pruebas; solo el primer valor
que se le da `cargo test` será usado. Pero sí hay una manera de ejecutar multiples pruebas.

#### El Filtrat Para Ejecutar Multiples Pruebas

Podemos especificar parte de un nombre de prueba y cualquier otra que coincida en ese valor
será ejecutada. Por ejemplo, ya que dos de nuestras pruebas tienen nombres que contienen `add`, podemos
correr esas dos con el comando `cargo test add`:

```text
$ cargo test add
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running target/debug/deps/adder-06a75b4a1f2515e9

running 2 tests
test tests::add_two_and_two ... ok
test tests::add_three_and_two ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 1 filtered out
```

Este comando ejecutó todas las pruebas con  `add` en el nombre y filtró la 
prueba llamada `one_hundred`. También date cuenta de que el módulo en el que ellas aparecen
se hace parte del nombre de las mismas, así que podemos ejecutar todas las pruebas en un módulo al
filtrar su nombre.

### Ignorando unas Pruebas a Menos de que Sean Especificamente Solicitadas

A veces unas pruebas específicas pueden consumir mucho tiempo al ejecutarse, así que
podrás no querer hacerlas durante la mayoría de las ejecuciones de `cargo test`. En vez de
enlistar como argumentos todas las pruebas que quieres ejecutar, puedes marcar la
prueba que consume más tiempo usando el atributo `ignore` para excluirla de dicha ejecución, como lo muestra
este ejemplo:

<span class="filename">Nombre del archivo: src/lib.rs</span>

```rust
#[test]
fn it_works() {
    assert_eq!(2 + 2, 4);
}

#[test]
#[ignore]
fn expensive_test() {
    // code that takes an hour to run
}
```

Después del comando `#[test]` añadimos a la linea el comando `#[ignore]` de la prueba que queremos excluir. Ahora
cuando ejecutemos nuestras pruebas, `it_works` se ejecuta, pero `expensive_test` no:

```text
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished dev [unoptimized + debuginfo] target(s) in 0.24 secs
     Running target/debug/deps/adder-ce99bcc2479f4607

running 2 tests
test expensive_test ... ignored
test it_works ... ok

test result: ok. 1 passed; 0 failed; 1 ignored; 0 measured; 0 filtered out
```

La función `expensive_test` es enlistada como `ignored`. Si queremos ejecutar sólo
las pruebas que han sido ignoradas, para eso podemos usar el comando `cargo test -- --ignored`:

```text
$ cargo test -- --ignored
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running target/debug/deps/adder-ce99bcc2479f4607

running 1 test
test expensive_test ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 1 filtered out
```

Al controlar cual prueba se ejecuta, puedes asegurarte de que usando el comando `cargo test` 
los resultados se mostrarán rápido. Cuando te encuentres en el punto donde tiene más sentido el chequear los resultados 
de las pruebas que contienen `ignored` en su comando y tienes tiempo para esperar los resultados, puedes ejecutar
el comando`cargo test -- --ignored` en su lugar.
