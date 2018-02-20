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

Cuando la expresión `match` ejecuta, compara el valor resultante contra el 
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
con un `Moneda::Penny` pero seguirá retornando el último valor del bloque, `1`:

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

En la expresión de coincidencia para este código, agregamos una variable llamada `EstadosUs` al
patrón que coincide con el valor de la variante `Moneda::Quarter`. Cuando un
`Moneda::Quarter` coincide, la variable `EstadosUs` se juntará al valor del 
estado de alojamientos. Entonces podemos usar `EstadosUs` en el código para ese brazo, asi:

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
<span class="caption">Listado 6-5: Una función que usa una expresión `match` en
una `Option<i32>`</span>

#### Emparejando `Some(T)`

Examinaremos la primera ejecución de `plus_one` en más detalles. Cuando llamamos
`plus_one(five)`, la variable `x` en el cuerpo de `plus_one` tendrá el valor
`Some(5)`. Luego lo comparamos contra cada brazo de coincidencia.


```rust,ignore
None => None,
```

El valor `Some(5)` no coincie con el patrón `None`, entonces continuamos al siguiente
brazo.

```rust,ignore
Some(i) => Some(i + 1),
```

Coincide `Some(5)` con `Some(i)`? Si lo hace! Tenemos la misma variante.
La `i` se une al valor contenido en `Some`, entonces `i` toma el valor `5`. El
código en el brazo de coincidencia es entonces ejecutado, asique agregamos uno
al valor de `i` y creamos un nuevo valor `Some` con nuestro total `6` adentro.

#### Emparejando `None`

Ahora consideremos la segunda llamda de `plus_one` en el Listado 6-5 donde `x` es
`None`. Ingreamos el valor `match` y lo comparamos al primer brazo.


```rust,ignore
None => None,
```

Coincide! No hay valor para agregar, entonces el programa se detiene y devuelve el valor
`None` en el lado derecho de `=>`. Debido a que el primer brazo coincide, no se 
comparan otros brazos.

Combinar `match` y enumaraciones en útil en muchas situaciones. Verás mucho este patrón
en el código Rust: `match` contra una enumeración, junta una variable con los
datos adentro, y entonces ejecuta código basado en él. Es un poco díficil ap principio, pero
una vez que lo usas, querrás tenerlo en todos los lenguajes. Es
consistentemente un usuario favorito.

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

No manejamos el caso `None`, entonces este código causará un bug. Por suerte, es un
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
olvidamos! Las coincidencias en Rust son *exhaustive*: debemos usar hasta la última 
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
nos importa *one* de los casos. Para esta situacuión, Rust provee `if let`.
