## The `match` Control Flow Operator

Rust has an extremely powerful control-flow operator called `match` that allows
us to compare a value against a series of patterns and then execute code based
on which pattern matches. Patterns can be made up of literal values, variable
names, wildcards, and many other things; Chapter 18 covers all the different
kinds of patterns and what they do. The power of `match` comes from the
expressiveness of the patterns and the compiler checks that all
possible cases are handled.

Think of a `match` expression kind of like a coin sorting machine: coins slide
down a track with variously sized holes along it, and each coin falls through
the first hole it encounters that it fits into. In the same way, values go
through each pattern in a `match`, and at the first pattern the value “fits,”
the value will fall into the associated code block to be used during execution.

Because we just mentioned coins, let’s use them as an example using `match`! We
can write a function that can take an unknown United States coin and, in a
similar way as the counting machine, determine which coin it is and return its
value in cents, as shown here in Listing 6-3:

```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u32 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

<span class="caption">Listing 6-3: An enum and a `match` expression that has
the variants of the enum as its patterns.</span>

Let’s break down the `match` in the `value_in_cents` function. First, we list
the `match` keyword followed by an expression, which in this case is the value
`coin`. This seems very similar to an expression used with `if`, but there’s a
big difference: with `if`, the expression needs to return a Boolean value.
Here, it can be any type. The type of `coin` in this example is the `Coin` enum
that we defined in Listing 6-3.

Next are the `match` arms. An arm has two parts: a pattern and some code. The
first arm here has a pattern that is the value `Coin::Penny` and then the `=>`
operator that separates the pattern and the code to run. The code in this case
is just the value `1`. Each arm is separated from the next with a comma.

When the `match` expression executes, it compares the resulting value against
the pattern of each arm, in order. If a pattern matches the value, the code
associated with that pattern is executed. If that pattern doesn’t match the
value, execution continues to the next arm, much like a coin sorting machine.
We can have as many arms as we need: in Listing 6-3, our `match` has four arms.

The code associated with each arm is an expression, and the resulting value of
the expression in the matching arm is the value that gets returned for the
entire `match` expression.

Curly brackets typically aren’t used if the match arm code is short, as it is
in Listing 6-3 where each arm just returns a value. If you want to run multiple
lines of code in a match arm, you can use curly brackets. For example, the
following code would print out “Lucky penny!” every time the method was called
with a `Coin::Penny` but would still return the last value of the block, `1`:

```rust
# enum Coin {
#    Penny,
#    Nickel,
#    Dime,
#    Quarter,
# }
#
fn value_in_cents(coin: Coin) -> u32 {
    match coin {
        Coin::Penny => {
            println!("Lucky penny!");
            1
        },
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

### Patterns that Bind to Values

Another useful feature of match arms is that they can bind to parts of the
values that match the pattern. This is how we can extract values out of enum
variants.

As an example, let’s change one of our enum variants to hold data inside it.
From 1999 through 2008, the United States minted quarters with different
designs for each of the 50 states on one side. No other coins got state
designs, so only quarters have this extra value. We can add this information to
our `enum` by changing the `Quarter` variant to include a `UsState` value stored
inside it, which we’ve done here in Listing 6-4:

```rust
#[derive(Debug)] // So we can inspect the state in a minute
enum UsState {
    Alabama,
    Alaska,
    // ... etc
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}
```

<span class="caption">Listing 6-4: A `Coin` enum where the `Quarter` variant
also holds a `UsState` value</span>

Let’s imagine that a friend of ours is trying to collect all 50 state quarters.
While we sort our loose change by coin type, we’ll also call out the name of
the state associated with each quarter so if it’s one our friend doesn’t have,
they can add it to their collection.

In the match expression for this code, we add a variable called `state` to the
pattern that matches values of the variant `Coin::Quarter`. When a
`Coin::Quarter` matches, the `state` variable will bind to the value of that
quarter’s state. Then we can use `state` in the code for that arm, like so:

```rust
# #[derive(Debug)]
# enum UsState {
#    Alabama,
#    Alaska,
# }
#
# enum Coin {
#    Penny,
#    Nickel,
#    Dime,
#    Quarter(UsState),
# }
#
fn value_in_cents(coin: Coin) -> u32 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {:?}!", state);
            25
        },
    }
}
```

If we were to call `value_in_cents(Coin::Quarter(UsState::Alaska))`, `coin`
would be `Coin::Quarter(UsState::Alaska)`. When we compare that value with each
of the match arms, none of them match until we reach `Coin::Quarter(state)`. At
that point, the binding for `state` will be the value `UsState::Alaska`. We can
then use that binding in the `println!` expression, thus getting the inner
state value out of the `Coin` enum variant for `Quarter`.

### Matching with `Option<T>`

In the previous section we wanted to get the inner `T` value out of the `Some`
case when using `Option<T>`; we can also handle `Option<T>` using `match` as we
did with the `Coin` enum! Instead of comparing coins, we’ll compare the
variants of `Option<T>`, but the way that the `match` expression works remains
the same.

Let’s say we want to write a function that takes an `Option<i32>`, and if
there’s a value inside, adds one to that value. If there isn’t a value inside,
the function should return the `None` value and not attempt to perform any
operations.

This function is very easy to write, thanks to `match`, and will look like
Listing 6-5:

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i + 1),
    }
}

