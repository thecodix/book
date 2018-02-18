## El operador de flujo de control `match` 

Rust tiene un extramadamente poderoso operador de flujo de control llamado `match` 
que nos permite comparar un valor contra una serie de patrones y luego ejecutar código 
basado en que patrón coincide. Los patrones pueden estar compuestos por valores literales, 
nombres de variable, comodines, y muchas otras cosas; El capítulo 18 cubre todos los
diferentes tipos de patrones y que hacen. El poder de `match` viene de la expresividad
de los patrones y el compilador verifica que todos los posibles casos son manejados.

Piense en una expresión de tipo `match` como una máquina clasificadora de monedas: las
monedas se deslizan por una pista con agujeros de distintos tamaños a lo largo de ella, y
cada moneda moneda cae a travpes del primer hoyo que encuentra que encaja. De la misma forma,
los valores van a través de cada patrón en una en un `match`, y en el primer patrón el valor
“encaja,” el valor caerá en el código de bloque asociado para ser usado durante la ejecución.

Ya que acabamos de mencionar a las monedas, utilicemoslas como ejemplo usando `match`! Podemos
escribir una función que pueda tomar una moneda desconocida de United States y en una
forma similar a la máquina contadora, determina que moneda es y devuelve su valor
en cantavos, como se muestra en en Listado 6-3:

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

<span class="caption">Listado 6-3: Una enumeración y una expresión`match` que tiene
las variantes de la enumeración como sus patrones.</span>

Desglosemos el `match` en la función `value_in_cents`. Primero, listamos la
palabra clave `match` seguida de una expresión, que en este caso es el valor
`coin`. Esto parece muy similar a una expresión usada con `if`, pero hay una
gran diferencia: con `if`, la expresión necesita devolver un valor Booleano.
Aquí, puede ser de cualquier tipo. El tipo de `coin` en este ejemplo es la enumeración
`Coin` que es definida en el Listado 6-3.

A continuación están los brazos del `match`. Un brazo tiene dos partes: un patrón
y algún código. El primer brazo aquí tiene un patrón que es el valor `Coin::Penny`
y luego el operador `=>` que separa el patrón y el código a ejecutar. El código en este caso
es solo el valor 1 `1`. Cada brazo está separado del próximo con una coma.

Cuando la expresión `match` ejectua, compara el valor resultante contra el 
patrón de cada brazo, en orden. Si un patrón coincide con el valor, el código
asociado con el patrón es ejecutado. Si el patrón no coincide con el valor, 
la ejecución continúa con el siguiente brazo, como en una maquina clasificadora de monedas.
Podemos tener tantos brazos como necesitemos: En el Listado 6-3, nuestro `match` tiene cuatro brazos.

El cógido asociado con cada brazo es una expresión, y el valor resultante de la
expresión en el brazo correspondiente es el valor que se devuelve para la expresión
entera `match`.

Las llaves tipicamente no son usadas si el código del brazo de coincidencia es corto, como
lo es en el Listado 6-3 donde cada brazo solo retorna un valor. Si quieres ejecutar múltiples
líneas de código en un brazo de coincidencia, puedes usar llaves. Por ejemplo, el
siguiente código imprimirá “Lucky penny!” cada vez que el método sea llamado
con un `Coin::Penny` pero seguirá retornando el último valor del bloque, `1`:

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

### Patrones que se unen a los Valores

Otra característica útil de los brazos de coincidencia es que ellos pueden unirse a partes
de los valores que coinciden con ek patrón. Así es como extraemos valores fuera de las
variantes de enumeración.

Como un ejemplo, cambiemos una de nuestras variantes para mantener los datos dentro de ella.
Desde 1999 hasta 2008, los Estados Unidos inventaron alojamientos con diferentes
diseños para cada uno de los 50 estados de un lado. Ninguna otra moneda obtuvo diseños
de estado, por lo que solo los alojamientos tienen este valor extra. Podemos agregar esta
información a nuestro `enum` usando la variante `Quarter` para incluir un valor `UsState`
almacenado dentro de ella, que hemos hecho en el Listado 6-4:

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

<span class="caption">Listado 6-4: Una enumeración `Coin` donde la variante `Quarter` 
tambien contiene un valor `UsState`</span>

Imaginemos que un amigo nuestro esta intentando recolectar los 50 alojamientos del estado.
Mientras clasificamos nuestro cambio impreciso por tipo de moneda, tambien llamamos el
nombre del estado asociado con cada alojamiento asique si uno de nuestros amigos no tiene,
ellos pueden agregarlo a su colección.

En la expresión de coincidencia para este código, agregamos una variable llamada `state` al
patrón que coincide con el valor de la variante `Coin::Quarter`. Cuando un
`Coin::Quarter` coincide, la variable `state` se juntará al valor del 
estado de alojamientos. Entonces podemos usar `state` en el código para ese brazo, asi:

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

Si tuviésemos que llamar `value_in_cents(Coin::Quarter(UsState::Alaska))`, `coin`
sería `Coin::Quarter(UsState::Alaska)`. Cuando comparamos el valor con cada uno
de los brazos de coincidencia, ninguno de ellos coincide hasta alcanzar `Coin::Quarter(state)`. A
ese punto, el enlace `state` tendrá el valor `UsState::Alaska`. Podemos entonces
usar ese enlace en la expresión `println!`, obteniendo el valor de estado de la variante
de enumeración  `Coin` para `Quarter`.

### Coincidencia con `Option<T>`

En la sección previa queriamos obtener el valor interno `T` del caso `Some`
cuando usamos `Option<T>`; también podemos manejar `Option<T>` usando `match` asi como
hicimos con la enumeración `Coin`! En lugar de comparar monedas, comparamos las variantes
de `Option<T>`, pero la manera en que la expresión `match` funciona es igual.

Digamos que queremos escribir una faunción que tome una `Option<i32>`, y si
tiene un valor adentro, agregamos uno a ese valor. Si no tiene un valor adentro,
la función debería retornar el valor `None` y no intentar realizar ninguna operación.

Esta función es muy fácil de escribir, gracias a `match`, y se verá como en el
Listado 6-5:

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
