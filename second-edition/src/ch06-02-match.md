## El operador de flujo de control `match`

Rust tiene un operador de flujo de control extremadamente poderoso llamado `match` que nos permite comparar un valor con una serie de patrones y luego ejecutar código en función de qué patrón coincida. Los patrones pueden estar formados por valores literales, nombres de variables, comodines y muchas otras cosas; El Capítulo 18 cubre todos los diferentes tipos de patrones y lo que hacen. El poder de `match` proviene de la expresividad de los patrones y el compilador verifica que se manejen todos los casos posibles.

Piense en una expresión de "coincidencia" similar a una máquina clasificadora de monedas: las monedas se deslizan por una pista con agujeros de varios tamaños a lo largo de ella, y cada moneda cae por el primer agujero que encuentra que encaja. De la misma manera, los valores pasan por cada patrón en un "emparejamiento", y en el primer patrón el valor "encaja", el valor caerá en el bloque de código asociado para ser utilizado durante la ejecución.

¡Debido a que acabamos de mencionar monedas, utilicémoslas como ejemplo usando `match`! Podemos escribir una función que puede tomar una moneda desconocida de los Estados Unidos y, de manera similar a la máquina de conteo, determinar qué moneda es y devolver su valor en centavos, como se muestra aquí en el Listado 6-3:

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

<span class="caption">Listado 6-3: Una expresión enum y `match` que tiene las variantes de la enumeración como sus patrones.</span>

Vamos a desglosar el `match` en la función` value_in_cents`. Primero, enumeramos la palabra clave `match` seguida de una expresión, que en este caso es el valor `coin`. Esto parece muy similar a una expresión utilizada con `if`, pero hay una gran diferencia: con `if`, la expresión debe devolver un valor booleano. Aquí, puede ser de cualquier tipo. El tipo de `Coin` en este ejemplo es la enumeración `Coin` que definimos en el Listado 6-3.

A continuación están los brazos del `match`. Un brazo tiene dos partes: un patrón y algo de código. El primer brazo aquí tiene un patrón que es el valor `Coin::Penny` y luego el operador` => `que separa el patrón y el código para ejecutar. El código en este caso es solo el valor `1`. Cada brazo está separado del siguiente con una coma.

Cuando se ejecuta la expresión `match`, compara el valor resultante con el patrón de cada brazo, en orden. Si un patrón coincide con el valor, se ejecuta el código asociado con ese patrón. Si ese patrón no coincide con el valor, la ejecución continúa al siguiente brazo, al igual que una máquina clasificadora de monedas. Podemos tener tantas armas como necesitemos: en el Listado 6-3, nuestro `match` tiene cuatro brazos.

El código asociado a cada brazo es una expresión, y el valor resultante de la expresión en el brazo correspondiente es el valor que se devuelve para toda la expresión `match`.

Los corchetes generalmente no se usan si el código del brazo del partido es corto, como en el Listado 6-3, donde cada brazo simplemente devuelve un valor. Si desea ejecutar varias líneas de código en un brazo de coincidencia, puede usar corchetes. Por ejemplo, el siguiente código imprimirá "Lucky penny!" Cada vez que se llama al método con un `Coin::Penny` pero aún devuelve el último valor del bloque,` 1`:

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

### Patrones que se unen a los valores

Otra característica útil de los brazos de partido es que pueden unirse a partes de los valores que coinciden con el patrón. Así es como podemos extraer valores de las variantes enum.

Como ejemplo, cambiemos una de nuestras variantes enum para contener datos dentro de ella. De 1999 a 2008, los Estados Unidos acuñaron cuartos con diferentes diseños para cada uno de los 50 estados en un lado. Ninguna otra moneda tiene diseños de estado, por lo que solo los cuartos tienen este valor adicional. Podemos agregar esta información a nuestro `enum` cambiando la variante` Quarter` para incluir un valor `UsState` almacenado en su interior, lo que hemos hecho aquí en el Listado 6-4:

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

<span class="caption">Listado 6-4: una enumeración `Coin` donde la variante` Quarter` también tiene un valor `UsState`</span>

