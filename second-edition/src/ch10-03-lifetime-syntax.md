## Validando Referencias con sus Tiempos de Vida

Cuando hablamos de referencias en el Capítulo 4, dejamos fuera un detalle importante:
cada referencia en Rust tienen un *tiempo de vida*,  el cual es el alcance en el que
la referencia es válida. La mayoría de las veces, los tiempos de vida son implícitos e inferidos, así
como la mayoría de los tipos de tiempo se infieren. De manera similar a cuando tenemos que anotar
tipos ya que son posibles varios de estos, hay casos en los que las vidas
de las referencias podrían relacionarse de diferentes maneras, por lo que Rust necesita
anotar las relaciones usando parámetros genéricos de por vida para que pueda
asegurarse de que las referencias reales utilizadas en tiempo de ejecución definitivamente son válidas.

Sí, es un poco inusual, y será diferente a las herramientas que has usado en otros
lenguajes de programación. Los tiempos de vida son, de alguna manera, lo más distintivos de las características
del Rust.

Los tiempos de vida son un gran tema que no se puede cubrir en su totalidad en este capítulo, por lo que
cubriremos las formas comunes en los que puedes encontrar la sintaxis de los tiempos de vida en este capítulo para
familiarízate con los conceptos. El Capítulo 19 contendrá información
más avanzada sobre todo lo que las vidas pueden hacer.

### Los Tiempos de Vida Previenen las Referencias Colgantes

El objetivo principal de los tiempos de vida es evitar referencias colgantes, lo que provocará que el
programa no pueda referenciar datos que no sean los datos a los que nos referimos.
Considere el programa en el listado 10-18, con un alcance externo y un alcance interno.
El ámbito externo declara una variable llamada `r` sin valor inicial, y 
el alcance interno declara una variable llamada `x` con el valor inicial de 5. Dentro
del alcance interno, intentamos establecer el valor de `r` como una referencia a` x`. Entonces
el alcance interno finaliza e intentamos imprimir el valor en `r`:

```rust,ignore
{
    let r;

    {
        let x = 5;
        r = &x;
    }

    println!("r: {}", r);
}
```

<span class="caption">Listado 10-18:Un intento de usar una referencia cuyo valor
ha salido del alcance</span>

> #### Las Variables Que No Se Inicien No Se Pueden Usar
>
> Los siguientes ejemplos declaran variables sin darles un valor inicial,
> para que el nombre de la variable exista en el ámbito externo. Esto podría parecer ser
> un conflicto con Rust ya que sería nulo. Sin embargo, si tratamos de usar una variable
> antes de darle un valor, obtendremos un error en tiempo de compilación. ¡Pruébalo!

Cuando compilamos este código, obtenemos un error:

```text
error: `x` does not live long enough
   |
6  |         r = &x;
   |              - borrow occurs here
7  |     }
   |     ^ `x` dropped here while still borrowed
...
10 | }
   | - borrowed value needs to live until here
```

La variablee `x` No “vivió lo suficiente.”¿Por qué no? Bueno, `x` estará
fuera del alcance cuando lleguemos al corchete de cierre en la línea 7, terminando el alcance
interior. Pero `r` es válido para el alcance externo; su alcance es más grande y decimos
que "vive más tiempo". Si Rust permitiera que este código funcionara, `r` sería
una referencia a la memoria que fue desasignada cuando `x` salió del alcance, y
cualquier cosa que tratasemos de hacer con `r`no funcionaría correctamente. ¿Entonces como determina
Rsut que este código no debería estar permitido?

#### El Verificador

La parte del compilador llamada *Verificador* compara alcances para
determinar que todos los puntos son válidos El Listado 10-19 muestra el mismo ejemplo del
Listado 10-18 con anotaciones que muestran las vidas de las variables:

```rust,ignore
{
    let r;                // -------+-- 'a
                          //        |
    {                     //        |
        let x = 5;        // -+-----+-- 'b
        r = &x;           //  |     |
    }                     // -+     |
                          //        |
    println!("r: {}", r); //        |
}                         // -------+
```

<span class="caption">Listado 10-19: Anotaciones del tiempo de vida de `r` y
`x`, llamado `'a` y `'b` respectivamente</span>

