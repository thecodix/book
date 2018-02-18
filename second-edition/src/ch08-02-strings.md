## Strings Store UTF-8 Encoded Text

We talked about strings in Chapter 4, but we’ll look at them in more depth now.
New Rustaceans commonly get stuck on strings due to a combination of three
concepts: Rust’s propensity for exposing possible errors, strings being a more
complicated data structure than many programmers give them credit for, and
UTF-8. These concepts combine in a way that can seem difficult when you’re
coming from other programming languages.

This discussion of strings is in the collections chapter because strings are
implemented as a collection of bytes plus some methods to provide useful
functionality when those bytes are interpreted as text. In this section, we’ll
talk about the operations on `String` that every collection type has, such as
creating, updating, and reading. We’ll also discuss the ways in which `String`
is different than the other collections, namely how indexing into a `String` is
complicated by the differences between how people and computers interpret
`String` data.

### What Is a String?

We’ll first define what we mean by the term *string*. Rust has only one string
type in the core language, which is the string slice `str` that is usually seen
in its borrowed form `&str`. In Chapter 4, we talked about *string slices*,
which are references to some UTF-8 encoded string data stored elsewhere. String
literals, for example, are stored in the binary output of the program and are
therefore string slices.

The `String` type is provided in Rust’s standard library rather than coded into
the core language and is a growable, mutable, owned, UTF-8 encoded string type.
When Rustaceans refer to “strings” in Rust, they usually mean the `String` and
the string slice `&str` types, not just one of those types. Although this
section is largely about `String`, both types are used heavily in Rust’s
standard library and both `String` and string slices are UTF-8 encoded.

Rust’s standard library also includes a number of other string types, such as
`OsString`, `OsStr`, `CString`, and `CStr`. Library crates can provide even
more options for storing string data. Similar to the `*String`/`*Str` naming,
they often provide an owned and borrowed variant, just like `String`/`&str`.
These string types can store text in different encodings or be represented in
memory in a different way, for example. We won’t discuss these other string
types in this chapter; see their API documentation for more about how to use
them and when each is appropriate.

### Creating a New String

Many of the same operations available with `Vec<T>` are available with `String`
as well, starting with the `new` function to create a string, shown in Listing
8-11:

```rust
let mut s = String::new();
```

<span class="caption">Listing 8-11: Creating a new, empty `String`</span>

This line creates a new empty string called `s` that we can then load data
into. Often, we’ll have some initial data that we want to start the string
with. For that, we use the `to_string` method, which is available on any type
that implements the `Display` trait, which string literals do. Listing 8-12
shows two examples:

```rust
let data = "initial contents";

let s = data.to_string();

// the method also works on a literal directly:
let s = "initial contents".to_string();
```

<span class="caption">Listing 8-12: Using the `to_string` method to create a
`String` from a string literal</span>

This code creates a string containing `initial contents`.

We can also use the function `String::from` to create a `String` from a string
literal. The code in Listing 8-13 is equivalent to the code from Listing 8-12
that uses `to_string`:

```rust
let s = String::from("initial contents");
```

<span class="caption">Listing 8-13: Using the `String::from` function to create
a `String` from a string literal</span>

Because strings are used for so many things, we can use many different generic
APIs for strings, providing us with a lot of options. Some of them can seem
redundant, but they all have their place! In this case, `String::from` and
`to_string` do the same thing, so which you choose is a matter of style.

Remember that strings are UTF-8 encoded, so we can include any properly encoded
data in them, as shown in Listing 8-14:

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

All of these are valid `String` values.

### Updating a String

A `String` can grow in size and its contents can change, just like the contents
of a `Vec<T>`, by pushing more data into it. In addition, we can conveniently
use the `+` operator or the `format!` macro to concatenate `String` values
together.

#### Appending to a String with `push_str` and `push`

We can grow a `String` by using the `push_str` method to append a string slice,
as shown in Listing 8-15:

```rust
let mut s = String::from("foo");
s.push_str("bar");
```

<span class="caption">Listing 8-15: Appending a string slice to a `String`
using the `push_str` method</span>

