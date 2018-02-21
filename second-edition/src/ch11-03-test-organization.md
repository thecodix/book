## Test Organization

As mentioned at the start of the chapter, testing is a complex discipline, and
different people use different terminology and organization. The Rust community
thinks about tests in terms of two main categories: *unit tests* and
*integration tests*. Unit tests are small and more focused, testing one module
in isolation at a time, and can test private interfaces. Integration tests are
entirely external to your library and use your code in the same way any other
external code would, using only the public interface and potentially exercising
multiple modules per test.

Writing both kinds of tests is important to ensure that the pieces of your
library are doing what you expect them to separately and together.

### Unit Tests

The purpose of unit tests is to test each unit of code in isolation from the
rest of the code to quickly pinpoint where code is and isn’t working as
expected. We put unit tests in the *src* directory in each file with the code
that they’re testing. The convention is that we create a module named `tests`
in each file to contain the test functions, and we annotate the module with
`cfg(test)`.

#### The Tests Module and `#[cfg(test)]`

The `#[cfg(test)]` annotation on the tests module tells Rust to compile and run
the test code only when we run `cargo test`, but not when we run `cargo build`.
This saves compile time when we only want to build the library and saves space
in the resulting compiled artifact because the tests are not included. You’ll
see that because integration tests go in a different directory, they don’t need
the `#[cfg(test)]` annotation. However, because unit tests go in the same files
as the code, we use `#[cfg(test)]` to specify that they shouldn’t be included
in the compiled result.

Recall that when we generated the new `adder` project in the first section of
this chapter, Cargo generated this code for us:

<span class="filename">Filename: src/lib.rs</span>

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}
```

This code is the automatically generated test module. The attribute `cfg`
stands for *configuration* and tells Rust that the following item should only
be included given a certain configuration option. In this case, the
configuration option is `test`, which is provided by Rust for compiling and
running tests. By using the `cfg` attribute, Cargo compiles our test code only
if we actively run the tests with `cargo test`. This includes any helper
functions that might be within this module, in addition to the functions
annotated with `#[test]`.

#### Testing Private Functions

There’s debate within the testing community about whether or not private
functions should be tested directly, and other languages make it difficult or
impossible to test private functions. Regardless of which testing ideology you
adhere to, Rust’s privacy rules do allow you to test private functions.
Consider the code in Listing 11-12 with the private function `internal_adder`:

<span class="filename">Filename: src/lib.rs</span>

```rust
pub fn add_two(a: i32) -> i32 {
    internal_adder(a, 2)
}

fn internal_adder(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn internal() {
        assert_eq!(4, internal_adder(2, 2));
    }
}
```

<span class="caption">Listing 11-12: Testing a private function</span>

Note that the `internal_adder` function is not marked as `pub`, but because
tests are just Rust code and the `tests` module is just another module, we can
import and call `internal_adder` in a test just fine. If you don’t think
private functions should be tested, there’s nothing in Rust that will compel
you to do so.

### Integration Tests

In Rust, integration tests are entirely external to your library. They use your
library in the same way any other code would, which means they can only call
functions that are part of your library’s public API. Their purpose is to test
that many parts of your library work together correctly. Units of code that
work correctly on their own could have problems when integrated, so test
coverage of the integrated code is important as well. To create integration
tests, you first need a *tests* directory.

#### The *tests* Directory

We create a *tests* directory at the top level of our project directory, next
to *src*. Cargo knows to look for integration test files in this directory. We
can then make as many test files as we want to in this directory, and Cargo
will compile each of the files as an individual crate.

Let’s create an integration test. With the code in Listing 11-12 still in the
*src/lib.rs* file, make a *tests* directory, create a new file named
*tests/integration_test.rs*, and enter the code in Listing 11-13:

<span class="filename">Filename: tests/integration_test.rs</span>

```rust,ignore
extern crate adder;

#[test]
fn it_adds_two() {
    assert_eq!(4, adder::add_two(2));
}
```

<span class="caption">Listing 11-13: An integration test of a function in the
`adder` crate</span>

We’ve added `extern crate adder` at the top of the code, which we didn’t need
in the unit tests. The reason is that each test in the `tests` directory is a
separate crate, so we need to import our library into each of them.

We don’t need to annotate any code in *tests/integration_test.rs* with
`#[cfg(test)]`. Cargo treats the `tests` directory specially and compiles files
in this directory only when we run `cargo test`. Run `cargo test` now:

```text
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31 secs
     Running target/debug/deps/adder-abcabcabc

running 1 test
test tests::internal ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

     Running target/debug/deps/integration_test-ce99bcc2479f4607

running 1 test
test it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

The three sections of output include the unit tests, the integration test, and
the doc tests. The first section for the unit tests is the same as we’ve been
seeing: one line for each unit test (one named `internal` that we added in
Listing 11-12) and then a summary line for the unit tests.

The integration tests section starts with the line `Running
target/debug/deps/integration-test-ce99bcc2479f4607` (the hash at the end of
your output will be different). Next, there is a line for each test function in
that integration test and a summary line for the results of the integration
test just before the `Doc-tests adder` section starts.

Recall that adding more unit test functions in any *src* file adds more test
result lines to the unit tests section. Adding more test functions to the
integration test file we created adds more lines to that file’s section. Each
integration test file has its own section, so if we add more files in the
*tests* directory, there will be more integration test sections.

We can still run a particular integration test function by specifying the test
function’s name as an argument to `cargo test`. To run all the tests in a
particular integration test file, use the `--test` argument of `cargo test`
followed by the name of the file:

```text
$ cargo test --test integration_test
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running target/debug/integration_test-952a27e0126bb565