<!-- Solo estoy comprobando que estoy leyendo esto correctamente: ¿el bloque interno es el b de por vida,
cierto? quiero dejar una nota por producción, asegurarme de que podemos dejar esto
claro -->
<!-- Sí, el bloque interior de la vida de `'b` empieza con la línea
`let x = 5;`y termina con el primer corchete de cierre en la 7ma línea. Si piensas
que los comentarios de arte de texto funcionan o deberíamos hacer un diagrama de SVG que busca
flechas y etiquetas más bonitas? --> /Carol

Hemos anotado la vida útil de `r` con`'a` y la vida útil de `x` con
`'b`. Como puedes ver, el bloque interno `'b` es mucho más pequeño que el tiempo de vida 
del bloque `'a`. En tiempo de compilación, Rust compara el tamaño de las dos vidas
y ve que `r` tiene un tiempo de vida de `'a`, pero que se refiere a un objeto con
un tiempo de vida de `'b`. El programa es rechazado porque la vida útil de `'b` es
más pequeña que el tiempo de vida de `'a`: el sujeto de la referencia no vive
tanto como la referencia.

Veamos un ejemplo en el listado 10-20 que no intenta hacer un balanceo
de referencia y compila sin ningún error:

```rust
{
    let x = 5;            // -----+-- 'b
                          //      |
    let r = &x;           // --+--+-- 'a
                          //   |  |
    println!("r: {}", r); //   |  |
                          // --+  |
}                         // -----+
```

<span class="caption">Listado 10-20: Una referencia válida porque los datos tienen una
vida más larga que la referencia</span>

Aquí, `x` tiene el tiempo de vida de `'b`, que en este caso es mayor que `'a`. Esto
significa que `r` puede referenciar a `x`: Rust reconoce que la referencia en `r` siempre
será válida mientras `x` sea válida.

Ahora que hemos mostrado dónde están las vidas de las referencias en un
ejemplo y discutido cómo Rust analiza los tiempos de vida para asegurar que las referencias
siempre sean válidas, hablemos de vidas genéricas de parámetros y retorno
de valores en el contexto de las funciones.

### Vida de las Funciones Genéricas

Vamos a escribir una función que devolverá el más largo de dos segmentos de cadena. 
Queremos poder llamar a esta función pasándola a dos secciones de cadena, y
queremos recuperar un segmento de cadena. El código en el listado 10-21 debe imprimir `La
cadena más larga es abcd` ya que hemos implementado la función más `larga`:

<span class="filename">Nombre de archivo: src/main.rs</span>

```rust,ignore
fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";

    let result = longest(string1.as_str(), string2);
    println!("The longest string is {}", result);
}
```

<span class="caption">Listado 10-21: Una función `principal` que se llame la función más `longest`
`laerga` para encontrar las dos partes de la cadena más larga</span>

Tenga en cuenta que queremos que la función tome segmentos de cadena (que son referencias, como
hemos hablado en el capítulo 4) ya que no queremos que la función más `larga` tome
propiedad de los argumentos. Queremos que la función sea capaz de aceptar
partes de `Cadena` (en cual es el tipo de variable `cadena1`) Así como
literales de esta (la cual es la variable `cadena2` la que lo contiene).

<!-- ¿por que es `a` una parte y `b` una literal? ¿Quieres decir "a" de la cadena "abcd"? -->
<!-- Cambié los nombres de las variables para eliminar la ambigüedad entre el nombre
de la variable `a` y la cadena "a" de "abcd". `cadena1` no es una parte, es una
`Cadena`, pero vamos a pasar una porción que se refiere a eso como `Cadena` a la
función más `larga` (`cadena1.as_str()` crea un corte que hace referencia a
`Cadena` almacenado en `cadena1`). Elegimos tener `cadena2` como una literal desde que
el lector podría tener un código con ambas `cadenas` y literales de ellas, y la manera
en la que la mayoría de los lectores entran en problemas con el tiempo de vida es involucrar rebanadas de cadenas,
así que queríamos demostrar la flexibilidad de tomar partes de cadenas de
argumentos pero los problemas que puede encontrar porque las partes de cadena llegan a ser
referencias.
Todo de los conceptos de `Cadena`/parte de cadena/literal de la cadena hemos cubierto
a fondo en el Capítulo 4, por eso ponemos dos referencias atrás aquí (encima
y debajo). Si estos temas te confunden en este contexto, estarías
interesado en saber si la relectura del Capítulo 4 aclara esa confusión.
/Carol -->

