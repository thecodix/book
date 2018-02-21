## Organización de prueba

Como se mencionó al comienzo del capítulo, las pruebas son una disciplina compleja, y
diferentes personas las manejan con diferente terminología y organización. La comunidad de Rust
piensa en las pruebas en términos de dos categorías principales: *pruebas unitarias* y
en aislamiento a la vez, y puede probar interfaces privadas. 
Las pruebas de integración son completamente externas a su biblioteca y debe usar 
su código de la misma manera que cualquier otro
código externo, usando solo la interfaz pública y potencialmente ejerciendo
múltiples módulos por prueba.

Escribir ambos tipos de pruebas es importante para asegurar que las piezas de 
la biblioteca están haciendo lo que espera que hagan por separado y juntos.

### Pruebas Unitarias

El propósito de las pruebas unitarias es probar cada unidad de código aisladamente del
resto del código para identificar rápidamente dónde está el código y en donde no está funcionando como
se está esperando. Ponemos pruebas unitarias en el directorio *src* en cada archivo con el código
que están probando. La convención es que creamos un módulo llamado `tests`
en cada archivo para contener las funciones de prueba, y anotamos el módulo con
`cfg(test)`.

#### El Módulo de Pruebas y `#[cfg(test)]`

La anotación `#[cfg(test)]` en el módulo de pruebas le dice a Rust que compile y ejecute
el código de prueba solo cuando ejecutamos `cargo test`, pero no cuando ejecutamos `cargo build`.
Esto ahorra tiempo de compilación cuando solo queremos construir la biblioteca y ahorrar espacio
en el artefacto compilado resultante porque las pruebas no están incluidas. Usted
ve que debido a que las pruebas de integración van en un directorio diferente, no necesitan
la anotación `#[cfg(test)]`. Sin embargo, debido a que las pruebas unitarias van en los mismos archivos
como el código, usamos `#[cfg(test)]` para especificar que no deberían incluirse
en el resultado compilado.

Recordemos que cuando generamos el nuevo proyecto `adder` en la primera sección de
este capítulo, Cargo generó este código para nosotros:

<span class="filename">Nombre de archivo: src/lib.rs</span>

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}
```

Este código es el módulo de prueba generado automáticamente. El atributo `cfg`
representa *configuración* y le dice a Rust que el siguiente artículo solo debería
ser incluido dado una cierta opción de configuración. En este caso, la
opción de configuración es `test`, que es proporcionado por Rust para compilar y
ejecutar pruebas. Usando el atributo `cfg`, Cargo compila solo nuestro código de prueba
si ejecutamos activamente las pruebas con `cargo test`. Este incluye cualquier función
de ayuda que podría estar dentro de este módulo, además de las funciones
anotadas con `#[test]`.

#### Prueba de funciones privadas

Existe un debate dentro de la comunidad de prueba sobre si las funciones 
privadas o no deben probarse directamente, y otros idiomas lo hacen difícil o
imposible para probar funciones privadas. Independientemente de la ideología de prueba que
cumpla, las reglas de privacidad de Rust te permiten probar funciones privadas.
Considere el código en el listado 11-12 con la función privada `internal_adder`:

<span class="filename">Nombre de archivo: src/lib.rs</span>

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

<span class="caption">Listado 11-12: Probando una función privada</span>

Tenga en cuenta que la fución `internal_adder` no está marcado como `pub`, porque
las pruebas son solo código Rust y los módulos `tests` son otro módulo, podemos
importarlos y llamar en una prueba `internal_adder` para que esten correctos. Si no piensas
que las funciones privadas deben ser probadas, no hay nada en Rust que te obligue
a hacer esto.000.

### Pruebas de integración

En Rust, las pruebas de integración son completamente externas a tu biblioteca. Usan tu
biblioteca de la misma manera que cualquier otro código, lo que significa que solo pueden ejecutar
funciones que son parte de la API pública de su biblioteca. Su propósito es probar
que muchas partes de tu biblioteca funcionan juntas correctamente. Unidades de código que
trabajan correctamente por sí solos podría tener problemas cuando se están integrado, así que prueba
la cobertura del código integrado también es importante. Para crear integración de
pruebas, primero necesita un directorio *pruebas*.

#### El Directorio *Pruebas*

Creamos un directorio *pruebas* en el nivel superior de nuestro directorio de proyectos, el siguiente
es *src*. Cargo sabe buscar archivos de prueba de integración en este directorio. Nosotros
podemos hacer tantos archivos de prueba como queramos en este directorio, y Cargo
compilará cada uno de los archivos como una caja individual.

Vamos a crear una prueba de integración. Con el código en el listado 11-12 todavía en el
archivo *src/lib.rs*, crea un directorio *pruebas*, crea un nuevo archivo llamado
*tests/integration_test.rs*, e ingrese el código en el listado 11-13:

<span class="filename">Nombre de archivo: tests/integration_test.rs</span>

```rust,ignore
extern crate adder;

#[test]
fn it_adds_two() {
    assert_eq!(4, adder::add_two(2));
}
```

<span class="caption">Listado 11-13: Una prueba de integración de una función en
la caja `adder`</span>

Hemos agregado `extern crate adder` en la parte superior del código, que no necesitamos
en las pruebas unitarias. La razón es que cada prueba en el directorio `tests` es una
caja separada, entonces necesitamos importar nuestra biblioteca en cada uno de ellos.

No es necesario anotar ningún código en *tests / integration_test.rs* con
`#[cfg(test)]`. Cargo trata al directorio `tests` especialmente y compila archivos
en este directorio solo cuando ejecutamos `cargo test`. Ejecuta ahora `cargo test`:

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

