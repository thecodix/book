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

### Checking Results with the `assert!` Macro

The `assert!` macro, provided by the standard library, is useful when you want
to ensure that some condition in a test evaluates to `true`. We give the
`assert!` macro an argument that evaluates to a Boolean. If the value is
`true`, `assert!` does nothing and the test passes. If the value is `false`,
the `assert!` macro calls the `panic!` macro, which causes the test to fail.
Using the `assert!` macro helps us check that our code is functioning in the
way we intend.

In Chapter 5, Listing 5-15, we used a `Rectangle` struct and a `can_hold`
method, which are repeated here in Listing 11-5. Let’s put this code in the
*src/lib.rs* file and write some tests for it using the `assert!` macro.

<span class="filename">Filename: src/lib.rs</span>

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

<span class="caption">Listing 11-5: Using the `Rectangle` struct and its
`can_hold` method from Chapter 5</span>

The `can_hold` method returns a Boolean, which means it’s a perfect use case
for the `assert!` macro. In Listing 11-6, we write a test that exercises the
`can_hold` method by creating a `Rectangle` instance that has a length of 8 and
a width of 7, and asserting that it can hold another `Rectangle` instance that
has a length of 5 and a width of 1:

<span class="filename">Filename: src/lib.rs</span>

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

<span class="caption">Listing 11-6: A test for `can_hold` that checks that a
larger rectangle can indeed hold a smaller rectangle</span>

Note that we’ve added a new line inside the `tests` module: the `use super::*;`
line. The `tests` module is a regular module that follows the usual visibility
rules we covered in Chapter 7 in the “Privacy Rules” section. Because the
`tests` module is an inner module, we need to bring the code under test in the
outer module into the scope of the inner module. We use a glob here so anything
we define in the outer module is available to this `tests` module.

We’ve named our test `larger_can_hold_smaller`, and we’ve created the two
`Rectangle` instances that we need. Then we called the `assert!` macro and
passed it the result of calling `larger.can_hold(&smaller)`. This expression
is supposed to return `true`, so our test should pass. Let’s find out!

```text
running 1 test
test tests::larger_can_hold_smaller ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

It does pass! Let’s add another test, this time asserting that a smaller
rectangle cannot hold a larger rectangle:

<span class="filename">Filename: src/lib.rs</span>

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

Because the correct result of the `can_hold` function in this case is `false`,
we need to negate that result before we pass it to the `assert!` macro. As a
result, our test will pass if `can_hold` returns `false`:

```text
running 2 tests
test tests::smaller_cannot_hold_larger ... ok
test tests::larger_can_hold_smaller ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

Two tests that pass! Now let’s see what happens to our test results when we
introduce a bug in our code. Let’s change the implementation of the `can_hold`
method by replacing the greater-than sign with a less-than sign when it
compares the lengths:

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

Running the tests now produces the following:

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

Our tests caught the bug! Because `larger.length` is 8 and `smaller.length` is
5, the comparison of the lengths in `can_hold` now returns `false`: 8 is not
less than 5.

### Testing Equality with the `assert_eq!` and `assert_ne!` Macros

A common way to test functionality is to compare the result of the code under
test to the value we expect the code to return to make sure they’re equal. We
could do this using the `assert!` macro and passing it an expression using the
`==` operator. However, this is such a common test that the standard library
provides a pair of macros—`assert_eq!` and `assert_ne!`—to perform this test
more conveniently. These macros compare two arguments for equality or
inequality, respectively. They’ll also print the two values if the assertion
fails, which makes it easier to see *why* the test failed; conversely, the
`assert!` macro only indicates that it got a `false` value for the `==`
expression, not the values that lead to the `false` value.

In Listing 11-7, we write a function named `add_two` that adds `2` to its
parameter and returns the result. Then we test this function using the
`assert_eq!` macro.

<span class="filename">Filename: src/lib.rs</span>

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

<span class="caption">Listing 11-7: Testing the function `add_two` using the
`assert_eq!` macro</span>

Let’s check that it passes!