Remítase a la sección "Partes de cadena como parámetros" del Capítulo 4 para obtener más información
de la discusión sobre por qué estos son los argumentos que queremos.

Si tratamos de implementar la función más `larga` como mostramos en el listado 10-22, este
no se compilará:

<span class="filename">Nombre del archivo: src/main.rs</span>

```rust,ignore
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

<span class="caption">Listado 10-22: una implementación de la función más
`larga` que devuelve lo más largo de dos segmentos de cadena, pero aún no
se compila</span>

En cambio, obtenemos el siguiente error que habla de tiempos de vida:

```text
error[E0106]: falta especificador de vida 
   |
1  | fn longest(x: &str, y: &str) -> &str {
   |                                 ^ parámetro de vida esperado
   |
   = ayuda: el tipo de devolución de esta función contiene un valor prestado, pero la
   firma no dice si es prestado de `x` o de `y`
```

El texto de ayuda nos dice que el tipo de devolución necesita un ciclo de vida genérico
del parámetro en él porque Rust no puede decir si la referencia que se devuelve se refiere
a `x` o a `y`. ¡En realidad, tampoco lo sabemos, ya que el bloque `if` en el cuerpo
de esta función regresa la referencia a `x` y el `otro` bloque regresa la
referencia a `y`!

Como estamos definiendo esta función, no conocemos los valores concretos que serán
pasadas a esta función, por lo que no sabemos si el caso `if` o` else`
se ejecutarán. Tampoco conocemos las vidas concretas de las referencias
ya que terminarán, por lo que no podemos mirar los ámbitos como lo hicimos en los listados
10-19 y 10-20 para determinar que la referencia que devolvemos siempre
sea válida. The borrow checker can’t determine this either, because it doesn’t
know how the lifetimes of `x` and `y` relate to the lifetime of the return
value. We’re going to add generic lifetime parameters that will define the
relationship between the references so that the borrow checker can perform its
análisis.

### Lifetime Annotation Syntax

Lifetime annotations don’t change how long any of the references involved live.
In the same way that functions can accept any type when the signature specifies
a generic type parameter, functions can accept references with any lifetime
when the signature specifies a generic lifetime parameter. What lifetime
annotations do is relate the lifetimes of multiple references to each other.

Lifetime annotations have a slightly unusual syntax: the names of lifetime
parameters must start with an apostrophe `'`. The names of lifetime parameters
are usually all lowercase, and like generic types, their names are usually very
short. `'a` is the name most people use as a default. Lifetime parameter
annotations go after the `&` of a reference, and a space separates the lifetime
annotation from the reference’s type.

Here’s some examples: we’ve got a reference to an `i32` without a lifetime
parameter, a reference to an `i32` that has a lifetime parameter named `'a`,
and a mutable reference to an `i32` that also has the lifetime `'a`:

```rust,ignore
&i32        // a reference
&'a i32     // a reference with an explicit lifetime
&'a mut i32 // a mutable reference with an explicit lifetime
```

One lifetime annotation by itself doesn’t have much meaning: lifetime
annotations tell Rust how the generic lifetime parameters of multiple
references relate to each other. If we have a function with the parameter
`first` that is a reference to an `i32` that has the lifetime `'a`, and the
function has another parameter named `second` that is another reference to an
`i32` that also has the lifetime `'a`, these two lifetime annotations that have
the same name indicate that the references `first` and `second` must both live
as long as the same generic lifetime.

### Lifetime Annotations in Function Signatures

Let’s look at lifetime annotations in the context of the `longest` function
we’re working on. Just like generic type parameters, generic lifetime
parameters need to be declared within angle brackets between the function name
and the parameter list. The constraint we want to tell Rust about for the
references in the parameters and the return value is that they all must have
the same lifetime, which we’ll name `'a` and add to each reference as shown in
Listing 10-23:

<span class="filename">Filename: src/main.rs</span>

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

<span class="caption">Listing 10-23: The `longest` function definition that
specifies all the references in the signature must have the same lifetime,
`'a`</span>

This will compile and will produce the result we want when used with the `main`
function in Listing 10-21.

The function signature now says that for some lifetime `'a`, the function will
get two parameters, both of which are string slices that live at least as long
as the lifetime `'a`. The function will return a string slice that also will
last at least as long as the lifetime `'a`. This is the contract we are telling
Rust we want it to enforce.

By specifying the lifetime parameters in this function signature, we are not
changing the lifetimes of any values passed in or returned, but we are saying
that any values that do not adhere to this contract should be rejected by the
borrow checker. This function does not know (or need to know) exactly how long
`x` and `y` will live, but only needs to know that there is some scope that
can be substituted for `'a` that will satisfy this signature.

When annotating lifetimes in functions, the annotations go on the function
signature, and not in any of the code in the function body. This is because
Rust is able to analyze the code within the function without any help, but when
a function has references to or from code outside that function, the lifetimes
of the arguments or return values will potentially be different each time the
function is called. This would be incredibly costly and often impossible for
Rust to figure out. In this case, we need to annotate the lifetimes ourselves.

When concrete references are passed to `longest`, the concrete lifetime that
gets substituted for `'a` is the part of the scope of `x` that overlaps with
the scope of `y`. Since scopes always nest, another way to say this is that the
generic lifetime `'a` will get the concrete lifetime equal to the smaller of
the lifetimes of `x` and `y`. Because we’ve annotated the returned reference
with the same lifetime parameter `'a`, the returned reference will therefore be
guaranteed to be valid as long as the shorter of the lifetimes of `x` and `y`.

Let’s see how this restricts the usage of the `longest` function by passing in
references that have different concrete lifetimes. Listing 10-24 is a
straightforward example that should match your intuition from any language:
`string1` is valid until the end of the outer scope, `string2` is valid until
the end of the inner scope, and `result` references something that is valid
until the end of the inner scope. The borrow checker approves of this code; it
will compile and print `The longest string is long string is long` when run:

<span class="filename">Filename: src/main.rs</span>

```rust
# fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
#     if x.len() > y.len() {
#         x
#     } else {
#         y
#     }
# }
#
fn main() {
    let string1 = String::from("long string is long");

    {
        let string2 = String::from("xyz");
        let result = longest(string1.as_str(), string2.as_str());
        println!("The longest string is {}", result);
    }
}
```

<span class="caption">Listing 10-24: Using the `longest` function with
references to `String` values that have different concrete lifetimes</span>

Next, let’s try an example that will show that the lifetime of the reference in
`result` must be the smaller lifetime of the two arguments. We’ll move the
declaration of the `result` variable outside the inner scope, but leave the
assignment of the value to the `result` variable inside the scope with
`string2`. Next, we’ll move the `println!` that uses `result` outside of the
inner scope, after it has ended. The code in Listing 10-25 will not compile:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
fn main() {
    let string1 = String::from("long string is long");
    let result;
    {
        let string2 = String::from("xyz");
        result = longest(string1.as_str(), string2.as_str());
    }
    println!("The longest string is {}", result);
}
```

<span class="caption">Listing 10-25: Attempting to use `result` after `string2`
has gone out of scope won’t compile</span>

If we try to compile this, we’ll get this error:

```text
error: `string2` does not live long enough
   |
6  |         result = longest(string1.as_str(), string2.as_str());
   |                                            ------- borrow occurs here
7  |     }
   |     ^ `string2` dropped here while still borrowed
