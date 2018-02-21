## Cómo Escribir Pruebas

Las pruebas son funciones de Rust que verifican que el codigo que no está usando funcione
de la manera esperada. Los cuerpos de las funciones de prueba realizan tipicamente estas tres
acciones:

1. Colocan un dato o estado necesitado
2. corren el código que queremos probar
3. Aseguran los resultados que estamos esperando

Miremos las caracteristicas que nos provee Rust específicamente para escribir pruebas que
engloben estas acciones, las cuales incluyen el atributo `test` , unos cuantos macros, y el
atributo `should_panic` .

### La Anatomía de una Función de Prueba

De la manera más simple, una prueba en Rust es una función que está anotada con el atributo `test`.
Los atributos son datos metas sobre piezas del código Rust; un ejemplo es
el atributo `derive` que usamos con las estructuras en el capítulo 5. Para cambiar una función
a una de prueba, agregamos `#[test]` en la línea anterior a `fn`. Cuando corremos nuestras 
pruebas con el comando `cargo test` Rust construye un binario que ejecute el test que corre
las funciones anotadas con el atributo `test` y reporta si cada 
función de prueba aprueba o falla.

En el capítulo 7 vimos que cuando hacemos un nuevo proyecto de biblioteca con Cargo, un módulo
de prueba con una función de prueba en él es automáticamente generado para nosotros. Este
módulo nos ayuda a empezar a escribir nuestras pruebas para que no tengamos que buscar la estructura
o sintaxis exactas de las funciones de prueba cada vez que empezamos un nuevo proyecto. Podemos
también agregar tantas funciones y módulos de prueba adicionales como queramos.

Exploraremos algunos aspectos de como trabajan las pruebas al experimentar con el la prueba
de la plantilla generada para nosotros sin probar ningún código. Entonces escribiremos unas
pruebas en el mundo real que llamen algún código que hayamos escrito y asegurar que su
comportamiento sea uno correcto.

Vamos a crear un nuevo proyecto en la biblioteca llamado `adder`:

```text
$ cargo new adder
     Created library `adder` project
$ cd adder
```

Los contenidos del archivo *src/lib.rs* en tu biblioteca de complementos deberían verse como
los del listado 11-1:

<span class="filename">Nombre del archivo: src/lib.rs</span>

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}
```

<span class="caption">Listado 11-1: El módulo de prueba y la función generada 
automáticamente por `cargo new`</span>

Por ahora ignoremos las dos lineas superiores y enfoquémonos en la función para ver cómo
funciona. Date cuenta de que la anotación `#[test]` está antes de la linea `fn`: este atributo
indica que ésta es una función de prueba, así que el ejecutador de la prueba sabe cómo tratar esta 
función de prueba. Podríamos tener funciones que no lo son en el módulo `tests` 
para que nos ayuden a crear escenarios o realizar operaciones comunes, así que necesitamos 
indicarle qué funciones son de prueba usando el atributo `#[test]`.

La función del cuerpo usa el macro `assert_eq!` para asegurar que 2 + 2 es igual a 4.
Esta afirmación sirve como un ejemplo del formato para el la prueba típica. Vamos a correrla
para ver que esta prueba es satisfactoria.

El comando `cargo test` corre todas las pruebas en nuestro proyecto, como lo muestra el listado
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

<span class="caption">Listado 11-2: La respuesta de correr la prueba que ha sido generada 
automaticamente</span>

Cargo compiló y corrió el la prueba. Luego de las lineas `Compiling`, `Finished`, y
`Running` esta linea `running 1 test`, la siguiente linea muestra el nombre 
de la función de prueba generada, llamada `it_works`, y el resultado del correr 
esa prueba, `ok`. El resumen completo de la prueba ejecutada es lo siguiente que aparece. El
texto `test result: ok.` significa que todos las pruebas han sido satisfactorias, y que la porción que 
lee `1 passed; 0 failed` es el total del número de pruebas que han sido aprobadas o fallidas.

