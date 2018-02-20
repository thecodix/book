## Controlando como las pruebas son ejecutadas

Así como `cargo run` compila tu código y luego ejecuta el binario resultante,
`cargo test` compila tu código en modo de prueba y ejecuta el binario de prueba
resultante. Puedes especificar unas opciones de lineas de comando para cambiar el comportamiento por defecto del
`cargo test`. Por ejemplo, el comportamiento por defecto del binario producido por
`cargo test` es ejecutar todas las pruebas en paralelo y capturar las respuestas generadas
durante la ejecución de la prueba, previniendo que la salida sea mostrada y haciendo
más fácil el leer la salida relacionada a los resultados de la prueba.

Algunas opciones de lineas de comando van hacia `cargo test` y otras van al binario resultante de la
prueba. Para separar estos dos tipos de argumentos, haces una lista de argumentos que
vayan a `cargo test` seguidos por el separador `--` y luego los argumentos que 
van al binario a prueba. El ejecutar `cargo test --help` muestra las opciones que puedes
usar con `cargo test`, y el ejecutar `cargo test -- --help` muestra las opciones
que puedes usar luego del separador `--`.

### Correr pruebas en paralelo o consecutivamente

Cuando ejecutas multiples pruebas, se ejecutan en paralelo usando hilos por defecto.
Esto significa que las pruebas se terminarán de ejuctar más rápido para que tengas una retroalimentación más rápida
sobre si tu código está trabajando o no. Ya que las pruebas están ejecutandose al
mismo tiempo, asegurate de que tus pruebas no dependan la una de la otra o no estén en ningún estado
compartido, incluyendo un ambiente compartido, como el directorio de trabajo actual o
variables ambientales.

Por ejemplo, digamos que cada una de nuestras pruebas ejecuta algun código que crea un archivo en el disco
llamado *test-output.txt* y escribe algunos datos en ese archivo. Entonces cada prueba lee
los datos en ese archivo y afirma que el archivo contiene un valor particular,
el cual es diferente en cada prueba. Ya que las pruebas corren al mismo tiempo, una
prueba puede sobreescribir el archivo entre que una prueba escribe y lee el
archivo. La segunda prueba entonces fallará, no porque el código sea incorrecto, pero
porque las pruebas interfirieron entre sí mientras se ejecutaban en paralelo.
Una solución es asegurar que cada prueba escriba un archivo distinto; otra
solución es ejecutar las pruebas una a la vez.

Si tú no quieres correr las pruebas en paralelo o si quieres un control
más detallado sobre el número de hilos usados, puedes enviar el marcador `--test-threads`
y el número de hilos que quieres usar en el binario de la prueba. Mira el
siguiente ejemplo:

```text
$ cargo test -- --test-threads=1
```

Colocamos que el número de hilos de pruebas sea `1`, diciendole al programa que no use ningún
paralelismo. El correr las pruebas usando un hilo tomará más tiempo que el ejecutarlas
en paralelo, pero las pruebas no interferiran entre sí si comparten
estado.

### Mostrando la función de salida

Por defecto, si una prueba pasa, la biblioteca de pruebas de Rust capturará cualquier cosa emitida a
una salida estandar. Por ejemplo, si llamamos `println!` en una prueba y la prueba
pasa, no veremos la salida `println!` en el terminal: solamente veremos la 
linea que indica que la prueba pasó. Si una prueba falla, veremos lo que haya sido
emitido a la salida estandar con el resto del mensaje de fallo.

Como un ejemplo, el listado 11-10 tiene una función tonta que emite el valor de su
parametro y devuelve 10, así como una prueba que pasa y otra que falla.

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

<span class="caption">Listado 11-10: Prueba para una función que llama
`println!`</span>

Cuando ejecutamos estas pruebas con `cargo test`, veremos la siguiente salida:

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

Nota que en ningun lado de esta salida vemos `I got the value 4`, lo que es lo que 
se emite cuando la prueba que pasa es ejecutada. Esa salida ha sido capturada. La
salida de la prueba que fallo, `I got the value 8`, aparece en la sección
del resumen de la salida de la prueba, el cual también muestra la causa del fallo de la prueba.