8  |     println!("The longest string is {}", result);
9  | }
   | - borrowed value needs to live until here
```

The error is saying that in order for `result` to be valid for the `println!`,
`string2` would need to be valid until the end of the outer scope. Rust knows
this because we annotated the lifetimes of the function parameters and return
values with the same lifetime parameter, `'a`.

We can look at this code as humans and see that `string1` is longer, and
therefore `result` will contain a reference to `string1`. Because `string1` has
not gone out of scope yet, a reference to `string1` will still be valid for the
`println!`. However, what we’ve told Rust with the lifetime parameters is that
the lifetime of the reference returned by the `longest` function is the same as
the smaller of the lifetimes of the references passed in. Therefore, the borrow
checker disallows the code in Listing 10-25 as possibly having an invalid
reference.

Try designing some more experiments that vary the values and lifetimes of the
references passed in to the `longest` function and how the returned reference
is used. Make hypotheses about whether your experiments will pass the borrow
checker or not before you compile, then check to see if you’re right!

### Thinking in Terms of Lifetimes

The exact way to specify lifetime parameters depends on what your function is
doing. For example, if we changed the implementation of the `longest` function
to always return the first argument rather than the longest string slice, we
wouldn’t need to specify a lifetime on the `y` parameter. This code compiles:

<span class="filename">Filename: src/main.rs</span>