let five = Some(5);
let six = plus_one(five);
let none = plus_one(None);
```
<span class="caption">Listado 6-5: Una función que usa una expresión `match` en
una `Option<i32>`</span>

#### Emparejando `Some(T)`

Examinaremos la primera ejecución de `plus_one` en más detalles. Cuando llamamos
`plus_one(five)`, la variable `x` en el cuerpo de `plus_one` tendrá el valor
`Some(5)`. Luego lo comparamos contra cada brazo de coincidencia.


```rust,ignore
None => None,
```

El valor `Some(5)` no coincide con el patrón `None`, entonces continuamos al siguiente
brazo.

```rust,ignore
Some(i) => Some(i + 1),
```

¿Coincide `Some(5)` con `Some(i)`? Si, lo hace! Tenemos la misma variante.
La `i` se une al valor contenido en `Some`, entonces `i` toma el valor `5`. El
código en el brazo de coincidencia es entonces ejecutado, asique agregamos uno
al valor de `i` y creamos un nuevo valor `Some` con nuestro total `6` adentro.

#### Emparejando `None`

Ahora consideremos la segunda llamda de `plus_one` en el Listado 6-5 donde `x` es
`None`. Ingresamos el valor `match` y lo comparamos al primer brazo.


```rust,ignore
None => None,
```

Coincide! No hay valor para agregar, entonces el programa se detiene y devuelve el valor
`None` en el lado derecho de `=>`. Debido a que el primer brazo coincide, no se 
comparan otros brazos.

Combinar `match` y enumeraciones es útil en muchas situaciones. Verás mucho este patrón
en el código Rust: `match` contra una enumeración, junta una variable con los
datos adentro, y entonces ejecuta código basado en él. Es un poco díficil al principio, pero
una vez que lo usas, querrás tenerlo en todos los lenguajes. Es
una de las funciones favoritas de los usuarios.

### Los emparejamientos son exhaustivos

Hay otro aspecto de `match` que debemos discutir. Considera esta versión
de nuestra función `plus_one`:


```rust,ignore
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        Some(i) => Some(i + 1),
    }
}
```

No manejamos el caso `None`, entonces este código causará un bug. Por suerte, es un bug
que Rust sabe como atrapar. Si intentamos compilar este código, obtendremos este
error:

```text
error[E0004]: non-exhaustive patterns: `None` not covered
 -->
  |
6 |         match x {
  |               ^ pattern `None` not covered
```


Rust sabe que no cubrimos cada caso posible every possible e incluso sabe que patrón
olvidamos! Las coincidencias en Rust son *agotadores*: debemos usar hasta la última 
posibilidad para que el código sea válido. Especialmente en el caso de
`Option<T>`, cuanto Rust previene olvidarnos de manejar explicitamente el caso
`None`, esto nos protege de asumir que tenemos un valor cuando podríamos tener
nulo, asi como el erro billón de dolares discutido anteriormente.

### El `_` Marcador de posición

Rust también tiene un patrón que podemos usar en situaciones cuando no queremos listar
todos los posibles valores. Por ejemplo, un `u8` puede tener valores válidos de 0 a 255. Si
si solo nos importan los valores 1, 3, 5 y 7, no queremos tener uqe listar
0, 2, 4, 6, 8, 9, todos hasta 255 . Por suerte, no tenemos que hacerlo: podemos
usar un patrón especial `_` en vez de:

```rust
let some_u8_value = 0u8;
match some_u8_value {
    1 => println!("one"),
    3 => println!("three"),
    5 => println!("five"),
    7 => println!("seven"),
    _ => (),
}
```

El patrón `_` concidirá cualquier valor. Poniéndolo después de nuestros otros brazos, el
`_` coincidierá con todos los casos posibles que no fueron especificados antes. El `()`
es solo la unidad de valor, por lo que nada pasará en el caso  `_`. Como resultado, podemos 
decir que no queremos hacer nada para todos los valores posibles que no listamos
antes del marcador de posición `_` .

Sin embargo, la expresión `match` puede ser un poco tediosa en una situación en que solo
nos importa *uno* de los casos. Para esta situacuión, Rust provee `if let`.
