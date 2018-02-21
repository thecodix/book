## How to Write Tests

Tests are Rust functions that verify that the non-test code is functioning in
the expected manner. The bodies of test functions typically perform these three
actions:

1. Set up any needed data or state
2. Run the code we want to test
3. Assert the results are what we expect

Let’s look at the features Rust provides specifically for writing tests that
take these actions, which include the `test` attribute, a few macros, and the
`should_panic` attribute.

### The Anatomy of a Test Function

At its simplest, a test in Rust is a function that’s annotated with the `test`
attribute. Attributes are metadata about pieces of Rust code; one example is
the `derive` attribute we used with structs in Chapter 5. To change a function
into a test function, we add `#[test]` on the line before `fn`. When we run our
tests with the `cargo test` command, Rust builds a test runner binary that runs
the functions annotated with the `test` attribute and reports on whether each
test function passes or fails.

In Chapter 7, we saw that when we make a new library project with Cargo, a test
module with a test function in it is automatically generated for us. This
module helps us start writing our tests so we don’t have to look up the exact
structure and syntax of test functions every time we start a new project. We
can add as many additional test functions and as many test modules as we want!

We’ll explore some aspects of how tests work by experimenting with the template
test generated for us without actually testing any code. Then we’ll write some
real-world tests that call some code that we’ve written and assert that its
behavior is correct.

Let’s create a new library project called `adder`:

```text
$ cargo new adder
     Created library `adder` project
$ cd adder
```

The contents of the *src/lib.rs* file in your adder library should look like
Listing 11-1:

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

<span class="caption">Listing 11-1: The test module and function generated
automatically by `cargo new`</span>

For now, let’s ignore the top two lines and focus on the function to see how it
works. Note the `#[test]` annotation before the `fn` line: this attribute
indicates this is a test function, so the test runner knows to treat this
function as a test. We could also have non-test functions in the `tests` module
to help set up common scenarios or perform common operations, so we need to
indicate which functions are tests by using the `#[test]` attribute.

The function body uses the `assert_eq!` macro to assert that 2 + 2 equals 4.
This assertion serves as an example of the format for a typical test. Let’s run
it to see that this test passes.

The `cargo test` command runs all tests in our project, as shown in Listing
11-2:

```text
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished dev [unoptimized + debuginfo] target(s) in 0.22 secs
     Running target/debug/deps/adder-ce99bcc2479f4607

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

<span class="caption">Listing 11-2: The output from running the automatically
generated test</span>

Cargo compiled and ran the test. After the `Compiling`, `Finished`, and
`Running` lines is the line `running 1 test`. The next line shows the name
of the generated test function, called `it_works`, and the result of running
that test, `ok`. The overall summary of running the tests appears next. The
text `test result: ok.` means that all the tests passed, and the portion that
reads `1 passed; 0 failed` totals the number of tests that passed or failed.

Because we don’t have any tests we’ve marked as ignored, the summary shows `0
ignored`. We also haven’t filtered the tests being run, so the end of the
summary shows `0 filtered out`. We’ll talk about ignoring and filtering out
tests in the next section, “Controlling How Tests Are Run.”

The `0 measured` statistic is for benchmark tests that measure performance.
Benchmark tests are, as of this writing, only available in nightly Rust. See
Chapter 1 for more information about nightly Rust.

The next part of the test output, which starts with `Doc-tests adder`, is for
the results of any documentation tests. We don’t have any documentation tests
yet, but Rust can compile any code examples that appear in our API
documentation. This feature helps us keep our docs and our code in sync! We’ll
discuss how to write documentation tests in the “Documentation Comments”
section of Chapter 14. For now, we’ll ignore the `Doc-tests` output.

Let’s change the name of our test to see how that changes the test output.
Change the `it_works` function to a different name, such as `exploration`, like
so:

<span class="filename">Filename: src/lib.rs</span>

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn exploration() {
        assert_eq!(2 + 2, 4);
    }
}
```

Then run `cargo test` again. The output now shows `exploration` instead of
`it_works`:

```text
running 1 test
test tests::exploration ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

Let’s add another test, but this time we’ll make a test that fails! Tests fail
when something in the test function panics. Each test is run in a new thread,
and when the main thread sees that a test thread has died, the test is marked
as failed. We talked about the simplest way to cause a panic in Chapter 9,
which is to call the `panic!` macro. Enter the new test, `another`, so your
*src/lib.rs* file looks like Listing 11-3:

<span class="filename">Filename: src/lib.rs</span>

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn exploration() {
        assert_eq!(2 + 2, 4);
    }

    #[test]
    fn another() {
        panic!("Make this test fail");
    }
}
```

<span class="caption">Listing 11-3: Adding a second test that will fail because
we call the `panic!` macro</span>

Run the tests again using `cargo test`. The output should look like Listing
11-4, which shows that our `exploration` test passed and `another` failed:

```text
running 2 tests
test tests::exploration ... ok
test tests::another ... FAILED

failures:

---- tests::another stdout ----
    thread 'tests::another' panicked at 'Make this test fail', src/lib.rs:10:8
note: Run with `RUST_BACKTRACE=1` for a backtrace.

failures:
    tests::another

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out

error: test failed
```

<span class="caption">Listing 11-4: Test results when one test passes and one
test fails</span>

Instead of `ok`, the line `test tests::another` shows `FAILED`. Two new
sections appear between the individual results and the summary: the first
section displays the detailed reason for each test failure. In this case,
`another` failed because it `panicked at 'Make this test fail'`, which happened
on line 10 in the *src/lib.rs* file. The next section lists just the names of
all the failing tests, which is useful when there are lots of tests and lots of
detailed failing test output. We can use the name of a failing test to run just
that test to more easily debug it; we’ll talk more about ways to run tests in
the “Controlling How Tests Are Run” section.

The summary line displays at the end: overall, our test result is `FAILED`.
We had one test pass and one test fail.

Now that you’ve seen what the test results look like in different scenarios,
let’s look at some macros other than `panic!` that are useful in tests.

### Chequear los resultados con el Macro `assert!` 

El comando macro `assert!` que es proporcionado por la biblioteca estándar, es bastante útil cuando quieres
asegurarte de que una condición valora al comando `true`. Le damos al
comando macro `assert!` un argumento que valora un binomio. Si el valor es
`true`, `assert!` no hace nada y la prueba es aprobada. Si el valor es `false`,
El comando macro `assert!` llama al otro comando macro `panic!` , lo que causa que la prueba falle.
El usar el comando macro `assert!` nos ayuda a chequear que nuestro código sea funcional de la
manera que queremos que sea.

En el capítulo 5, en el listado 5-15, usamos una estructura `Rectangle` y un método `can_hold`,
los que se repiten también en el listado 11-5. Pongamos este código en el 
archivo *src/lib.rs* y escribamos algunas pruebas para el mismo código usando el macro `assert!`.

<span class="filename">Nombre del Archivo: src/lib.rs</span>

```rust
#[derive(Debug)]
pub struct Rectangle {
    length: u32,
    width: u32,
}

impl Rectangle {
    pub fn can_hold(&self, other: &Rectangle) -> bool {
        self.length > other.length && self.width > other.width
    }
}
```

<span class="caption">Listado 11-5: Usando la estructura `Rectangle` y su
método `can_hold` en el capítulo 5</span>

El método `can_hold` da un binomio, lo que significa que es un caso de uso perfecto
para el comando macro `assert!`. En el listado 11-6, ya escribimos una prueba que pone a prueba el
método `can_hold` al crear una instancia `Rectangle` que tiene una longitud de 8 y
un ancho de 7, y afirma que puede albergar otra instancia `Rectangle` que 
tiene una longitud de 5 y un ancho de 1:

<span class="filename">Nombre del archivo: src/lib.rs</span>

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn larger_can_hold_smaller() {
        let larger = Rectangle { length: 8, width: 7 };
        let smaller = Rectangle { length: 5, width: 1 };

        assert!(larger.can_hold(&smaller));
    }
}
```

<span class="caption">Listado 11-6: Una prueba para `can_hold` que chequea que un
rectángulo más grande puede albergar dentro de sí un rectángulo más pequeño</span>

Date cuenta de que hemos añadido una nueva linea dentro del módulo `tests`: la linea de comando `use super::*;`.
El módulo `tests` es un módulo rectangular que sigue las reglas usuales de la
visibilidad que cubrimos en el capítulo 7 en la sección de las “Reglas de privacidad”. Ya que
el módulo `tests` es un módulo interno, necesitamos traer el código bajo prueba en el
módulo exterior dentro del perímetro del módulo interno. Aquí usamos un comando global para que
cualquier cosa que definamos en el módulo externo sea disponible para éste modulo `tests`.

Hemos nombrado a nuestra prueba `larger_can_hold_smaller`, y hemos creado las dos instancias
`Rectangle` que necesitabamos. Entonces llamamos al macro `assert!` y
le pasa el resultado de la acción de llamar a `larger.can_hold(&smaller)`. Ésta expresión
supone que devolverá `true`, así que nuestro texto debería ser aprobado. ¡Vamos a averiguarlo!

```text
running 1 test
test tests::larger_can_hold_smaller ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

¡Sí es aprobado! Agreguemos otra prueba, esta vez afirmando que un rectángulo
más pequeño no puede albergar uno más grande:

<span class="filename">Nombre del archivo: src/lib.rs</span>

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn larger_can_hold_smaller() {
        // --snip--
    }

    #[test]
    fn smaller_cannot_hold_larger() {
        let larger = Rectangle { length: 8, width: 7 };
        let smaller = Rectangle { length: 5, width: 1 };

        assert!(!smaller.can_hold(&larger));
    }
}
```

Ya que el resultado correcto de la función `can_hold` es en este caso `false`,
necesitamos negar ese resultado antes de que lo pasemos al macro `assert!`. Como
resultado, nuestra prueba será aprobada solo si `can_hold` da como resultado `false`:

```text
running 2 tests
test tests::smaller_cannot_hold_larger ... ok
test tests::larger_can_hold_smaller ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

¡Dos pruebas que pasan! Ahora veamos qué le pasa a nuestro resultado cuando
introducimos un error intencional en nuestro código. Cambiemos la implementación del método `can_hold`
reemplazando el símbolo mayor que con símbolo de menor que cuando el código
compara las longitudes:

```rust
# #[derive(Debug)]
# pub struct Rectangle {
#     length: u32,
#     width: u32,
# }
// --snip--

impl Rectangle {
    pub fn can_hold(&self, other: &Rectangle) -> bool {
        self.length < other.length && self.width > other.width
    }
}
```

El ejecutar las pruebas producirá lo siguiente:

```text
running 2 tests
test tests::smaller_cannot_hold_larger ... ok
test tests::larger_can_hold_smaller ... FAILED

failures:

---- tests::larger_can_hold_smaller stdout ----
    thread 'tests::larger_can_hold_smaller' panicked at 'assertion failed:
    larger.can_hold(&smaller)', src/lib.rs:22:8
note: Run with `RUST_BACKTRACE=1` for a backtrace.

failures:
    tests::larger_can_hold_smaller

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out
```

¡Nuestras pruebas lograron encontrar el error! Ya que `larger.length` es 8 y `smaller.length` es
5, la comparación de las longitudes en `can_hold` ahora da `false`: en resumen, 8 no es
menor que 5.

### Probando la igualdad con los macros `assert_eq!` y `assert_ne!` 

Una manera común de probar funcionalidad es comparar el resultado del código bajo
prueba al valor que esperamos de resultado para asegurarnos de que son iguales. 
Podríamos hacer esto usando el macro `assert!` y al pasarlo a una expresión usando el 
operador `==` . Sin embargo, Este test es tan común que la biblioteca estandar
provee un par de macros —`assert_eq!` y `assert_ne!`— para realizar este test
de una forma más conveniente. Estos macros comparan dos argumentos para igualidad o
desigualdad, respectivamente, y también emitirán los dos valores si la afirmación
falla, lo que hace más fácil el ver *por qué* la prueba falló; En cambio, el
macro `assert!` sólo indica que tiene un valor `false` para la expresión `==`,
no los valores que llevan al valor `false`.