```rust
fn longest<'a>(x: &'a str, y: &str) -> &'a str {
    x
}
```

In this example, we’ve specified a lifetime parameter `'a` for the parameter
`x` and the return type, but not for the parameter `y`, since the lifetime of
`y` does not have any relationship with the lifetime of `x` or the return value.

When returning a reference from a function, the lifetime parameter for the
return type needs to match the lifetime parameter of one of the arguments. If
the reference returned does *not* refer to one of the arguments, the only other
possibility is that it refers to a value created within this function, which
would be a dangling reference since the value will go out of scope at the end
of the function. Consider this attempted implementation of the `longest`
function that won’t compile:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
fn longest<'a>(x: &str, y: &str) -> &'a str {
    let result = String::from("really long string");
    result.as_str()
}
```

Even though we’ve specified a lifetime parameter `'a` for the return type, this
implementation fails to compile because the return value lifetime is not
related to the lifetime of the parameters at all. Here’s the error message we
get:

```text
error: `result` does not live long enough
  |
3 |     result.as_str()
  |     ^^^^^^ does not live long enough
4 | }
  | - borrowed value only lives until here
  |
note: borrowed value must be valid for the lifetime 'a as defined on the block
at 1:44...
  |
1 | fn longest<'a>(x: &str, y: &str) -> &'a str {
  |                                             ^
```

The problem is that `result` will go out of scope and get cleaned up at the end
of the `longest` function, and we’re trying to return a reference to `result`
from the function. There’s no way we can specify lifetime parameters that would
change the dangling reference, and Rust won’t let us create a dangling
reference. In this case, the best fix would be to return an owned data type
rather than a reference so that the calling function is then responsible for
cleaning up the value.

Ultimately, lifetime syntax is about connecting the lifetimes of various
arguments and return values of functions. Once they’re connected, Rust has
enough information to allow memory-safe operations and disallow operations that
would create dangling pointers or otherwise violate memory safety.

### Lifetime Annotations in Struct Definitions

Up until now, we’ve only defined structs to hold owned types. It is possible
for structs to hold references, but we need to add a lifetime annotation on
every reference in the struct’s definition. Listing 10-26 has a struct named
`ImportantExcerpt` that holds a string slice:

<span class="filename">Filename: src/main.rs</span>

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.')
        .next()
        .expect("Could not find a '.'");
    let i = ImportantExcerpt { part: first_sentence };
}
```

<span class="caption">Listing 10-26: A struct that holds a reference, so its
definition needs a lifetime annotation</span>

This struct has one field, `part`, that holds a string slice, which is a
reference. Just like with generic data types, we have to declare the name of
the generic lifetime parameter inside angle brackets after the name of the
struct so that we can use the lifetime parameter in the body of the struct
definition.

The `main` function here creates an instance of the `ImportantExcerpt` struct
that holds a reference to the first sentence of the `String` owned by the
variable `novel`.

### Lifetime Elision

In this section, we’ve learned that every reference has a lifetime, and we need
to specify lifetime parameters for functions or structs that use references.
However, in Chapter 4 we had a function in the “String Slices” section, shown
again in Listing 10-27, that compiled without lifetime annotations:

<span class="filename">Filename: src/lib.rs</span>

```rust
fn first_word(s: &str) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}
```

<span class="caption">Listing 10-27: A function we defined in Chapter 4 that
compiled without lifetime annotations, even though the parameter and return
type are references</span>

The reason this function compiles without lifetime annotations is historical:
in early versions of pre-1.0 Rust, this indeed wouldn’t have compiled. Every
reference needed an explicit lifetime. At that time, the function signature
would have been written like this:

```rust,ignore
fn first_word<'a>(s: &'a str) -> &'a str {
```

After writing a lot of Rust code, the Rust team found that Rust programmers
were typing the same lifetime annotations over and over in particular
situations. These situations were predictable and followed a few deterministic
patterns. The Rust team then programmed these patterns into the Rust compiler’s
code so that the borrow checker can infer the lifetimes in these situations
without forcing the programmer to explicitly add the annotations.

We mention this piece of Rust history because it’s entirely possible that more
deterministic patterns will emerge and be added to the compiler. In the future,
even fewer lifetime annotations might be required.

The patterns programmed into Rust’s analysis of references are called the
*lifetime elision rules*. These aren’t rules for programmers to follow; the
rules are a set of particular cases that the compiler will consider, and if
your code fits these cases, you don’t need to write the lifetimes explicitly.

The elision rules don’t provide full inference: if Rust deterministically
applies the rules but there’s still ambiguity as to what lifetimes the
references have, it won’t guess what the lifetime of the remaining references
should be. In this case, the compiler will give you an error that can be
resolved by adding the lifetime annotations that correspond to your intentions
for how the references relate to each other.

First, some definitions: Lifetimes on function or method parameters are called
*input lifetimes*, and lifetimes on return values are called *output lifetimes*.

Now, on to the rules that the compiler uses to figure out what lifetimes
references have when there aren’t explicit annotations. The first rule applies
to input lifetimes, and the second two rules apply to output lifetimes. If the
compiler gets to the end of the three rules and there are still references that
it can’t figure out lifetimes for, the compiler will stop with an error.

1. Each parameter that is a reference gets its own lifetime parameter. In other
   words, a function with one parameter gets one lifetime parameter: `fn
   foo<'a>(x: &'a i32)`, a function with two arguments gets two separate
   lifetime parameters: `fn foo<'a, 'b>(x: &'a i32, y: &'b i32)`, and so on.

2. If there is exactly one input lifetime parameter, that lifetime is assigned
   to all output lifetime parameters: `fn foo<'a>(x: &'a i32) -> &'a i32`.