running 1 test
test it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

This command runs only the tests in the *tests/integration_test.rs* file.

#### Submodules in Integration Tests

As you add more integration tests, you might want to make more than one file in
the *tests* directory to help organize them; for example, you can group the
test functions by the functionality they’re testing. As mentioned earlier, each
file in the *tests* directory is compiled as its own separate crate.

Treating each integration test file as its own crate is useful to create
separate scopes that are more like the way end users will be using your crate.
However, this means files in the *tests* directory don’t share the same
behavior as files in *src* do, which you learned in Chapter 7 regarding how to
separate code into modules and files.

El comportamiento diferente de los archivos en el directorio *tests* es más notable
cuando tienes un conjunto de funciones auxiliares que serían útiles en múltiples
archivos de prueba de integración e intenta seguir los pasos en "Mover módulos"
a otros archivos” sección del Capítulo 7 para extraerlos en un módulo común. por
ejemplo, si creamos *tests / common.rs* y colocamos una función llamada `setup` en
el, podemos agregar un código a `setup` que queremos llamar desde una prueba múltiples
funciones en múltiples archivos de prueba:

<span class="filename">Nombre de archivo: tests/common.rs</span>

```rust
pub fn setup() {
    // setup code specific to your library's tests would go here
}
```

Cuando ejecutemos las pruebas nuevamente, veremos una nueva sección en la salida de prueba para el
archivo *common.rs*, a pesar de que este archivo no contiene ninguna función de prueba, ni
llamamos a la función `setup` desde donde sea:

```text
running 1 test
test tests::internal ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

     Running target/debug/deps/common-b8b07b6f1be2db70

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

     Running target/debug/deps/integration_test-d993c68b431d39df

running 1 test
test it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

Tener `common` aparece en los resultados de la prueba con `running 0 tests` mostrado para
lo que no queríamos. Solo queríamos compartir un código con el otro
archivo de prueba de integración.

Para evitar tener `common` en la salida de prueba, en lugar de crear
*tests/common.rs*, crearemos *tests/common/mod.rs*. En las "Reglas del Módulo
archivos del sistema” sección del Capítulo 7, utilizamos la convención de nomenclatura
*module_name/mod.rs* para archivos de módulos que tienen submódulos, y nosotros no
tenemos submódulos para `common` aquí, pero nombrar el archivo de esta manera le dice a Rust que no
tratemos el archivo `common` como un archivo de prueba de integración. Cuando movemos la
función del código `setup` en *tests/common/mod.rs* y eliminamos el
archivo *tests/common.rs*, la sección en la salida de prueba ya no aparecerá.
Los archivos en subdirectorios del directorio * tests * no se compilan por separado en
cajas o tienen secciones en la salida de prueba.

Después de crear *tests / common / mod.rs*, podemos usarlo desde cualquiera de
archivos de prueba de integración como un módulo. Aquí hay un ejemplo de llamar en  la función 
'setup' de la prueba `it_adds_two` en *tests/integration_test.rs*:

<span class="filename">Nombre de archivo: tests/integration_test.rs</span>

```rust,ignore
extern crate adder;

mod common;

#[test]
fn it_adds_two() {
    common::setup();
    assert_eq!(4, adder::add_two(2));
}
```

Nótese que la declaración `mod common;` es lo mismo que las declaraciones del módulo
que demostramos en el Listado 7-4. Luego, en la función de prueba, podemos llamar a la
función `common::setup()`.

#### Pruebas de integración para cajas binarias

Si nuestro proyecto es una caja binaria que solo contiene un archivo *src/main.rs* y
no tiene un archivo *src/lib.rs*, no podemos crear pruebas de integración en el
directorio *tests* y usar el `extern crate` para importar funciones definidas en
el archivo *src/main.rs*. Solo las cajas de la biblioteca exponen funciones que otras cajas pueden
llamar y usar; las cajas binarias se deben ejecutar por sí mismas.

Esta es una de las razones por las cuales los proyectos de Rust que proporcionan un binario tienen una
archivo sencillo *src/main.rs* que llama a la lógica que está en el
archivo *src/lib.rs*. Usando esa estructura, las pruebas de integración * pueden * probar la
caja de biblioteca mediante el uso de `extern crate` para ejercer la funcionalidad importante.
Si la funcionalidad importante funciona, la pequeña cantidad de código en el
archivo *src/main.rs* funcionará también, y esa pequeña cantidad de código no
necesita ser probada.

## Resúmen

Las características de prueba de Rust proporcionan una manera de especificar cómo debe funcionar el código para
asegúrarte de que siga funcionando como esperamos incluso mientras hacemos cambios. Pruebas unitarias
ejercen diferentes partes de una biblioteca por separado y pueden probar implementación
de detalles privados. Las pruebas de integración comprueban que muchas partes de la biblioteca
trabajan juntos correctamente y usan la API pública de la biblioteca para probar el código
de la misma forma que el código externo lo usará. Aunque el sistema de tipos de Rust y
las reglas de propiedad ayudan a prevenir algunos tipos de errores, las pruebas siguen siendo importantes para
ayudan a reducir los errores de lógica que tienen que ver con cómo se espera que su código se comporte.

¡Combinemos el conocimiento que aprendió en este capítulo y en los anteriores
capítulos y trabajemos en un proyecto en el próximo capítulo!