En el listado 11-7, escribimos una función llamada `add_two` que añada `2` a su
parámetro y devuelva el resultado. Entonces probamos esta función usando el
macro`assert_eq!`.

<span class="filename">Nombre del archivo: src/lib.rs</span>

```rust
pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_adds_two() {
        assert_eq!(4, add_two(2));
    }
}
```

<span class="caption">Listado 11-7: Probando la función `add_two` usando el
macro `assert_eq!`</span>

¡Chequeemos si es aprobada!

```text
running 1 test
test tests::it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

El primer argumento que le dimos al macro `assert_eq!`, `4`, es igual al 
resultado de llamar a `add_two(2)`. La linea de comando para esta prueba es `test
tests::it_adds_two ... ok`, ¡y el texto `ok` indica que nuestra ha sido satisfactoria!

Introduzcamos un error intencional en nuestro código para ver cómo luce cuando una prueba que
usa `assert_eq!` falla. Cambia la implementación de la función `add_two` 
a en cambio un `3`:

```rust
pub fn add_two(a: i32) -> i32 {
    a + 3
}
```

Ejecuta esta prueba de nuevo:

```text
running 1 test
test tests::it_adds_two ... FAILED

failures:

---- tests::it_adds_two stdout ----
        thread 'tests::it_adds_two' panicked at 'assertion failed: `(left == right)`
  left: `4`,
 right: `5`', src/lib.rs:11:8
note: Run with `RUST_BACKTRACE=1` for a backtrace.

failures:
    tests::it_adds_two

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out
```

¡Nuestra prueba encontró el error! La prueba llamada `it_adds_two` falló, mostrando el mensaje
`` assertion failed: `(left == right)` `` y mostrando a su vez que `left` era `4` y
`right` era `5`. Este mensaje es bastante útil y nos ayuda a comenzar a eliminar errores; significa que
el argumento `left` para `assert_eq!` era `4`, pero el argumento `right`, donde 
teníamos `add_two(2)`, era `5`.

Nota que en algunos idiomas y estructuras de pruebas, los parámetros para la
función que afirma que dos valores que son iguales son llamados `expected` y `actual`,
y el orden en el cual especificamos los argumentos sí es importante. Sin embargo, en Rust,
son llamados `left` y `right`, y el orden en el cual especificamos el valor
que esperamos y el valor que el código bajo prueba produce no importa.
Podríamos escribir la afirmación en este test como `assert_eq!(add_two(2), 4)`, lo que
resultaría en un mensaje de afirmación fallida que muestra `` assertion failed: `(left ==
right)` `` y que `left` era `5` y `right` era `4`.

El macro `assert_ne!` pasará si los dos valores que le damos no son iguales
y fallará si son iguales. Éste macro es más util para los casos en los que no estamos seguros
cual *será* el valor, pero sabemos lo que el valor definitivamente no *será* si nuestro
código está funcionando como lo habíamos previsto. Por ejemplo, si estamos probando una función que
está garantizada a cambiar su respuesta de alguna manera, pero la manera en la que la respuesta
cambia depende del día de la semana en el que ejecutemos nuestras pruebas, la mejor 
afirmación será que la respuesta de la función no es igual a la entrada.

Bajo su superficie, los macros `assert_eq!` y `assert_ne!` usan los operadores
`==` y `!=`, de forma respectiva. Cuando la afirmación falla, estos macros emiten sus
argumentos usando un formateo de eliminación de errores, lo que significa que los valores comparados deben
implementar los rasgos `PartialEq` y `Debug`. Todos los tipos primitivos y la mayoría
de los mismos de la biblioteca estandar implementan estos rasgos. Para estructuras y enums
that you define, you’ll need to implement `PartialEq` to assert that values of
those types are equal or not equal. You’ll need to implement `Debug` to print
out the values when the assertion fails. Because both traits are derivable
traits, as mentioned in Listing 5-12 in Chapter 5, this is usually as
straightforward as adding the `#[derive(PartialEq, Debug)]` annotation to your
struct or enum definition. See Appendix C for more details about these and
other derivable traits.