3. If there are multiple input lifetime parameters, but one of them is `&self`
   or `&mut self` because this is a method, then the lifetime of `self` is
   assigned to all output lifetime parameters. This makes writing methods much
   nicer.

Let’s pretend we’re the compiler and apply these rules to figure out what the
lifetimes of the references in the signature of the `first_word` function in
Listing 10-27 are. The signature starts without any lifetimes associated with
the references:

```rust,ignore
fn first_word(s: &str) -> &str {
```

Then we (as the compiler) apply the first rule, which says each parameter gets
its own lifetime. We’re going to call it `'a` as usual, so now the signature is:

```rust,ignore
fn first_word<'a>(s: &'a str) -> &str {
```

On to the second rule, which applies because there is exactly one input
lifetime. The second rule says the lifetime of the one input parameter gets
assigned to the output lifetime, so now the signature is:

```rust,ignore
fn first_word<'a>(s: &'a str) -> &'a str {
```

Now all the references in this function signature have lifetimes, and the
compiler can continue its analysis without needing the programmer to annotate
the lifetimes in this function signature.

Let’s do another example, this time with the `longest` function that had no
lifetime parameters when we started working with in Listing 10-22:

```rust,ignore
fn longest(x: &str, y: &str) -> &str {
```

Pretending we’re the compiler again, let’s apply the first rule: each parameter
gets its own lifetime. This time we have two parameters, so we have two
lifetimes:

```rust,ignore
fn longest<'a, 'b>(x: &'a str, y: &'b str) -> &str {
```

Looking at the second rule, it doesn’t apply since there is more than one input
lifetime. Looking at the third rule, this also does not apply because this is a
function rather than a method, so none of the parameters are `self`. So we’re
out of rules, but we haven’t figured out what the return type’s lifetime is.
This is why we got an error trying to compile the code from Listing 10-22: the
compiler worked through the lifetime elision rules it knows, but still can’t
figure out all the lifetimes of the references in the signature.

Because the third rule only really applies in method signatures, let’s look at
lifetimes in that context now, and see why the third rule means we don’t have
to annotate lifetimes in method signatures very often.

### Lifetime Annotations in Method Definitions

<!-- Is this different to the reference lifetime annotations, or just a
finalized explanation? -->
<!-- This is about lifetimes on references in method signatures, which is where
the 3rd lifetime elision rule kicks in. It can also be confusing where lifetime
parameters need to be declared and used since the lifetime parameters could go
with the struct's fields or with references passed into or returned from
methods. /Carol -->

When we implement methods on a struct with lifetimes, the syntax is again the
same as that of generic type parameters that we showed in Listing 10-11: the
place that lifetime parameters are declared and used depends on whether the
lifetime parameter is related to the struct fields or the method arguments and
return values.

