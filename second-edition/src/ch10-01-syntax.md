## Tipos de datos genéricos

Usando los genéricos donde usualmente ponemos tipos, como en firmas de función
o estructuras, nos deja crear definiciones que podemos usar para tipos 
de datos concretos. Miremos cómo definir funciones, estructuras, enum y
métodos usando genéricos, y al final de esta sección discutiremos el
desempeño del código usando genéricos.

### Usando Tipos de Datos Genéricos en la Definición de Funciones

Podemos definir funciones que usan genéricos en la firma de la función
donde los tipos de datos de los parametros y los valores de respuesta van. De esta manera,
el código que escribimos puede ser más flexible y proveer más funcionalidad a los llamados de
nuestra función, aunque no introduzcamos una duplicación de código.

Continuando con nuestra función `largest`, el listado 10-4 muestra dos funciones
que proveen la misma funcionalidad para encontrar el valor más grande en un pedazo. La
primera función es la que extrajimos en el listado 10-3 que encuentra el más grande
`i32` en un pedazo. La segunda función encuentra el más grande `char` en un pedazo:

<span class="filename">Nombre del archivo: src/main.rs</span>

```rust
fn largest_i32(list: &[i32]) -> i32 {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn largest_char(list: &[char]) -> char {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest_i32(&number_list);
    println!("The largest number is {}", result);
#    assert_eq!(result, 100);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest_char(&char_list);
    println!("The largest char is {}", result);
#    assert_eq!(result, 'y');
}
```

<span class="caption">Listado 10-4: Dos funciones que difieren solo en sus
nombres y los tipos en sus firmas</span>

Aquí, las funciones `largest_i32` y `largest_char` tienen exactamente el mismo cuerpo,
así que sería bueno si pudieramos convertir estas dos funciones en una y eliminar
la duplicación. Afortunadamente, ¡Podemos hacer eso introduciendo un parámetro de un tipo
genérico!

Para parametrizar los tipos en las firmas de una función, vamos a
definir, necesitamos crear un nombre para el tipo de parámetro, justo como nosotros le damos
nombres para los valores del parametro de una función. Vamos a escoger el nombre
`T`. Cualquier identificador puede ser usado como el nombre para un tipo de parametro, pero escogeremos
`T` porque la convención de nombres de Rust es Camelcase. Los parámetros de nombre genéricos
también tienden a ser cortos por conveniencia, usualmente solo una letra. Diminutivo para
“type”, `T` es la opción por defecto para la mayoría de los programadores de Rust.

Cuando usamos un parametro en el cuerpo de la función, debemos declarar el
parametro en la firma para que el compilador sepa qué es lo que ese nombre en el
cuerpo significa. De manera similar, cuando usamos un nombre de tipo de parametro en una 
firma de función, debemos declarar el nombre de tipo de parametro antes de que lo usemos. Las declaraciones
de los nombres de los tipos van en paréntesis angulares entre el nombre de la función y
la lista de parametros.

La firma de la función de la función genérica `largest` que vamos a definir 
lucirá así:

```rust,ignore
fn largest<T>(list: &[T]) -> T {
```

Leeríamos esto como: la función `largest` es genérica sobre un tipo `T`. 
Tiene un parámetro llamado `list`, y el tipo de `list` es un pedazo de valores del
tipo `T`. La función `largest` dará un valor del mismo tipo `T`.

El listado 10-5 muestra la función unificada de `largest` usando tipos de datos genéricos
en su firma, y muestra cómo podríamos llamar a `largest` con
un pedazo de los valores de  `i32` o los valores de `char`. ¡Nota que este código no
compilará aun!

<span class="filename">Nombre del archivo: src/main.rs</span>

```rust,ignore
fn largest<T>(list: &[T]) -> T {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest(&number_list);
    println!("The largest number is {}", result);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest(&char_list);
    println!("The largest char is {}", result);
}
```

<span class="caption">Listado 10-5: Una definición de la función `largest` que 
usa tipos de parametros genéricos pero no compila aun</span>

Si tratásemos de compilar este código ahora mismo, nos daría este error:

```text
error[E0369]: binary operation `>` cannot be applied to type `T`
  |
5 |         if item > largest {
  |            ^^^^
  |
note: an implementation of `std::cmp::PartialOrd` might be missing for `T`
```