After these two lines, `s` will contain `foobar`. The `push_str` method takes a
string slice because we don’t necessarily want to take ownership of the
parameter. For example, the code in Listing 8-16 shows that it would be
unfortunate if we weren’t able to use `s2` after appending its contents to `s1`:

```rust
let mut s1 = String::from("foo");
let s2 = "bar";
s1.push_str(&s2);
println!("s2 is {}", s2);
```

<span class="caption">Listing 8-16: Using a string slice after appending its
contents to a `String`</span>

If the `push_str` method took ownership of `s2`, we wouldn’t be able to print
out its value on the last line. However, this code works as we’d expect!

The `push` method takes a single character as a parameter and adds it to the
`String`. Listing 8-17 shows code that adds the letter l character to a
`String` using the `push` method:

```rust
let mut s = String::from("lo");
s.push('l');
```

<span class="caption">Listing 8-17: Adding one character to a `String` value
using `push`</span>

As a result of this code, `s` will contain `lol`.

#### Concatenation with the `+` Operator or the `format!` Macro

Often, we’ll want to combine two existing strings. One way is to use the `+`
operator, as shown in Listing 8-18:

```rust
let s1 = String::from("Hello, ");
let s2 = String::from("world!");
let s3 = s1 + &s2; // Note that s1 has been moved here and can no longer be used
```

<span class="caption">Listing 8-18: Using the `+` operator to combine two
`String` values into a new `String` value</span>

The string `s3` will contain `Hello, world!` as a result of this code. The
reason `s1` is no longer valid after the addition and the reason we used a
reference to `s2` has to do with the signature of the method that gets called
when we use the `+` operator. The `+` operator uses the `add` method, whose
signature looks something like this:

```rust,ignore
fn add(self, s: &str) -> String {
```

This isn’t the exact signature that’s in the standard library: in the standard
library, `add` is defined using generics. Here, we’re looking at the signature
of `add` with concrete types substituted for the generic ones, which is what
happens when we call this method with `String` values. We’ll discuss generics
in Chapter 10. This signature gives us the clues we need to understand the
tricky bits of the `+` operator.

Primero, `s2` tiene un `&`, queriendo decir que estamos añadiendo una *reference* de la segunda
cadena de caracteres a la segunda cadena debido al parámetro `s` en la función `add`:
solo podemos añadir una `&str` a una `String`; no podemos añadir dos valores de `String`
juntos. Pero espera - el tipo de `&s2` es `&String`, no `&str`, como se especifica
en el segundo parámetro a `add`. Entonces,  ¿por qué el Listado 8-18 recopila?

La razón por al cual somos capaces de usar `&s2` en la convocatoria a `add` es que el compilador
puede *coerce* el argumento `&String` en una `&str`. Cuando convocamos el método `add`,
Rust usa una *deref coercion*, el cual vuelve a `&s2` en `&s2[..]`.
Discutiremos el deref coercion con mayor profundidad en el Capítulo 15. Porque `add`
no toma posesión del parámetro `s`, `s2` seguirá siendo un `String` válido
luego de esta operación.

Segundo, podemos ver en la firma que `add` toma posesión de `self`,
porque `self` *not* tiene una `&`. Esto significa que el `s1` en el Listado 8-18 será
movido a la convocatoria `add` y que ya no será válida luego de eso. Así que aunque `let
s3 = s1 + &s2;` parece que va a copiar ambas cadenas y crear una nueva, este
declaración en realidad toma posesión de `s1`, añade una copia de los contenidos de
`s2`, y luego regresa posesión del resultado. En otras palabras, pararece que
está haciendo un montón de copias, pero no: la implementación es más eficiente
que el copiado.

Si necesitamos encadenar múltiples cadenas de caracteres, el comportamiento de `+` se vuelve difícil de manejar:

```rust
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");

let s = s1 + "-" + &s2 + "-" + &s3;
```

En este punto, `s` será `tic-tac-toe`. Con todos los caracteres `+` y `"`,
será difícil ver qué está pasando. Para más cadenas complicadas
combinándose, podemos usar la Macro `format!`:

