## Los strings almacenan texto codificado en UTF-8

Hablamos sobre cadenas de caracteres (Strings) en el Capítulo 4, pero ahora los veremos con más profundidad. Los nuevos "Rustaceans" se quedan atascados en las cadenas de caracteres comúnmente debido a la combinación de tres conceptos: la propensidad de Rust a exponer posbiles errores, las cadenas de caracteres siendo una estructura de datos más complicada de lo que muchos programadores les dan crédito, y UTF-8. Estos conceptos se combinan de una manera que puede parecer difícil cuando vienes
de otros lenguajes de programación.

Esta discusión de cadenas de caracteres está en este capítulo porque las cadenas de caracteres están implementadas como una colección de bytes más algunos métodos para proveer funcionalidades útiles cuando esos bytes son interceptados como texto. En esta sección, vamos a
hablar sobre las operaciones en `String` que todo tipo de colección tiene, tales como crear, actualizar, y leer. También discutiremos la forma en la que `String` es completamente diferente a otras colecciones, concretamente, cómo indexar en una `String` es
complicado por las diferencias entre cómo interpretan las personas y las computadoras los datos de `String`.

### ¿Qué es una cadena de caracteres (String)?

Primero definiremos a qué nos referimos con el término *string*. Rust tiene un solo tipo de cadena de caracteres en el lenguaje central, el cual es el trozo de cadena `str` que es visto usualmente en su forma prestada `&str`. En el Capítulo 4, hablamos sobre *string slices*, las cuales son referencias a algunas cadenas de caracteres de datos cifradas en UTF-8 guardadas en otra parte. Las String
literales, por ejemplo, son guardadas en la salida binaria del programa y son, por lo tanto, trozos de cadena.

El tipo `String` es dado en la biblioteca estándar de Rust en vez de estar cifrado en el lenguaje central y es un tipo de cadena de caracteres cifrado en UTF-8 cultivable, mutable y propio.
Cuando los "Rustaceans" se refieren a “strings” en Rust, ellos usualemnte quieren decir el `String` y los tipos de trozo de cadena `&str`, no sólo uno de esos tipos. A pesar de que esta sección es en garn parte sobre `String`, ambos tipos son muy usados en la biblioteca estándar de Rust y ambos `String` y trozos de cadena son cifrados en UTF-8.

La biblioteca estándar de Rust también incluye una serie de otros tipos de cadenas, tales como `OsString`, `OsStr`, `CString`, y `CStr`. Los cajones de biblioteca pueden proveer inclusomás opciones para almacenar datos de cadenas de caracteres. Similar a la asignación de nombre `*String`/`*Str`, ellos a menudo proveen una variante propia y prestada, justo como `String`/`&str`.
Estos tipos de cadenas pueden almacenar texto en diferentes cifrados o ser representados en la memoria de una manera diferente, por ejemplo. No discutiremos estos otros tipos de cadena en este capítulo; vea su manual API para más sobre cómo usarlos
y cuando cada uno es apropiado.

### Creando una nueva cadena de caracteres

muchas de las mismas operaciones disponibles con `Vec<T>` están disponibles con `String` también, empezando con la función `new` para crear una cadena de caracteres, mostrada en el Listado
8-11:

```rust
let mut s = String::new();
```

<span class="caption">Listing 8-11: Creating a new, empty `String`</span>

Esta línea crea una nueva cadena vacía llamada `s` a la que luego podemos cargar datos. A menudo, tendremos algunos datos iniciales con los que querremos iniciar la cadena. Para esto, usamos el método `to_string`, el cual está disponible en cualquier tipo
que implemente la característica `Display`, la cual las string literales tienen. El listado 8-12
muetra dos ejemplos:

```rust
let data = "initial contents";

let s = data.to_string();

// the method also works on a literal directly:
let s = "initial contents".to_string();
```

<span class="caption">Listing 8-12: Using the `to_string` method to create a
`String` from a string literal</span>

Este código crea una cadena, conteniendo `initial contents`.

Podemos también usar la función `String::from` para crear una `String` desde una string literal. El código en el Listado 8-13 es equivalente al código del Listado 8-12
que usa `to_string`:

```rust
let s = String::from("initial contents");
```

<span class="caption">Listing 8-13: Using the `String::from` function to create
a `String` from a string literal</span>

Debido a que las cadenas de caracteres son usadas de muchas maneras, podemos usar muchas diferentes APIs genericas para las cadenas, proporcionándonos un montón de opciones. Algunas de ellas pueden parecer redundantes, ¡pero todas ellas tienen su lugar! En este caso, `String::from` y `to_string` hacen lo mismo, entonces, la que elijas es una cuestión de estilo.

Recuerda que las cadenas de caracteres están cifradas en UTF-8, así que podemos incluir cualquier dato propiamente cifrado en ellas, como se muestra en el Listado 8-14:

```rust
let hello = String::from("السلام عليكم");
let hello = String::from("Dobrý den");
let hello = String::from("Hello");
let hello = String::from("שָׁלוֹם");
let hello = String::from("नमस्ते");
let hello = String::from("こんにちは");
let hello = String::from("안녕하세요");
let hello = String::from("你好");
let hello = String::from("Olá");
let hello = String::from("Здравствуйте");
let hello = String::from("Hola");
```

<span class="caption">Listing 8-14: Storing greetings in different languages in
strings</span>

Todos estos son valores válidos de `String`.

### Actualizando una cadena de caracteres

Una `String` puede incrementar su tamaño y su contenido puede cambiar, justo como el contenido de un `Vec<T>`, poniendo más datos en él. En adición, podemos usar convenientemente el operador `+` o la Macro `format!` para encadenar valores de `String` juntos.

#### Añadiendo a una cadena de caracteres con `push_str` y `push`

Podemos aumentar el tamaño de `String` usando el método `push_str` para añadir un trozo de cadena, como se muestra en el Listado 8-15:

```rust
let mut s = String::from("foo");
s.push_str("bar");
```

<span class="caption">Listing 8-15: Appending a string slice to a `String`
using the `push_str` method</span>

Después de estas dos líneas, `s` contendrá `foobar`. El método `push_str` toma un trozo de cadena porque no queremos necesariamente tomar posesión del parametro. Por ejemplo, el código en el Listado 8-16 muestra que sería desafortunado si no estuviéramos capaces de usar `s2` después de añadir su contenido a `s1`:

```rust
let mut s1 = String::from("foo");
let s2 = "bar";
s1.push_str(&s2);
println!("s2 is {}", s2);
```

<span class="caption">Listing 8-16: Using a string slice after appending its
contents to a `String`</span>

Si el método `push_str` toma posesión de `s2`, nosotros no seríamos capaces de poner su valor en la última línea. Sin embargo, ¡este código funciona como hemos esperado!

El método `push` toma un solo caracter como parámetro y lo añade a el `String`. El Listado 8-17 muestra un código que añade el caracter letra l a una `String` usando el método `push`:

```rust
let mut s = String::from("lo");
s.push('l');
```

<span class="caption">Listing 8-17: Adding one character to a `String` value
using `push`</span>

Como resultado de este código, `s` contendrá `lol`.

#### Encadenamiento con el operador `+` o la Macro `format!`

A menudo, querremos combinar dos cadenas de caracteres existentes. Una forma es usar el operador `+`, como se muestra en el Listado 8-18:

```rust
let s1 = String::from("Hello, ");
let s2 = String::from("world!");
let s3 = s1 + &s2; // Note that s1 has been moved here and can no longer be used
```

<span class="caption">Listing 8-18: Using the `+` operator to combine two
`String` values into a new `String` value</span>

La cadena `s3` contendrá `Hello, world!` como resultado de este códiigo. La razón `s1` ya no es is válida luego de la adición y la razón que usamos en referencia a `s2` tiene que ver con la firma del método que es llamado cuando usamos el operador `+`. El operador `+` usa el método `add`, cuya firma luce algo así:

```rust,ignore
fn add(self, s: &str) -> String {
```

Esta no es la firma exacta que está en la biblioteca estándar: En la biblioteca estándar, `add` es definido usando genéricos. Aquí, estamos echando un vistazo a la firma de `add` con tipos concretos substituidos por los genéricos, que es lo que pasa cuando convocamos este método con valores de `String`. Descutiremos genéricos en el Capítulo 10. Esta firma nos da las pistas que necesitamos para entener los bits complicados del operador `+`.

Primero, `s2` tiene un `&`, queriendo decir que estamos añadiendo una *referencia* de la segunda cadena de caracteres a la segunda cadena debido al parámetro `s` en la función `add`: solo podemos añadir una `&str` a una `String`; no podemos añadir dos valores de `String`
juntos. Pero espera - el tipo de `&s2` es `&String`, no `&str`, como se especifica en el segundo parámetro a `add`. Entonces,  ¿por qué el Listado 8-18 compila?

La razón por al cual somos capaces de usar `&s2` en la convocatoria a `add` es que el compilador puede *coerce* el argumento `&String` en una `&str`. Cuando convocamos el método `add`, Rust usa una *deref coercion*, el cual vuelve a `&s2` en `&s2[..]`.
Discutiremos el deref coercion con mayor profundidad en el Capítulo 15. Porque `add` no toma posesión del parámetro `s`, `s2` seguirá siendo un `String` válido luego de esta operación.