Si queremos ver también los valores emitidos para las pruebas que pasan, podemos deshabilitar el
comportamiento de captura de salida usando el marcador `--nocapture`:

```text
$ cargo test -- --nocapture
```

Cuando ejecutamos la prueba del listado 11-10 de nuevo con el marcador `--nocapture`,
veremos la siguiente respuesta:

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

Nota que la salida para la prueba y los resultados de la prueba son intercalados; la
razón de esto es que las pruebas están ejecutandose en paralelo, como hemos hablado en la 
sección anterior. Intenta usar la opción `--test-threads=1` y el marcador `--nocapture`,
¡Y entonces ve cómo luce la salida!

### Ejecutando un subconjunto de pruebas por nombre

Algunas veces, el correr un proceso de prueba puede tomar un largo tiempo. Si estás trabajando en un 
código en un área en particular, querrás ejecutar sólo las pruebas que le pertenezcan a
ese código. Puedes escoger cual prueba ejecutar al pasar  `cargo test` el nombre 
o nombres de la(s) prueba(s) que quieres ejecutar como argumento.

Para demostrar cómo ejecutar un subconjunto de pruebas, crearemos tres pruebas para nuestra
función `add_two`, como se muestra en el listado 11-11, y escogeremos cuales ejecutar:

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
diferentes</span>

Si ejecutamos las pruebas sin pasar ningun argumento, como vimos antes, todas las
pruebas se ejecutaran en paralelo:

```text
running 3 tests
test tests::add_two_and_two ... ok
test tests::add_three_and_two ... ok
test tests::one_hundred ... ok

test result: ok. 3 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

#### Ejecutando pruebas en solitario

Podemos pasar el nombre de cualquier función de prueba a `cargo test` para ejecutar sólo esa prueba:

```text
$ cargo test one_hundred
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running target/debug/deps/adder-06a75b4a1f2515e9

running 1 test
test tests::one_hundred ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 2 filtered out
```

Solo la prueba con el nombre `one_hundred` fue ejecutada; las otras dos pruebas no coincidían
con ese nombre. La salida de la prueba nos deja saber que teníamos más pruebas de las que
este comando ejecuto mostrando `2 filtered out` al final de la linea de resumen.

No podemos especificar los nombres de multiples pruebas de esta manera; solo el primer valor
dado a `cargo test` será usado. Pero hay una manera de ejecutar multiples pruebas.

#### Filtrando para ejecutar multiples pruebas

Podemos especificar parte de un nombre de prueba y cualquier prueba que coincida en ese valor
será ejecutada. Por ejemplo, ya que dos de nuestros nombres de prueba contienen `add`, podemos
ejecutar esas dos al ejecutar `cargo test add`:

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
prueba llamada `one_hundred`. También nota que el módulo en el que las pruebas aparecen
se hace parte del nombre de la prueba, así que podemos ejecutar todas las pruebas en un módulo al
filtrar el nombre del módulo.

### Ignorando unas pruebas a menos de que sean especificamente solicitadas

A veces unas pruebas específicas pueden consumir mucho tiempo al ejecutarse, así que
querrás excluirlas durante la mayoría de las ejecuciones de `cargo test`. En vez de
enlistar como argumentos todas las pruebas que quieres ejecutar, puedes anotar la
prueba que consume tiempo usando el atributo `ignore` para excluirlo, como lo muestra
aquí:

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

Después de `#[test]` añadimos la linea `#[ignore]` a la prueba que queremos excluir. AhoraNow
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

La función `expensive_test` se enlista como `ignored`. Si queremos ejecutar sólo
las pruebas ignoradas, podemos usar `cargo test -- --ignored`:

```text
$ cargo test -- --ignored
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running target/debug/deps/adder-ce99bcc2479f4607

running 1 test
test expensive_test ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 1 filtered out
```

Al controlar cual prueba se ejecuta, puedes asegurarte de que tus resultados `cargo test` 
serán rápidos. Cuando estás en un punto donde tiene más sentido el chequear los resultados 
de las pruebas `ignored` y tienes tiempo para esperar los resultados, puedes ejecutar
`cargo test -- --ignored` en su lugar.