```text
running 1 test
test tests::it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

The first argument we gave to the `assert_eq!` macro, `4`, is equal to the
result of calling `add_two(2)`. The line for this test is `test
tests::it_adds_two ... ok`, and the `ok` text indicates that our test passed!

Let’s introduce a bug into our code to see what it looks like when a test that
uses `assert_eq!` fails. Change the implementation of the `add_two` function to
instead add `3`:

```rust
pub fn add_two(a: i32) -> i32 {
    a + 3
}
```

Run the tests again:

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

Our test caught the bug! The `it_adds_two` test failed, displaying the message
`` assertion failed: `(left == right)` `` and showing that `left` was `4` and
`right` was `5`. This message is useful and helps us start debugging: it means
the `left` argument to `assert_eq!` was `4`, but the `right` argument, where we
had `add_two(2)`, was `5`.

Note that in some languages and test frameworks, the parameters to the
functions that assert two values are equal are called `expected` and `actual`,
and the order in which we specify the arguments matters. However, in Rust,
they’re called `left` and `right`, and the order in which we specify the value
we expect and the value that the code under test produces doesn’t matter. We
could write the assertion in this test as `assert_eq!(add_two(2), 4)`, which
would result in a failure message that displays `` assertion failed: `(left ==
right)` `` and that `left` was `5` and `right` was `4`.

The `assert_ne!` macro will pass if the two values we give it are not equal and
fail if they’re equal. This macro is most useful for cases when we’re not sure
what a value *will* be, but we know what the value definitely *won’t* be if our
code is functioning as we intend. For example, if we’re testing a function that
is guaranteed to change its input in some way, but the way in which the input
is changed depends on the day of the week that we run our tests, the best thing
to assert might be that the output of the function is not equal to the input.

Under the surface, the `assert_eq!` and `assert_ne!` macros use the operators
`==` and `!=`, respectively. When the assertions fail, these macros print their
arguments using debug formatting, which means the values being compared must
implement the `PartialEq` and `Debug` traits. All the primitive types and most
of the standard library types implement these traits. For structs and enums
que definas, necesitarás implementar `PartialEq` para afirmar que los valores de
esos tipos son iguales o desiguales. Necesitarás implementar `Debug` para emitir
los valores cuando la afirmación falle. Ya que los dos rasgos son rasgos
derivables, como fue mencionado en el Listado 5-12 en el Capítulo 5, esto es usualmente tan
directo como el agregar la anotación `#[derive(PartialEq, Debug)]` a tu 
estructura o definición enum. Ve el apéndice C para más detalles sobre estos
y otros rasgos derivables.

### Añadir mensajes de fallo personalizados

También podemos añadir un mensaje personalizado para ser mostrado con el mensaje de fallo como
argumentos opcionales para los macros `assert!`, `assert_eq!`, y `assert_ne!`. Cualquiera
de los argumentos especificados luego del unico argumento requerido para `assert!` o los dos 
requeridos para `assert_eq!` y `assert_ne!` son enviados al 
macro `format!` (discutidos en el capitulo 8 en la sección “Concatenación con el operador `+`
o el macro `format!`”), así que puedes pasar una cadena de formato que 
contenga los marcadores de posición `{}` y los valores que vayan en esos marcadores. Los mensajes
personalizados son útiles para documentar lo que una afirmación significa; cuando un test falla,
tendremos una mejor idea de cuál es el problema con el código.

Por ejemplo, digamos que tenemos una función que salude a las personas por su nombre, y
queremos probar que el nombre que pasemos en la función aparezca en el resultado:

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

Los requerimientos para este programa aún no han sido aceptados, y estamos
bastante seguros que el texto `Hello` en el inicio del saludo cambiará.
Decidimos que no queremos tener que actualizar el test para el nombre cuando eso
pase, así que en vez de chequear por la igualdad exacta al valor devuelto desde
la función `greeting`, sólo afirmaremos que la salida contenga el texto de
la entrada del parámetro.

Introduzcamos un bug dentro de este código cambiando `greeting` para que no incluya
`name` para ver como luce esta falla del test
```rust
pub fn greeting(name: &str) -> String {
    String::from("Hello!")
}
```

Ejecutar este test produce lo siguiente:

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

Este resultado solo indica que la afirmación falló y en cual línea
está la afirmación. Un mensaje de fallo más util en este caso podría emitir
el valor que obtuvimos de la función `greeting`. Cambiemos la función test,
dándole un mensaje personalizado de fallo hecho por un formato de hilos con un marcador
lleno con el valor que obtuvimos por la función `greeting`:

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

Ahora cuando ejecutemos el test, obtendremos un mensaje de error más informativo:

```text
---- tests::greeting_contains_name stdout ----
        thread 'tests::greeting_contains_name' panicked at 'Greeting did not
contain name, value was `Hello!`', src/lib.rs:12:8
note: Run with `RUST_BACKTRACE=1` for a backtrace.
```