### Adding Custom Failure Messages

We can also add a custom message to be printed with the failure message as
optional arguments to the `assert!`, `assert_eq!`, and `assert_ne!` macros. Any
arguments specified after the one required argument to `assert!` or the two
required arguments to `assert_eq!` and `assert_ne!` are passed along to the
`format!` macro (discussed in Chapter 8 in the “Concatenation with the `+`
Operator or the `format!` Macro” section), so you can pass a format string that
contains `{}` placeholders and values to go in those placeholders. Custom
messages are useful to document what an assertion means; when a test fails,
we’ll have a better idea of what the problem is with the code.

For example, let’s say we have a function that greets people by name, and we
want to test that the name we pass into the function appears in the output:

<span class="filename">Filename: src/lib.rs</span>

```rust
pub fn greeting(name: &str) -> String {
    format!("Hello {}!", name)
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn greeting_contains_name() {
        let result = greeting("Carol");
        assert!(result.contains("Carol"));
    }
}
```

The requirements for this program haven’t been agreed upon yet, and we’re
pretty sure the `Hello` text at the beginning of the greeting will change. We
decided we don’t want to have to update the test for the name when that
happens, so instead of checking for exact equality to the value returned from
the `greeting` function, we’ll just assert that the output contains the text of
the input parameter.

Let’s introduce a bug into this code by changing `greeting` to not include
`name` to see what this test failure looks like:

```rust
pub fn greeting(name: &str) -> String {
    String::from("Hello!")
}
```

Running this test produces the following:

```text
running 1 test
test tests::greeting_contains_name ... FAILED

failures:

---- tests::greeting_contains_name stdout ----
        thread 'tests::greeting_contains_name' panicked at 'assertion failed:
result.contains("Carol")', src/lib.rs:12:8
note: Run with `RUST_BACKTRACE=1` for a backtrace.

failures:
    tests::greeting_contains_name
```

This result just indicates that the assertion failed and which line the
assertion is on. A more useful failure message in this case would print the
value we got from the `greeting` function. Let’s change the test function,
giving it a custom failure message made from a format string with a placeholder
filled in with the actual value we got from the `greeting` function:

```rust,ignore
#[test]
fn greeting_contains_name() {
    let result = greeting("Carol");
    assert!(
        result.contains("Carol"),
        "Greeting did not contain name, value was `{}`", result
    );
}
```

Now when we run the test, we’ll get a more informative error message:

```text
---- tests::greeting_contains_name stdout ----
        thread 'tests::greeting_contains_name' panicked at 'Greeting did not
contain name, value was `Hello!`', src/lib.rs:12:8
note: Run with `RUST_BACKTRACE=1` for a backtrace.
```

We can see the value we actually got in the test output, which would help us
debug what happened instead of what we were expecting to happen.

### Checking for Panics with `should_panic`

In addition to checking that our code returns the correct values we expect,
it’s also important to check that our code handles error conditions as we
expect. For example, consider the `Guess` type that we created in Chapter 9,
Listing 9-9. Other code that uses `Guess` depends on the guarantee that `Guess`
instances will only contain values between 1 and 100. We can write a test that
ensures that attempting to create a `Guess` instance with a value outside that
range panics.

We do this by adding another attribute, `should_panic`, to our test function.
This attribute makes a test pass if the code inside the function panics; the
test will fail if the code inside the function doesn’t panic.

Listing 11-8 shows a test that checks that the error conditions of `Guess::new`
happen when we expect:

<span class="filename">Filename: src/lib.rs</span>

```rust
pub struct Guess {
    value: u32,
}

impl Guess {
    pub fn new(value: u32) -> Guess {
        if value < 1 || value > 100 {
            panic!("Guess value must be between 1 and 100, got {}.", value);
        }

        Guess {
            value
        }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic]
    fn greater_than_100() {
        Guess::new(200);
    }
}
```