Imaginemos que un amigo nuestro está tratando de reunir los 50 alojamientos estatales. Mientras clasificamos nuestro cambio suelto por tipo de moneda, también llamaremos el nombre del estado asociado con cada trimestre, por lo que si es uno que nuestro amigo no tiene, pueden agregarlo a su colección.

En la expresión de coincidencia para este código, agregamos una variable llamada `state` al patrón que coincide con los valores de la variante `Coin::Quarter`. Cuando coincide un `Coin::Quarter`, la variable `state` se vinculará al valor del estado de ese trimestre. Entonces podemos usar `state` en el código para ese brazo, así:

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

Si tuviéramos que llamar a `value_in_cents(Coin::Quarter(UsState::Alaska))`, `coin` sería `Coin::Quarter(UsState::Alaska)`. Cuando comparamos ese valor con cada uno de los brazos de coincidencia, ninguno de ellos coincide hasta que llegamos a `Coin::Quarter(State)`. En ese punto, el enlace para `state` será el valor` UsState::Alaska`. Entonces podemos usar ese enlace en la expresión `println!`, Obteniendo así el valor de estado interno de la variante enum `Coin` para` Quarter`.

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

<span class="caption">Listing 6-5: A function that uses a `match` expression on
an `Option<i32>`</span>

#### Matching `Some(T)`

Let’s examine the first execution of `plus_one` in more detail. When we call
`plus_one(five)`, the variable `x` in the body of `plus_one` will have the
value `Some(5)`. We then compare that against each match arm.

```rust,ignore
None => None,
```

The `Some(5)` value doesn’t match the pattern `None`, so we continue to the
next arm.

```rust,ignore
Some(i) => Some(i + 1),
```

Does `Some(5)` match `Some(i)`? Well yes it does! We have the same variant.
The `i` binds to the value contained in `Some`, so `i` takes the value `5`. The
code in the match arm is then executed, so we add one to the value of `i` and
create a new `Some` value with our total `6` inside.

#### Matching `None`

Now let’s consider the second call of `plus_one` in Listing 6-5 where `x` is
`None`. We enter the `match` and compare to the first arm.

```rust,ignore
None => None,
```

It matches! There’s no value to add to, so the program stops and returns the
`None` value on the right side of `=>`. Because the first arm matched, no other
arms are compared.

Combining `match` and enums is useful in many situations. You’ll see this
pattern a lot in Rust code: `match` against an enum, bind a variable to the
data inside, and then execute code based on it. It’s a bit tricky at first, but
once you get used to it, you’ll wish you had it in all languages. It’s
consistently a user favorite.

### Matches Are Exhaustive

There’s one other aspect of `match` we need to discuss. Consider this version
of our `plus_one` function:

```rust,ignore
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        Some(i) => Some(i + 1),
    }
}
```

We didn’t handle the `None` case, so this code will cause a bug. Luckily, it’s
a bug Rust knows how to catch. If we try to compile this code, we’ll get this
error:

```text
error[E0004]: non-exhaustive patterns: `None` not covered
 -->
  |
6 |         match x {
  |               ^ pattern `None` not covered
```

Rust knows that we didn’t cover every possible case and even knows which
pattern we forgot! Matches in Rust are *exhaustive*: we must exhaust every last
possibility in order for the code to be valid. Especially in the case of
`Option<T>`, when Rust prevents us from forgetting to explicitly handle the
`None` case, it protects us from assuming that we have a value when we might
have null, thus making the billion dollar mistake discussed earlier.

### The `_` Placeholder

Rust also has a pattern we can use in situations when we don’t want to list all
possible values. For example, a `u8` can have valid values of 0 through 255. If
we only care about the values 1, 3, 5, and 7, we don’t want to have to list out
0, 2, 4, 6, 8, 9 all the way up to 255. Fortunately, we don’t have to: we can
use the special pattern `_` instead:

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

The `_` pattern will match any value. By putting it after our other arms, the
`_` will match all the possible cases that aren’t specified before it. The `()`
is just the unit value, so nothing will happen in the `_` case. As a result, we
can say that we want to do nothing for all the possible values that we don’t
list before the `_` placeholder.

However, the `match` expression can be a bit wordy in a situation in which we
only care about *one* of the cases. For this situation, Rust provides `if let`.