Ya que no tenemos ninguna prueba que tengamos marcada como ignorado, el resumen muestra `0
ignored`. Tampoco hemos filtrado las pruebas que se están ejecutando, así que el final del
resumen muestra `0 filtered out`. Hablaremos del ignorar y filtrar
pruebas en la siguiente sección, “Controlando cómo son ejecutados los tests.”

La estadística `0 measured` se usa para pruebas de referencia que miden el desempeño.
Las tests de referencia son, en estos escritos, solo disponible en el Rust nightly. Mira
El capítulo 1 para más información sobre el Rust nightly.

La siguiente parte de la respuesta de la prueba, la cual comienza con `Doc-tests adder`, es para
los resultados de cualquier prueba de documentación. No tenemos ninguna prueba de documentación
por ahora, pero Rust puede compilar cualquier ejemplo de códigos que aparezcan en nuestra documentación
API. ¡Esta caracteristica nos ayuda a mantener nuestros documentos y nuestro código en un estado de sincronización! Discutiremos
cómo escribir pruebas de documentación en la sección 
“Comentarios de documentación” del capítulo 14. Por ahora, ignoraremos la respuesta `Doc-tests`.

Cambiemos el nombre de nuestra prueba para ver cómo eso cambia el resultado de la misma.
Cambia la función `it_works` a un nombre diferente, uno como `exploration`, de esta manera:


<span class="filename">Nombre del archivo: src/lib.rs</span>

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn exploration() {
        assert_eq!(2 + 2, 4);
    }
}
```

Entonces ejecuta `cargo test` de nuevo. La respuesta ahora muestra `exploration` en vez de
`it_works`:

```text
running 1 test
test tests::exploration ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

Añadamos otra prueba, ¡pero esta vez haremos una que falle! Ellas fallan
cuando algo dentro de la función de prueba entra en pánico. Cada una es ejecutada en un nuevo hilo,
y cuando el hilo principal se da cuenta de que un hilo de prueba ha muerto, la prueba es marcada 
como fallida. Hablamos sobre la manera más simple para causar pánico en el capítulo 9,
la cual es llamar al macro `panic!`. Ingresa en la nueva prueba, `another`, para que tu archivo
*src/lib.rs* se parezca al del listado 11-3:

<span class="filename">Nombre del Archivo: src/lib.rs</span>

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

<span class="caption">Listado 11-3: Añadiendo una segunda prueba que fallará porque 
llamamos el macro `panic!` </span>

Ejecuta la prueba de nuevo usando `cargo test`. La respuesta debería lucir como la del listado 
11-4, la cual muestra que nuestra prueba `exploration` es satisfactoria y `another` es fallida:

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

<span class="caption">Listado 11-4: Los resultados de las pruebas cuando una es satisfactoria y la
otra es fallida</span>

En vez de `ok`, la linea `test tests::another` muestra `FAILED`. Dos nuevas
secciones se muestran entre los resultados individuales y el resumen: La primera sección
muestra la razón detallada de por qué cada prueba fue falliza. En este caso,
`another` falló porque `panicked at 'Make this test fail'`, lo que sucedió
en la linea 10 del archivo *src/lib.rs*. La siguiente sección hace una lista solo de los nombres de
todos las pruebas fallidas, lo que es útil cuando hay muchas pruebas y muchas 
respuestas detalladas de las pruebas fallidas. Podemos usar el nombre de una prueba fallida para ejecutarla
solamente y así eliminar los fallos de la misma; hablaremos más sobre otras formas de ejecutar pruebas en
la sección “Controlando cómo las pruebas son ejecutadas”.

La linea de resumen se muestra al final: en general, nuestro resultado de prueba es `FAILED`.
Hicimos que una prueba que fuese satisfactoria y otra fallida.

Ahora que ya has visto como lucen los resultados de las pruebas en diferentes escenarios,
miremos otros macros además del `panic!` que son útiles en ellas.

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