Lifetime names for struct fields always need to be declared after the `impl`
keyword and then used after the struct’s name, since those lifetimes are part
of the struct’s type.

In method signatures inside the `impl` block, references might be tied to the
lifetime of references in the struct’s fields, or they might be independent. In
addition, the lifetime elision rules often make it so that lifetime annotations
aren’t necessary in method signatures. Let’s look at some examples using the
struct named `ImportantExcerpt` that we defined in Listing 10-26.

First, here’s a method named `level`. The only parameter is a reference to
`self`, and the return value is just an `i32`, not a reference to anything:

```rust
# struct ImportantExcerpt<'a> {
#     part: &'a str,
# }
#
impl<'a> ImportantExcerpt<'a> {
    fn level(&self) -> i32 {
        3
    }
}
```

The lifetime parameter declaration after `impl` and use after the type name is
required, but we’re not required to annotate the lifetime of the reference to
`self` because of the first elision rule.

Here’s an example where the third lifetime elision rule applies:

```rust
# struct ImportantExcerpt<'a> {
#     part: &'a str,
# }
#
impl<'a> ImportantExcerpt<'a> {
    fn announce_and_return_part(&self, announcement: &str) -> &str {
        println!("Attention please: {}", announcement);
        self.part
    }
}
```

There are two input lifetimes, so Rust applies the first lifetime elision rule
and gives both `&self` and `announcement` their own lifetimes. Then, because
one of the parameters is `&self`, the return type gets the lifetime of `&self`,
and all lifetimes have been accounted for.

### The Static Lifetime

There is *one* special lifetime we need to discuss: `'static`. The `'static`
lifetime is the entire duration of the program. All string literals have the
`'static` lifetime, which we can choose to annotate as follows:

```rust
let s: &'static str = "I have a static lifetime.";
```

The text of this string is stored directly in the binary of your program and
the binary of your program is always available. Therefore, the lifetime of all
string literals is `'static`.

<!-- How would you add a static lifetime (below)? -->
<!-- Just like you'd specify any lifetime, see above where it shows `&'static str`. /Carol -->

You may see suggestions to use the `'static` lifetime in error message help
text, but before specifying `'static` as the lifetime for a reference, think
about whether the reference you have is one that actually lives the entire
lifetime of your program or not (or even if you want it to live that long, if
it could). Most of the time, the problem in the code is an attempt to create a
dangling reference or a mismatch of the available lifetimes, and the solution
is fixing those problems, not specifying the `'static` lifetime.

### Generic Type Parameters, Trait Bounds, and Lifetimes Together

Let’s briefly look at the syntax of specifying generic type parameters, trait
bounds, and lifetimes all in one function!

```rust
use std::fmt::Display;

fn longest_with_an_announcement<'a, T>(x: &'a str, y: &'a str, ann: T) -> &'a str
    where T: Display
{
    println!("Announcement! {}", ann);
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

This is the `longest` function from Listing 10-23 that returns the longest of
two string slices, but with an extra argument named `ann`. The type of `ann` is
the generic type `T`, which may be filled in by any type that implements the
`Display` trait as specified by the `where` clause. This extra argument will be
printed out before the function compares the lengths of the string slices,
which is why the `Display` trait bound is necessary. Because lifetimes are a
type of generic, the declarations of both the lifetime parameter `'a` and the
generic type parameter `T` go in the same list within the angle brackets after
the function name.

## Summary

We covered a lot in this chapter! Now that you know about generic type
parameters, traits and trait bounds, and generic lifetime parameters, you’re
ready to write code that isn’t duplicated but can be used in many different
situations. Generic type parameters mean the code can be applied to different
types. Traits and trait bounds ensure that even though the types are generic,
those types will have the behavior the code needs. Relationships between the
lifetimes of references specified by lifetime annotations ensure that this
flexible code won’t have any dangling references. And all of this happens at
compile time so that run-time performance isn’t affected!

Believe it or not, there’s even more to learn in these areas: Chapter 17 will
discuss trait objects, which are another way to use traits. Chapter 19 will be
covering more complex scenarios involving lifetime annotations. Chapter 20 will
get to some advanced type system features. Up next, though, let’s talk about
how to write tests in Rust so that we can make sure our code using all these
features is working the way we want it to!