Segundo, podemos ver en la firma que `add` toma posesión de `self`, porque `self` *not* tiene una `&`. Esto significa que el `s1` en el Listado 8-18 será movido a la convocatoria `add` y que ya no será válida luego de eso. Así que aunque `let s3 = s1 + &s2;` parece que va a copiar ambas cadenas y crear una nueva, este declaración en realidad toma posesión de `s1`, añade una copia de los contenidos de
`s2`, y luego regresa posesión del resultado. En otras palabras, pararece que está haciendo un montón de copias, pero no: la implementación es más eficiente que el copiado.

Si necesitamos encadenar múltiples cadenas de caracteres, el comportamiento de `+` se vuelve difícil de manejar:

```rust
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");

let s = s1 + "-" + &s2 + "-" + &s3;
```

En este punto, `s` será `tic-tac-toe`. Con todos los caracteres `+` y `"`, será difícil ver qué está pasando. Para más cadenas complicadas combinándose, podemos usar la Macro `format!`:

```rust
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");

let s = format!("{}-{}-{}", s1, s2, s3);
```

Este código también establece a `s` para `tic-tac-toe`. La Macro `format!` funciona en la misma forma que `println!`, pero en lugar de poner la salida en la pantalla, devuelve una `String` con los contenidos. La versión del código que usa `format!` es mucho más fácil de leer y tampoco toma posesión de ninguno de sus parámetros.

### Indexación en cadenas de caracteres

En muchos otros lenguajes de programación, acceder a caracteres individuales en una cadena de caracteres haciendo referencia de ellos indexando es una operación válida y común. Sin embargo, si intentamos acceder a partes de una `String` usando sintaxis de indexación en Rust, tendremos un error. Considere el código valido en Listado 8-19:

```rust,ignore
let s1 = String::from("hello");
let h = s1[0];
```

<span class="caption">Listing 8-19: Attempting to use indexing syntax with a
String</span>

Este código resultará en el siguiente error:

```text
error[E0277]: the trait bound `std::string::String: std::ops::Index<{integer}>` is not satisfied
 -->
  |
3 |     let h = s1[0];
  |             ^^^^^ the type `std::string::String` cannot be indexed by `{integer}`
  |
  = help: the trait `std::ops::Index<{integer}>` is not implemented for `std::string::String`
```

El error y la nota cuentan la historia: Las cadenas de caracteres de Rust no soportan indexado. ¿Pero por qué no? Para responder esa pregunta, necesitamos discutir cómo Rust almacena cadenas en la memoria.

#### Representación interna

Una `String` es un envoltorio sobre un `Vec<u8>`. Echemos un vistazo a algunas de nuestras cadenas de ejemplo correctamente cifradas en UTF-8 del Listado 8-14. Primero, esta:

```rust
let len = String::from("Hola").len();
```

En este caso, `len` será cuatro, lo que se refiere a `Vec` almacenando la cadena “Hola” que pesa cuatro bytes. Cada una de esas letras pesa un byte cuando se codifican en UTF-8. Pero, ¿qué pasa con la siguiente línea?

```rust
let len = String::from("Здравствуйте").len();
```

Note que esta cadena empieza con la letra capital cirílica Ze, no con el número arábico 3. Preguntado cuánto pesa la cadena, podrías decir 12. Sin embargo, la respuesta de Rust es 24: ese es el número de bytes que toma codificar “Здравствуйте” en UTF-8, porque cada valor escalar de Unicode usa dos bytes de almacenamiento. Por lo tanto, un índice en los bytes de la cadena no siempre se correlacionará
con un valor escalar de Unicode. Para demostrarlo, considere este codígo inválido de Rust:

```rust,ignore
let hello = "Здравствуйте";
let answer = &hello[0];
```

¿Cuál debería ser el valor de `answer`? Debería ser `З`, ¿la primera letra? Cuando se codifica en UTF-8, el primer byte de `З` es `208`, y el segundo es `151`, así que `answer` debería ser, de hecho, `208`, pero `208` no es un caracter válido por sí mismo. Volviendo, `208` no es probalemente lo que un usuario querría si ellos preguntan por la primera letra de esta cadena; sin embargo, esa es la unica información que Rust tiene en el byte indexado 0. Volviendo, el valor del byte no es probablemente lo que los usuarios quieren, incluso si la cadena sólo contiene letras latinas: si `&"hello"[0]` era un código válido que regresaba el valor del byte, regresaría `104`, no `h`. Para evitar que se regrese un valor inesperado y cause bugs que puede que no sean descubiertos inmediatamente, Rust no recopila este código del todo y previene malentendidos antes en el proceso de desarrollo.

#### ¡Bytes y valores escalares y grafema clusters! Oh Dios!

Otro punto sobre UTF-8 es que en realidad hay tres formas relevantes de echar un vistazo a las cadenas de caracteres desde la perspectiva de Rust: con bytes, valores escalares, y grafema clusters (lo más cercano a lo que llamaríamos *letters*).