<span class="caption">Listing 11-8: Testing that a condition will cause a
`panic!`</span>

We place the `#[should_panic]` attribute after the `#[test]` attribute and
before the test function it applies to. Let’s look at the result when this test
passes:

```text
running 1 test
test tests::greater_than_100 ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

Looks good! Now let’s introduce a bug in our code by removing the condition
that the `new` function will panic if the value is greater than 100:

```rust
# pub struct Guess {
#     value: u32,
# }
#
// --snip--

impl Guess {
    pub fn new(value: u32) -> Guess {
        if value < 1  {
            panic!("Guess value must be between 1 and 100, got {}.", value);
        }

        Guess {
            value
        }
    }
}
```

When we run the test in Listing 11-8, it will fail:

```text
running 1 test
test tests::greater_than_100 ... FAILED

failures:

failures:
    tests::greater_than_100

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out
```

We don’t get a very helpful message in this case, but when we look at the test
function, we see that it’s annotated with `#[should_panic]`. The failure we got
means that the code in the test function did not cause a panic.

Tests that use `should_panic` can be imprecise because they only indicate that
the code has caused some panic. A `should_panic` test would pass even if the
test panics for a different reason than the one we were expecting to happen. To
make `should_panic` tests more precise, we can add an optional `expected`
parameter to the `should_panic` attribute. The test harness will make sure that
the failure message contains the provided text. For example, consider the
modified code for `Guess` in Listing 11-9 where the `new` function panics with
different messages depending on whether the value was too small or too large:

<span class="filename">Filename: src/lib.rs</span>

```rust
# pub struct Guess {
#     value: u32,
# }
#
// --snip--

impl Guess {
    pub fn new(value: u32) -> Guess {
        if value < 1 {
            panic!("Guess value must be greater than or equal to 1, got {}.",
                   value);
        } else if value > 100 {
            panic!("Guess value must be less than or equal to 100, got {}.",
                   value);
        }

        Guess {
            value
        }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic(expected = "Guess value must be less than or equal to 100")]
    fn greater_than_100() {
        Guess::new(200);
    }
}
```

<span class="caption">Listing 11-9: Testing that a condition will cause a
`panic!` with a particular panic message</span>

This test will pass because the value we put in the `should_panic` attribute’s
`expected` parameter is a substring of the message that the `Guess::new`
function panics with. We could have specified the entire panic message that we
expect, which in this case would be `Guess value must be less than or equal to
100, got 200.` What you choose to specify in the expected parameter for
`should_panic` depends on how much of the panic message is unique or dynamic
and how precise you want your test to be. In this case, a substring of the
panic message is enough to ensure that the code in the test function executes
the `else if value > 100` case.

To see what happens when a `should_panic` test with an `expected` message
fails, let’s again introduce a bug into our code by swapping the bodies of the
`if value < 1` and the `else if value > 100` blocks:

```rust,ignore
if value < 1 {
    panic!("Guess value must be less than or equal to 100, got {}.", value);
} else if value > 100 {
    panic!("Guess value must be greater than or equal to 1, got {}.", value);
}
```

This time when we run the `should_panic` test, it will fail:

```text
running 1 test
test tests::greater_than_100 ... FAILED

failures:

---- tests::greater_than_100 stdout ----
        thread 'tests::greater_than_100' panicked at 'Guess value must be
greater than or equal to 1, got 200.', src/lib.rs:11:12
note: Run with `RUST_BACKTRACE=1` for a backtrace.
note: Panic did not include expected string 'Guess value must be less than or
equal to 100'

failures:
    tests::greater_than_100

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out
```

The failure message indicates that this test did indeed panic as we expected,
but the panic message did not include the expected string `'Guess value must be
less than or equal to 100'`. The panic message that we did get in this case was
`Guess value must be greater than or equal to 1, got 200.` Now we can start
figuring out where our bug is!

Now that you know several ways to write tests, let’s look at what is happening
when we run our tests and explore the different options we can use with `cargo
test`.
