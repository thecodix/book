## Los strings almacenan texto codificado en UTF-8

Hablamos sobre cadenas de caracteres (Strings) en el Capítulo 4, pero ahora los veremos con más profundidad. Los nuevos "Rustaceans" se quedan atascados en las cadenas de caracteres comúnmente debido a la combinación de tres conceptos: la propensidad de Rust a exponer posbiles errores, las cadenas de caracteres siendo una estructura de datos más complicada de lo que muchos programadores les dan crédito, y UTF-8. Estos conceptos se combinan de una manera que puede parecer difícil cuando vienes de otros lenguajes de programación.

Esta discusión de cadenas de caracteres está en este capítulo porque las cadenas de caracteres están implementadas como una colección de bytes más algunos métodos para proveer funcionalidades útiles cuando esos bytes son interceptados como texto. En esta sección, vamos a hablar sobre las operaciones en `String` que todo tipo de colección tiene, tales como crear, actualizar, y leer. También discutiremos la forma en la que `String` es completamente diferente a otras colecciones, concretamente, cómo indexar en una `String` es complicado por las diferencias entre cómo interpretan las personas y las computadoras
los datos de `String`.

### ¿Qué es una cadena de caracteres (String)?

Primero definiremos a qué nos referimos con el término *string*. Rust tiene un solo tipo de cadena de caracteres en el lenguaje central, el cual es el trozo de cadena `str` que es visto usualmente en su forma prestada `&str`. En el Capítulo 4, hablamos sobre *string slices*, las cuales son referencias a algunas cadenas de caracteres de datos cifradas en UTF-8 guardadas en otra parte. Las String literales, por ejemplo, son guardadas en la salida binaria del programa y son, por lo tanto, trozos de cadena.

El tipo `String` es dado en la biblioteca estándar de Rust en vez de estar cifrado en el lenguaje central y es un tipo de cadena de caracteres cifrado en UTF-8 cultivable, mutable y propio. Cuando los "Rustaceans" se refieren a “strings” en Rust, ellos usualemnte quieren decir el `String` y los tipos de trozo de cadena `&str`, no sólo uno de esos tipos. A pesar de que esta sección es en garn parte sobre `String`, ambos tipos son muy usados en la biblioteca estándar de Rust y ambos `String` y trozos de cadena son cifrados en UTF-8.

La biblioteca estándar de Rust también incluye una serie de otros tipos de cadenas, tales como `OsString`, `OsStr`, `CString`, y `CStr`. Los cajones de biblioteca pueden proveer incluso más opciones para almacenar datos de cadenas de caracteres. Similar a la asignación de nombre `*String`/`*Str`, ellos a menudo proveen una variante propia y prestada, justo como `String`/`&str`. Estos tipos de cadenas pueden almacenar texto en diferentes cifrados o ser representados en la memoria de una manera diferente, por ejemplo. No discutiremos estos otros tipos de cadena en este capítulo; vea su manual API para más sobre cómo usarlos y cuando cada uno es apropiado.

### Creando una nueva cadena de caracteres

Muchas de las mismas operaciones disponibles con `Vec<T>` están disponibles con `String` también, empezando con la función `new` para crear una cadena de caracteres, mostrada en el Listado
8-11:

```rust
let mut s = String::new();
```

<span class="caption">Listing 8-11: Creating a new, empty `String`</span>

Esta línea crea una nueva cadena vacía llamada `s` a la que luego podemos cargar datos.
A menudo, tendremos algunos datos iniciales con los que querremos iniciar la cadena. Para esto, usamos el método `to_string`, el cual está disponible en cualquier tipo que implemente la característica `Display`, la cual las string literales tienen. El listado 8-12 muetra dos ejemplos:

```rust
let data = "initial contents";

let s = data.to_string();

// the method also works on a literal directly:
let s = "initial contents".to_string();
```

<span class="caption">Listing 8-12: Using the `to_string` method to create a
`String` from a string literal</span>

Este código crea una cadena, conteniendo `initial contents`.

Podemos también usar la función `String::from` para crear una `String` desde una string literal. El código en el Listado 8-13 es equivalente al código del Listado 8-12 que usa `to_string`:

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