La nota menciona `std::cmp::PartialOrd`, el cual es un *rasgo*. Vamos a 
hablar sobre los rasgos en la siguiente sección, pero brevemente, lo que el error dice
es que el cuerpo de `largest` no trabajará por todos los tipos posibles que `T` podría 
ser; ya que nosotros queremos comparar los valores del tipo `T` en el cuerpo, solo podemos usar
tipos que sepan como ser ordenados. La libreria estandar ha definido el rasgo
`std::cmp::PartialOrd` que los tipos pueden implementar para habilitar las comparaciones. 
Volveremos a hablar de los rasgos y como especificar que un tipo genérico tiene un 
rasgo particular en la siguiente sección, pero veamos este ejemplo por un momento y
explorar otros lugares en donde podemos usar los tipos genéricos de parametros primero.

<!-- Liz: this is the reason we had the topics in the order we did in the first
draft of this chapter; it's hard to do anything interesting with generic types
in functions unless you also know about traits and trait bounds. I think this
ordering could work out okay, though, and keep a stronger thread with the
`longest` function going through the whole chapter, but we do pause with a
not-yet-compiling example here, which I know isn't ideal either. Let us know
what you think. /Carol -->

### Usando tipos de datos genéricos en definiciones de estructuras

Podemos definir estructuras para usar tipos de parametros genericos en uno o más de los
campos de estructura con la sintaxis `<>` también. El listado 10-6 muestra la definición y
el uso de una estructura `Point` que pueda albergar los valores de cordenadas `x` y `y` de cada tipo:

<span class="filename">Nombre del archivo: src/main.rs</span>

```rust
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let integer = Point { x: 5, y: 10 };
    let float = Point { x: 1.0, y: 4.0 };
}
```

<span class="caption">Listado 10-6: A `Point` estructura que `x` y `y`
albergan los valores del tipo `T`</span>

La sintaxis es similar a usar genéricos en definiciones de funciones. Primero, tenemos
que declarar el nombre del tipo de parametro dentro de parentesis angulares justo luego del
nombre de la estructura. Entonces podemos usar el tipo genérico en la definición de la estructura
donde especificariamos tipos de datos concretos.

Nota que ya que solo hemos usado un tipo genérico en la definición de 
`Point`, lo que estamos diciendo es que la estructura `Point` es genérica sobre un tipo
`T`, y los campos `x` y `y` son *ambos* ese mismo tipo, como sea que termine
siendo. Si tratamos de crear una instancia de un `Point` que tenga valores de
diferentes tipos, como en el listado 10-7, nuestro codigo no compilara:

<span class="filename">Nombre de archivo: src/main.rs</span>

```rust,ignore
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let wont_work = Point { x: 5, y: 4.0 };
}
```

<span class="caption">Listado 10-7: Los campos `x` y `y` deberan ser los mismos
tipos porque ambos tienen el mismo tipo de dato genérico `T`</span>

Si tratasemos de compilar esto, nos daría el siguiente error:

```text
error[E0308]: mismatched types
 -->
  |
7 |     let wont_work = Point { x: 5, y: 4.0 };
  |                                      ^^^ expected integral variable, found
  floating-point variable
  |
  = note: expected type `{integer}`
  = note:    found type `{float}`
```

Cuando le asignamos un valor entero de 5 a `x`, el compilador entonces sabe que para esta 
instancia de `Point` que el tipo genérico `T` será un número entero. Entonces nosotros
especificamos 4.0 para `y`, lo que es definido para que tenga el mismo tipo `x`, entonces tenemos un
error de desajuste de tipos.

Si quisieramos definir una estructura `Point` donde `x` y `y` pudieran tener diferentes
tipos pero aun haría que esos tipos fueran genéricos, podemos usar multiples parametros de tipos
genéricos. En el listado 10-8, hemos cambiado la definición de `Point` para que fuese
generica sobre tipos `T` y `U`. El campo `x` es de tipo `T`, y el campo `y`
es de tipo `U`:

<span class="filename">Nombre del archivo: src/main.rs</span>

```rust
struct Point<T, U> {
    x: T,
    y: U,
}

fn main() {
    let both_integer = Point { x: 5, y: 10 };
    let both_float = Point { x: 1.0, y: 4.0 };
    let integer_and_float = Point { x: 5, y: 4.0 };
}
```

<span class="caption">Listado 10-8: Un `Point` genérico sobre dos tipos para que 
`x` y `y` puedan ser valores de dos tipos diferentes</span>