Si echamos un vistazo a la palabra hindi “नमस्ते” escrita en el script Devanagari, es guardada por último como `Vec` del valor `u8` que se ve así:

```text
[224, 164, 168, 224, 164, 174, 224, 164, 184, 224, 165, 141, 224, 164, 164,
224, 165, 135]
```

Eso es 18 bytes y es cómo las computadoras guardan finalmente esta información. Si les echamos un vistazo como un valor escalar de Unicode, el cual es lo que el tipo de `char` de Rust es, esos bytes se ven así:

```text
['न', 'म', 'स', '्', 'त', 'े']
```

Hay seis valores `char` aquí, pero el cuarto y el quinto no son letras: son diacríticos que no tienen sentido por sí mismos. Finalmente, si les echamos un vistazo como un grafema clusters, tendremos lo que una persona llamaría cuatro letras que forman la palabra hindi:

```text
["न", "म", "स्", "ते"]
```

Rust proporciona diferentes maneras de interpretar las información bruta de las cadenas que las computadoras almacenan, así que cada programa puede escoger  la interpretación que necesite, sin importar en qué idioma humano esté la infromación.

Una razón final de por qué Rust no nos permite indexar en una `String` para obtener un caracter es que se espera que las operaciones de indexación siempre tomen un tiempo constante (O(1)). Pero es imposible garantizar ese desempeño con una `String`, porque Rust tendría que guíar los contenidos desde el inicio hasta el índice para determinar cuántos caracteres válidos hubo.

### Dividiendo Strings

Indexar en una cadena de caracteres es a menudo una mala idea porque no está claro lo que el tipo de retorno de la operación de indexación de cadenas debería ser: un valor de byte, un caracter, un grafema cluster, o un trozo de cadena. Por lo tanto, Rust te pide ser más específico si de verdad necesitas usar indices para crear trozos de cadena. Para ser más específico en tu indexado e indicar que quieres un trozo de cadena, en lugar de indexar usando `[]` con un solo número, puedes usar `[]` con un rango para crear un trozo de cadena que contenga bytes en particular:

```rust
let hello = "Здравствуйте";

let s = &hello[0..4];
```

Aquí, `s` será un `&str` que contiene los primeros cuatro bytes de la cadena. Anteriormente, mencionamos que cada uno de estos caracteres pesaban dos bytes, lo que significa que `s` será `Зд`.

¿Qué pasaría si usamos `&hello[0..1]`? La respuesta: Rust se aterrorizará al momento de ejecución de la misma forma que cuando accede a un índice inválido en un vector:

```text
thread 'main' panicked at 'byte index 1 is not a char boundary; it is inside 'З' (bytes 0..2) of `Здравствуйте`', src/libcore/str/mod.rs:2188:4
```

Deberías usar rangos para crear trozos de cadena con precaución, porque puede colgar tu programa.

### Métodos para iterar sobre cadenas

Afortunadamente, podemos acceder a elementos en una cadena de otras maneras.

Si necesitamos realizar operación en un valor escalar individual de Unicode, la mejor forma de hacerlo es usar el método `chars`. Convocando `chars` en “नमस्ते” se seleccionan y retornan seis valores del tipo `char`, y podemos iterar sobre el resultado
con el fin de acceder a cada elemento:

```rust
for c in "नमस्ते".chars() {
    println!("{}", c);
}
```

Este código mostrará lo siguiente:

```text
न
म
स
्
त
े
```

El método `bytes` retorna cada byte puro, lo cual podría ser apropiado para tu dominio:

```rust
for b in "नमस्ते".bytes() {
    println!("{}", b);
}
```

Este código mostrará los 18 bytes que forman esta `String`, empezando con:

```text
224
164
168
224
// ... etc
```

Pero asegúrate de recordar que un valor escalar de Unicode puede ser hecho por más de un byte.

Conseguir un grafema clusters desde cadenas de caracteres es complejo, así que esta funcionalidad no es proporcionada por la biblioteca estándar. Las cajas están disponibles en [crates.io](https://crates.io) si esta es la funcionalidad que necesitas.

### Las cadenas de caracteres no son tan simples

Resumiendo, las cadenas de caracteres son complicadas. Los diferentes lenguajes de programación hacen diferentes elecciones sobre cómo presentar esta complejidad al programador. Rust ha elegido hacer el manejo correcto de la información de `String`. El comportamiento por defecto para todos los programas de Rust, lo que significa que los programadores tienen que pensar más en manejar información de UTF-8 por adelantado. Este punto intermedio expone más la complejidad de las cadenas de caracteres que de otros lenguajes de programación, pero le impide tener errores de manejo que hacen referencia a caracteres no ASCII luego en tu ciclo de vida de desarrollo.

Cambiemos a algo un poquito menos complejo: ¡mapas hash!