Esta no es la firma exacta que está en la biblioteca estándar: En la biblioteca estándar, `add` es definido usando genéricos. Aquí, estamos echando un vistazo a la firma de `add` con tipos concretos substituidos por los genéricos, que es lo que pasa cuando convocamos este método con valores de `String`. Descutiremos genéricos en el Capítulo 10. Esta firma nos da las pistas que necesitamos para entener los bits complicados del operador `+`

First, `s2` has an `&`, meaning that we’re adding a *reference* of the second
string to the first string because of the `s` parameter in the `add` function:
we can only add a `&str` to a `String`; we can’t add two `String` values
together. But wait - the type of `&s2` is `&String`, not `&str`, as specified
in the second parameter to `add`. So why does Listing 8-18 compile?

The reason we’re able to use `&s2` in the call to `add` is that the compiler
can *coerce* the `&String` argument into a `&str`. When we call the `add`
method, Rust uses a *deref coercion*, which here turns `&s2` into `&s2[..]`.
We’ll discuss deref coercion in more depth in Chapter 15. Because `add` does
not take ownership of the `s` parameter, `s2` will still be a valid `String`
after this operation.

Second, we can see in the signature that `add` takes ownership of `self`,
because `self` does *not* have an `&`. This means `s1` in Listing 8-18 will be
moved into the `add` call and no longer be valid after that. So although `let
s3 = s1 + &s2;` looks like it will copy both strings and create a new one, this
statement actually takes ownership of `s1`, appends a copy of the contents of
`s2`, and then returns ownership of the result. In other words, it looks like
it’s making a lot of copies but isn’t: the implementation is more efficient
than copying.

If we need to concatenate multiple strings, the behavior of `+` gets unwieldy:

```rust
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");

let s = s1 + "-" + &s2 + "-" + &s3;
```

At this point, `s` will be `tic-tac-toe`. With all of the `+` and `"`
characters, it’s difficult to see what’s going on. For more complicated string
combining, we can use the `format!` macro:

```rust
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");

let s = format!("{}-{}-{}", s1, s2, s3);
```

This code also sets `s` to `tic-tac-toe`. The `format!` macro works in the same
way as `println!`, but instead of printing the output to the screen, it returns
a `String` with the contents. The version of the code using `format!` is much
easier to read and also doesn’t take ownership of any of its parameters.

### Indexing into Strings

In many other programming languages, accessing individual characters in a
string by referencing them by index is a valid and common operation. However,
if we try to access parts of a `String` using indexing syntax in Rust, we’ll
get an error. Consider the invalid code in Listing 8-19:

```rust,ignore
let s1 = String::from("hello");
let h = s1[0];
```

<span class="caption">Listing 8-19: Attempting to use indexing syntax with a
String</span>

This code will result in the following error:

```text
error[E0277]: the trait bound `std::string::String: std::ops::Index<{integer}>` is not satisfied
 -->
  |
3 |     let h = s1[0];
  |             ^^^^^ the type `std::string::String` cannot be indexed by `{integer}`
  |
  = help: the trait `std::ops::Index<{integer}>` is not implemented for `std::string::String`
```

The error and the note tell the story: Rust strings don’t support indexing. But
why not? To answer that question, we need to discuss how Rust stores strings in
memory.

#### Internal Representation

A `String` is a wrapper over a `Vec<u8>`. Let’s look at some of our properly
encoded UTF-8 example strings from Listing 8-14. First, this one:

```rust
let len = String::from("Hola").len();
```

In this case, `len` will be four, which means the `Vec` storing the string
“Hola” is four bytes long. Each of these letters takes one byte when encoded in
UTF-8. But what about the following line?

```rust
let len = String::from("Здравствуйте").len();
```

Note that this string begins with the capital Cyrillic letter Ze, not the
Arabic number 3. Asked how long the string is, you might say 12. However,
Rust’s answer is 24: that’s the number of bytes it takes to encode
“Здравствуйте” in UTF-8, because each Unicode scalar value takes two bytes of
storage. Therefore, an index into the string’s bytes will not always correlate
to a valid Unicode scalar value. To demonstrate, consider this invalid Rust
code:

```rust,ignore
let hello = "Здравствуйте";
let answer = &hello[0];
```