Podemos ver el valor que obtuvimos en la respuesta del test, lo que nos ayudaría
a eliminar los errores de lo que pasó en vez de lo que esperamos que pase

### Comprobar errores por panico con `should_panic`

En adición al chequear que nuestro código devuelva los valores correctos que esperamos,
es importante revisar que nuestro código maneje condiciones de error como nosotros
esperamos. Por ejemplo, considera el tipo `Guess` que creamos en el Capítulo 9,
en el Listado 9-9. Otro código que usa `Guess` depende de la garantía de que las instancias `Guess`
solo contengan valores entre 1 y 100. Podemos hacer un test que
asegure ese intento al crear una instancia `Guess` con un valor fuera del 
rango de entrar en pánico.

Hacemos esto añadiendo otro atributo, `should_panic`, a nuestra función test.
Este atributo hace que el test pase si el código dentro de la función entra en pánico; el
test fallará si el código dentro de la función no entra en pánico.

Listado 11-8 muestra un test que revisa que las condiciones de error de `Guess::new`
pasan como lo hemos previsto:

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

<span class="caption">Listado 11-8: Prueba que una condición cause un
`panic!`</span>

Ponemos el atributo `#[should_panic]` luego del atributo `#[test]` y
antes de la función test que le aplica. Miremos el resultado cuando esta test
pasa:

```text
running 1 test
test tests::greater_than_100 ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

¡Luce bien! Ahora introduzcamos un bug en nuestro código eliminando la condición
que la función `new` entrará en pánico si el valor es mayor a 100:

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

Cuando ejecutemos el test en el listado 11-8, fallará:

```text
running 1 test
test tests::greater_than_100 ... FAILED

failures:

failures:
    tests::greater_than_100

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out
```

No recibimos un mensaje muy útil en esta ocación, pero cuando le echamos un vistazo a la función
test, vemos que está anotada con `#[should_panic]`. El fallo que obtuvimos
significa que el código en la función test no causo pánico.

Los tests que usan `should_panic` pueden no ser precisa porque ellas sólo indican que 
código ha causado algún pánico. Un test `should_panic` pasaría si el
test entra en pánico por una razón diferente que la que estabamos esperando que ocurriera. Para
hacer los tests `should_panic` más precisas, podemos añadir un parámetro `expected`
opcional al atributo `should_panic`. El aprovechamiento del test se asegurará que 
el mensaje de fallo contenga el texto provisto. Por ejemplo, considera el
código modificado para `Guess` en el listado 11-9 donde la función  `new` entre en pánico con
diferentes mensajes dependiendo de si el valor fue muy pequeño o muy grande:

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

<span class="caption">Listado 11-9: Prueba de que una condición causará un
`panic!` con un mensaje de pánico particular</span>

Este test pasará porque el valor que pusimos en el atributo `should_panic` de
`expected` del atributo es una subcadena del mensaje con la que la función `Guess::new`
entra en pánico. Podríamos haber especificado el mensaje de pánico entero que
esperamos, el cual en este caso sería `Guess value must be less than or equal to
100, got 200.` Lo que tú decidas especificar en el parámetro esperado para
`should_panic` depende de qué tan dinámico o único el mensaje de pánico sea
y qué tan precisa quieres que sea tu test. En este caso, una subcadena del 
mensaje de pánico es suficiente para asegurar que el código en la función test ejecute
el caso `else if value > 100`.

Para ver lo que pasa cuando un test `should_panic` con un mensaje `expected` 
falla, introduzcamos de nuevo un bug en nuestro código cambiando los cuerpos de
los bloques`if value < 1` y `else if value > 100` :

```rust,ignore
if value < 1 {
    panic!("Guess value must be less than or equal to 100, got {}.", value);
} else if value > 100 {
    panic!("Guess value must be greater than or equal to 1, got {}.", value);
}
```

Esta vez cuando ejecutemos `should_panic` fallará:

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

El mensaje de fallo indica que este test de hecho entró en pánico como nosotros esperábamos,
pero el mensaje de pánico no incluyo el hilo `'Guess value must be
less than or equal to 100'` que esperábamos. El mensaje de pánico que sí obtuvimos en este caso fue
`Guess value must be greater than or equal to 1, got 200.` ¡Ahora podemos empezar a
averiguar donde está el bug!

Ahora que sabes varias formas de hacer tests, echemos un vistazo a lo que está pasando
cuando ejecutamos nuestros test y exploramos las diferentes opciones que podemos usar con `cargo
test`.