¡Ahora todas estas instancias de `Point` están permitidas! Podrás usar cuantos tipos de parámetros
de tipo genérico quieras en una definición, pero el usar más de unos cuantos se hace
dificil de leer y entender. Si llegas al punto de necesitar muchos tipos
genéricos, es probablemente un signo de que tu código podría necesitar un poco de reestructuración para
ser separado en piezas más pequeñas.

### Usando Tipos de Datos Genéricos en Definiciones Enum

Similares a las estructuras, Los Enums pueden ser definidos para albergar tipos de datos genéricos en sus
variantes. Usamos el enum `Option<T>` proporcionado por la biblioteca estándar en el
Capítulo 6, y ahora su definición debería de tener más sentido. Echemos
otro vistazo:

```rust
enum Option<T> {
    Some(T),
    None,
}
```

En otras palabras, `Option<T>` es un Enum genérico del tipo `T`. El cual tiene dos
variantes: `Some`, que alberga un valor de tipo `T`, y una variante `None` que 
no alberga ningún valor. La biblioteca estándar solo debe de tener esta única
definición para respaldar la creación de valores en este enum que tenga cualquier
tipo concreto. La idea de “un valor opcional” es más un concepto abstracto que
un tipo específico, y Rust nos deja expresar este concepto abstracto sin muchas
duplicaciones.

Enums también pueden usar múltiples tipos genéricos. La definición del enum `Result`
que usamos en el capítulo 9 es un ejemplo:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

El Enum `Result` es genérico sobre dos tipos, `T` y `E`. `Result` tiene dos
variantes: `Ok`, la cual alberga un valor de tipo `T`, y `Err`, el cual alberga un valor
de tipo `E`. Esta definición hace conveniente el usar el enum `Result` 
en donde queramos tener una operación que pueda tener éxito (y dar un valor de algún
tipo `T`) o fallar (y dar un error de algún tipo `E`). Recuerda el listado 9-2
donde abrimos un archivo: en ese caso, `T` estaba rellenado con el tipo
`std::fs::File` cuando el archivo se abría con éxito y `E` estaba rellenado
con el tipo `std::io::Error` cuando habían problemas abriendo el archivo.

Cuando reconoces situaciones en tu código con múltiple estructura o definiciones
Enum que eran diferentes solo en los tipos de los valores que albergaban, puedes
retirar la duplicación al usar el mismo proceso que usamos con las definiciones de
función para introducir tipos genéricos en su lugar.

### Usando Tipos de Datos Genéricos en Definiciones de Métodos

Como hicimos en el capítulo 5, podemos implementar métodos en estructuras y Enums que
tengan tipos genéricos en sus definiciones. El listado 10-9 muestra la estructura `Point<T>`
que definimos en el listado 10-6. Hemos entonces identificado un método llamado `x` en
`Point<T>` que da una referencia a los datos en el campo `x`:

<span class="filename">Number of archive: src/main.rs</span>

```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}

fn main() {
    let p = Point { x: 5, y: 10 };

    println!("p.x = {}", p.x());
}
```

<span class="caption">Listado 10-9: Implementando un método llamado `x` en la estructura
`Point<T>` que dará una referencia al campo `x`, la cual es del 
tipo `T`.</span>

Nota que tenemos que declarar `T` justo luego de `impl` para usar `T` en el
tipo `Point<T>`. Declarando `T` como un tipo genérico antes del `impl` es como Rust
sabe que el tipo en los corchetes angulares en `Point` es un tipo genérico en lugar de un
tipo concreto. por ejemplo, podríamos escoger el implementar métodos en instancias
`Point<f32>` en lugar de instancias `Point` con cualquier tipo genérico.
El listado 10-10 muestra que no declaramos nada luego del `impl` en este 
caso, ya que estamos usando el tipo concreto, `f32`:

```rust
# struct Point<T> {
#     x: T,
#     y: T,
# }
#
impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}
```

<span class="caption">Listado 10-10: Construyendo un bloqueo `impl`que solo 
aplica a una estructura con un tipo específico es usada para el parámetro de tipo genérico
 `T`</span>

Este código significa que el tipo `Point<f32>` tendrá un método llamado
`distance_from_origin`, y otras instancias de `Point<T>` donde `T` no es de 
tipo `32` no tendrá este método definido. Este método mide que tan lejos es nuestro
punto del punto de las coordinadas (0.0, 0.0) y usa operaciones
matemáticas que solo son disponibles para tipos de punto flotante-