What should the value of `answer` be? Should it be `З`, the first letter? When
encoded in UTF-8, the first byte of `З` is `208`, and the second is `151`, so
`answer` should in fact be `208`, but `208` is not a valid character on its
own. Returning `208` is likely not what a user would want if they asked for the
first letter of this string; however, that’s the only data that Rust has at
byte index 0. Returning the byte value is probably not what users want, even if
the string contains only Latin letters: if `&"hello"[0]` was valid code that
returned the byte value, it would return `104`, not `h`. To avoid returning an
unexpected value and causing bugs that might not be discovered immediately,
Rust doesn’t compile this code at all and prevents misunderstandings earlier in
the development process.

#### Bytes and Scalar Values and Grapheme Clusters! Oh My!

Another point about UTF-8 is that there are actually three relevant ways to
look at strings from Rust’s perspective: as bytes, scalar values, and grapheme
clusters (the closest thing to what we would call *letters*).

If we look at the Hindi word “नमस्ते” written in the Devanagari script, it is
ultimately stored as a `Vec` of `u8` values that looks like this:

```text
[224, 164, 168, 224, 164, 174, 224, 164, 184, 224, 165, 141, 224, 164, 164,
224, 165, 135]
```

That’s 18 bytes and is how computers ultimately store this data. If we look at
them as Unicode scalar values, which are what Rust’s `char` type is, those
bytes look like this:

```text
['न', 'म', 'स', '्', 'त', 'े']
```

There are six `char` values here, but the fourth and sixth are not letters:
they’re diacritics that don’t make sense on their own. Finally, if we look at
them as grapheme clusters, we’d get what a person would call the four letters
that make up the Hindi word:

```text
["न", "म", "स्", "ते"]
```

Rust provides different ways of interpreting the raw string data that computers
store so that each program can choose the interpretation it needs, no matter
what human language the data is in.

A final reason Rust doesn’t allow us to index into a `String` to get a
character is that indexing operations are expected to always take constant time
(O(1)). But it isn’t possible to guarantee that performance with a `String`,
because Rust would have to walk through the contents from the beginning to the
index to determine how many valid characters there were.

### Slicing Strings

Indexing into a string is often a bad idea because it’s not clear what the
return type of the string indexing operation should be: a byte value, a
character, a grapheme cluster, or a string slice. Therefore, Rust asks you to
be more specific if you really need to use indices to create string slices. To
be more specific in your indexing and indicate that you want a string slice,
rather than indexing using `[]` with a single number, you can use `[]` with a
range to create a string slice containing particular bytes:

```rust
let hello = "Здравствуйте";

let s = &hello[0..4];
```

Here, `s` will be a `&str` that contains the first four bytes of the string.
Earlier, we mentioned that each of these characters was two bytes, which means
`s` will be `Зд`.

What would happen if we used `&hello[0..1]`? The answer: Rust will panic at
runtime in the same way that accessing an invalid index in a vector does:

```text
thread 'main' panicked at 'byte index 1 is not a char boundary; it is inside 'З' (bytes 0..2) of `Здравствуйте`', src/libcore/str/mod.rs:2188:4
```

You should use ranges to create string slices with caution, because it can
crash your program.

### Methods for Iterating Over Strings

Fortunately, we can access elements in a string in other ways.

If we need to perform operations on individual Unicode scalar values, the best
way to do so is to use the `chars` method. Calling `chars` on “नमस्ते” separates
out and returns six values of type `char`, and we can iterate over the result
in order to access each element:

```rust
for c in "नमस्ते".chars() {
    println!("{}", c);
}
```

This code will print the following:

```text
न
म
स
्
त
े
```

The `bytes` method returns each raw byte, which might be appropriate for your
domain:

```rust
for b in "नमस्ते".bytes() {
    println!("{}", b);
}
```

This code will print the 18 bytes that make up this `String`, starting with:

```text
224
164
168
224
// ... etc
```

But be sure to remember that valid Unicode scalar values may be made up of more
than one byte.

Getting grapheme clusters from strings is complex, so this functionality is not
provided by the standard library. Crates are available on
[crates.io](https://crates.io) if this is the functionality you need.

### Strings Are Not So Simple

To summarize, strings are complicated. Different programming languages make
different choices about how to present this complexity to the programmer. Rust
has chosen to make the correct handling of `String` data the default behavior
for all Rust programs, which means programmers have to put more thought into
handling UTF-8 data upfront. This trade-off exposes more of the complexity of
strings than other programming languages do but prevents you from having to
handle errors involving non-ASCII characters later in your development life
cycle.

Let’s switch to something a bit less complex: hash maps!