Las tres secciones de salida incluyen las pruebas unitarias, la prueba de integración y
las pruebas de doc. La primera sección para las pruebas unitarias es la misma que hemos estado
viendo: una línea para cada prueba de unidad (uno llamada `internal` que agregamos en el
Listado 11-12) y luego una línea de resumen para las pruebas unitarias.

La sección de pruebas de integración comienza con la línea `Running
target/debug/deps/integration-test-ce99bcc2479f4607` (el hash al final de
tu salida será diferente). A continuación, hay una línea para cada función de prueba en
esa prueba de integración y una línea de resumen para los resultados de la integración de
prueba justo antes de que la sección `Doc-tests adder` comience.

Recuerde que agregar más funciones de prueba unitarias en cualquier archivo *src* agrega más pruebas en las
líneas de resultado a la sección de pruebas de la unidad. Agregar más funciones de prueba al
archivo de prueba de integración que creamos agrega más líneas a la sección de ese archivo. Cada
archivo de prueba de integración tiene su propia sección, por lo que si agregamos más archivos en el
directorio *tests*, habrá más secciones de prueba de integración.

Todavía podemos ejecutar una función de prueba de integración particular al especificar la prueba con
el nombre de la función como argumento para `cargo test`. Para ejecutar todas las pruebas en un
archivo particular de prueba de integración, use el argumento `--test` de `cargo test`
seguido por el nombre del archivo:

```text
$ cargo test --test integration_test
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running target/debug/integration_test-952a27e0126bb565

running 1 test
test it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

Este comando ejecuta solo las pruebas en el archivo *tests / integration_test.rs*.

#### Submódulos en Pruebas de Integración

A medida que agrega más pruebas de integración, es posible que desee crear más de un archivo en
el directorio de *tests* para ayudar a organizarlos; por ejemplo, puede agrupar y
probar funciones por la funcionalidad que están probando. Como se mencionó anteriormente, cada
archivo en el directorio *tests* se compila como su propia caja separada.

El tratamiento de cada archivo de prueba de integración como su propia caja es útil para crear
ámbitos separados que se parecen más a la manera en que los usuarios finales usarán su caja.
Sin embargo, esto significa que los archivos en el directorio *tests* no comparten el mismo
comportamiento como archivos en *src*, que aprendió en el Capítulo 7 con respecto a cómo
separar códigos en módulos y archivos.

The different behavior of files in the *tests* directory is most noticeable
when you have a set of helper functions that would be useful in multiple
integration test files and you try to follow the steps in the “Moving Modules
to Other Files” section of Chapter 7 to extract them into a common module. For
example, if we create *tests/common.rs* and place a function named `setup` in
it, we can add some code to `setup` that we want to call from multiple test
functions in multiple test files:

<span class="filename">Filename: tests/common.rs</span>

```rust
pub fn setup() {
    // setup code specific to your library's tests would go here
}
```

When we run the tests again, we’ll see a new section in the test output for the
*common.rs* file, even though this file doesn’t contain any test functions, nor
did we call the `setup` function from anywhere:

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

Having `common` appear in the test results with `running 0 tests` displayed for
it is not what we wanted. We just wanted to share some code with the other
integration test files.

To avoid having `common` appear in the test output, instead of creating
*tests/common.rs*, we’ll create *tests/common/mod.rs*. In the “Rules of Module
Filesystems” section of Chapter 7, we used the naming convention
*module_name/mod.rs* for files of modules that have submodules, and we don’t
have submodules for `common` here, but naming the file this way tells Rust not
to treat the `common` module as an integration test file. When we move the
`setup` function code into *tests/common/mod.rs* and delete the
*tests/common.rs* file, the section in the test output will no longer appear.
Files in subdirectories of the *tests* directory don’t get compiled as separate
crates or have sections in the test output.

After we’ve created *tests/common/mod.rs*, we can use it from any of the
integration test files as a module. Here’s an example of calling the `setup`
function from the `it_adds_two` test in *tests/integration_test.rs*:

<span class="filename">Filename: tests/integration_test.rs</span>

```rust,ignore
extern crate adder;

mod common;

#[test]
fn it_adds_two() {
    common::setup();
    assert_eq!(4, adder::add_two(2));
}
```

Note that the `mod common;` declaration is the same as the module declarations
we demonstrated in Listing 7-4. Then in the test function, we can call the
`common::setup()` function.

#### Integration Tests for Binary Crates

If our project is a binary crate that only contains a *src/main.rs* file and
doesn’t have a *src/lib.rs* file, we can’t create integration tests in the
*tests* directory and use `extern crate` to import functions defined in the
*src/main.rs* file. Only library crates expose functions that other crates can
call and use; binary crates are meant to be run on their own.

This is one of the reasons Rust projects that provide a binary have a
straightforward *src/main.rs* file that calls logic that lives in the
*src/lib.rs* file. Using that structure, integration tests *can* test the
library crate by using `extern crate` to exercise the important functionality.
If the important functionality works, the small amount of code in the
*src/main.rs* file will work as well, and that small amount of code doesn’t
need to be tested.

## Summary

Rust’s testing features provide a way to specify how code should function to
ensure it continues to work as we expect even as we make changes. Unit tests
exercise different parts of a library separately and can test private
implementation details. Integration tests check that many parts of the library
work together correctly, and they use the library’s public API to test the code
in the same way external code will use it. Even though Rust’s type system and
ownership rules help prevent some kinds of bugs, tests are still important to
help reduce logic bugs having to do with how your code is expected to behave.

Let’s combine the knowledge you learned in this chapter and in previous
chapters and work on a project in the next chapter!