parámetros de tipo genérico en una definición de estructura no son siempre el mismo 
parámetro de tipo que tú quieres usar en firmas de método de estructura. El listado
10-11 define un método `mixup` en la estructura `Point<T, U>` desde el listado 10-8.
El método toma otro `Point` como un parámetro, el que puede tener diferentes 
tipos que el `self` `Point` que estamos nombrando en `mixup`. El método crea
una nueva instancia `Point` que tiene el valor `x` del `self` `Point` (que es
de tipo `T`) y el valor `y` del `Point` aprobado (el cual es de tipo
 `W`):

<span class="filename">Nombre del archivo: src/main.rs</span>

```rust
struct Point<T, U> {
    x: T,
    y: U,
}

impl<T, U> Point<T, U> {
    fn mixup<V, W>(self, other: Point<V, W>) -> Point<T, W> {
        Point {
            x: self.x,
            y: other.y,
        }
    }
}

fn main() {
    let p1 = Point { x: 5, y: 10.4 };
    let p2 = Point { x: "Hello", y: 'c'};

    let p3 = p1.mixup(p2);

    println!("p3.x = {}, p3.y = {}", p3.x, p3.y);
}
```

<span class="caption">Listado 10-11: Metodos que usan diferentes tipos genéricos
que su definición de estructura</span>

En `main`, hemos definido un `Point` que tiene un `i32` por `x` (con valor `5`)
y un `f64` por `y` (con valor `10.4`). `p2` es un `Point` que tiene un pedazo de cordón de `x` (con valor `"Hello"`) y un `char` por `y` (con valor `c`).
El llamar `mixup` en `p1` con el argumento `p2` nos da `p3`, que tendrá
un `i32` por `x`, ya que `x` vino de `p1`. `p3` tendrá un `char` por `y`,
ya que `y` vino de `p2`. El `println!` emitirá un `p3.x = 5, p3.y = c`.

Date cuenta de que los parámetros genéricos `T` y `U` son declarados después de `impl`, ya que
ellos van con la definición de la estructura. Los parámetros genéricos `V` y `W` son
declarados luego de `fn mixup`, que ellos solo son relevantes para el método.

### El Desempeño del Código Usando Genéricos

Te estarás preguntando mientras leías esta sección si hay un tiempo de coste en el tiempo de ejecución
al usar tipos de parámetros genéricos. Tengo buenas noticias: ¡La forma en la que Rust ha implementado los
genéricos significa que tu código no se ejecutará más lento como si hubieses especificado
tipos concretos en vez de tipos de parámetros genéricos!

Rust logra esto al realizar una *monotransformación* del código usando los genéricos
al tiempo de compilar. La monotransformación es el proceso de convertir un código genérico en un
código especificado con tipos concretos que de hecho se usan rellenos.


Lo que el compilador hace es lo opuesto de los pasos que hemos realizado para crear
la función genérica en el listado 10-5. El compilador mira por todos los lugares que el
código genérico ha sido llamado y genera códigos para los tipos concretos con los que el
código genérico es llamado.

Trabajemos con un ejemplo que usa el enum `Option` en la biblioteca estándar:

```rust
let integer = Some(5);
let float = Some(5.0);
```

Cuando Rust compila este código, este realiza la monotransformación. El compilador
leerá los valores que han sido pasados a `Option` y verá que tenemos dos
tipos de `Option<T>`: uno es `i32`, y uno es `f64`. Y así, expandirá
la definición genérica de `Option<T>` en `Option_i32` y `Option_f64`,
y así reemplazando la definición genérica con las especificadas.

La versión monotransformada de nuestro código que el compilador genera lucirá
así, con los usos del genérico `Option` reemplazados con las definiciones
especificas creadas por el compilador:

<span class="filename">Nombre del archivo: src/main.rs</span>

```rust
enum Option_i32 {
    Some(i32),
    None,
}

enum Option_f64 {
    Some(f64),
    None,
}

fn main() {
    let integer = Option_i32::Some(5);
    let float = Option_f64::Some(5.0);
}
```

Podemos escribir códigos no duplicados usando genéricos, y Rust compilará eso
en un código que especifica el tipo en cada instancia. eso significa que no pagamos
costos de tiempos de ejecución por usar genéricos; cuando el código se ejecuta, se lleva a cabo como si
lo hubiésemos duplicado cada definición particular a mano. El proceso de 
monotransformación es lo que hace los genéricos de Rust extremamente eficientes en el tiempo de ejecución.