```rust
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");

let s = format!("{}-{}-{}", s1, s2, s3);
```

Este código también establece a `s` para `tic-tac-toe`. La Macro `format!` funciona en la misma
forma que `println!`, pero en lugar de poner la salida en la pantalla, regresa
una `String` con los contenidos. La versión del código que usa `format!` es mucho más
fácil de leer y tampoco toma posesión de ninguno de sus parámetros.

### Indexación en cadenas de caracteres

En muchos otros lenguajes de programación, acceder a caracteres individuales en una
cadena de caracteres haciendo referencia de ellos indexando es una operación válida y común. Sin embargo,
si intentamos acceder a partes de una `String` usando sintaxis de indexación en Rust, tendremos
un error. Considere el código valido en Listado 8-19:

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

El error y la nota cuentan la historia: Las cadenas de caracteres de Rust no soportan indexado0. ¿Pero
por qué no? Para responder esa pregunta, necesitamos discutir cómo Rust almacena cadenas en
la memoria.

#### Representación interna

Una `String` es un envoltorio sobre un `Vec<u8>`. Echemos un vistazo a algunas de nuestras cadenas
de ejemplo correctamente cifradas en UTF-8 del Listado 8-14. Primero, esta:

```rust
let len = String::from("Hola").len();
```

En este caso, `len` será cuatro, lo que se refiere a `Vec` almacenando la cadena
“Hola” que pesa cuatro bytes. Cada una de esas letras pesa un byte cuando se codifican en
UTF-8. Pero, ¿qué pasa con la siguiente línea?

```rust
let len = String::from("Здравствуйте").len();
```

Note que esta cadena empieza con la letra capital cirílica Ze, no con el
número arábico 3. Preguntado cuánto pesa la cadena, podrías decir 12. Sin embargo,
la respuesta de Rust es 24: ese es el número de bytes que toma codificar
“Здравствуйте” en UTF-8, porque cada valor escalar de Unicode usa dos bytes de
almacenamiento. Por lo tanto, un índice en los bytes de la cadena no siempre se correlacionará
con un valor escalar de Unicode. Para demostralo, considere este codígo inválido de
Rust:

```rust,ignore
let hello = "Здравствуйте";
let answer = &hello[0];
```

¿Cuál debería ser el valor de `answer`? Debería ser `З`, ¿la primera letra? Cuando
se codifica en UTF-8, el primer byte de `З` es `208`, y el segundo es `151`, así que
`answer` debería ser, de hecho, `208`, pero `208` no es un caracter válido por sí
mismo. Volviendo, `208` no es probalemente lo que un usuario querría si ellos preguntan por la
primera letra de esta cadena; sin embargo, esa es la unica información que Rust tiene en
el byte indexado 0. Volviendo, el valor del byte no es probablemente lo que los usuarios quieren, incluso si
la cadena sólo contiene letras latinas: si `&"hello"[0]` era un código válido que
regresaba el valor del byte, regresaría `104`, no `h`. Para evitar que se regrese un
valor inesperado y cause bugs que puede que no sean descubiertos inmediatamente,
Rust no recopila este código del todo y previene malentendidos antes en
el proceso de desarrollo.

#### ¡Bytes y valores escalares y grafema clusters! Oh Dios!

Otro punto sobre UTF-8 es que en realidad hay tres formas relevantes de
echar un vistazo a las cadenas de caracteres desde la perspectiva de Rust: con bytes, valores escalares, y grafema
clusters (lo más cercano a lo que llamaríamos *letters*).

Si echamos un vistazo a la palabra hindi “नमस्ते” escrita en el script Devanagari, es
guardada por último como `Vec` del valor `u8` que se ve así:

```text
[224, 164, 168, 224, 164, 174, 224, 164, 184, 224, 165, 141, 224, 164, 164,
224, 165, 135]
```

Eso es 18 bytes y es cómo las computadoras guardan finalmente esta información. Si les echamos un vistazo
como un valor escalar de Unicode, el cual es lo que el tipo de `char` de Rust es, esos
bytes se ven así:

```text
['न', 'म', 'स', '्', 'त', 'े']
```

Hay seis valores `char` aquí, pero el cuarto y el quinto no son letras:
son diacríticos que no tienen sentido por sí mismos. Finalmente, si les echamos un vistazo
como un grafema clusters, tendremos lo que una persona llamaría cuatro letras
que forman la palabra hindi:

```text
["न", "म", "स्", "ते"]
```

Rust proporciona diferentes maneras de interpretar las información cruda de las cadenas que las computadoras
almacenan, así que cada programa puede escoger  la interpretación que necesite, sin importar
en qué idioma humano esté la infromación.

Una razón final de por qué Rust no nos permite indexar en una `String` para obtener un
caracter es que se espera que las operaciones de indexación siempre tomen un tiempo constante
(O(1)). Pero es imposible garantizar ese desempeño con una `String`,
porque Rust tendría que guíar los contenidos desde el inicio hasta el
índice para determinar cuántos caracteres válidos hubo.

### Trozos de cadena

Indexar en una cadena de caracteres es a menudo una mala idea porque no está claro lo que el
tipo de retorno de la operación de indexación de cadenas debería ser: un valor de byte, un
caracter, un grafema cluster, o un trozo de cadena. Por lo tanto, Rust te pide
ser más específico si de verdad necesitas usar indices para crear trozos de cadena. Para
ser más específico en tu indexado e indicar que quieres un trozo de cadena,
en lugar de indexar usando `[]` con un solo número, puedes usar `[]` con un
rango para crear un trozo de cadena que contenga bytes en particular:

```rust
let hello = "Здравствуйте";

let s = &hello[0..4];
```

Aquí, `s` será un `&str` que contiene los primeros cuatro bytes de la cadena.
Anteriormente, mencionamos que cada uno de estos caracteres pesaban dos bytes, lo que significa que
`s` será `Зд`.

¿Qué pasaría si usamos `&hello[0..1]`? La respuesta: Rust se aterrorizará
al momento de ejecución de la misma forma que cuando accede a un índice inválido en un vector:

```text
thread 'main' panicked at 'byte index 1 is not a char boundary; it is inside 'З' (bytes 0..2) of `Здравствуйте`', src/libcore/str/mod.rs:2188:4
```

Deberías usar rangos para crear trozos de cadena con precaución, porque puede
colgar tu programa.

### Métodos para iterar sobre cadenas

Afortunadamente, podemos acceder a elementos en una cadena de otras maneras.

Si necesitamos realizar operación en un valor escalar individual de Unicode, la mejor
forma de hacerlo es usar el método `chars`. Convocando `chars` en “नमस्ते” se seleccionan
y retornan seis valores del tipo `char`, y podemos iterar sobre el resultado
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

El método `bytes` retorna cada byte puro, lo cual podría ser apropiado para tu
dominio:

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

Pero asegúrate de recordar que un valor escalar de Unicode puede ser hecho por más
de un byte.

Conseguir un grafema clusters desde cadenas de caracteres es complejo, así que esta funcionalidad no es
proporcionada por la biblioteca estándar. Los cajones están disponibles en
[crates.io](https://crates.io) si esta es la funcionalidad que necesitas.

### Las cadenas de caracteres no son tan simples

Resumiendo, las cadenas de caracteres son complicadas. Los diferentes lenguajes de programación hacen
diferentes elecciones sobre cómo presentar esta complejidad al programador. Rust
ha elegido hacer el manejo correcto de la información de `String`. El comportamiento por defecto
para todos los programas de Rust, lo que significa que los programadores tienen que pensar más en
manejar información de UTF-8 por adelantado. Este punto intermedio expone más la complejidad de
las cadenas de caracteres que de otros lenguajes de programación, pero le impide tener
errores de manejo, envolviendo caracteres non-ASCII luego en tu ciclo de vida de
desarrollo.

Cambiemos a algo un poquito menos complejo: ¡mapas hash!
